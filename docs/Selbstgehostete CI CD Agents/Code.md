# Code
In diesem Schritt richten wir ein neues Azure DevOps Repository ein, das als Grundlage für die Erstellung und Verwaltung von CI/CD-Pipelines dient. Dieses Repository wird die notwendigen Konfigurationsdateien und Docker-Skripte enthalten, um einen selbstgehosteten Agenten in einer Containerumgebung zu betreiben. 

## Neues Azure DevOps Repo erstellen
Erstelle ein neues Azure DevOps Repo.
Klone das Repo lokal und lege folgende Struktur an:
```
Repo
└───agent
    │   BuildEnvironment.yml
    │   TestAgent.yml
└───docker
    │   Dockerfile
    └───azure-pipelines-agent
        |    start.sh
```
Füge die Inhalte wie beschrieben in die Dateien ein.

Die Files sollten folgende Inhalte haben:

### start.sh
**docker\azure-pipeline-agent\start.sh**
```sh
#!/bin/bash
set -e

if [ -z "$AZP_URL" ]; then
  echo 1>&2 "error: missing AZP_URL environment variable"
  exit 1
fi

if [ -z "$AZP_TOKEN_FILE" ]; then
  if [ -z "$AZP_TOKEN" ]; then
    echo 1>&2 "error: missing AZP_TOKEN environment variable"
    exit 1
  fi

  AZP_TOKEN_FILE=/azp/.token
  echo -n $AZP_TOKEN > "$AZP_TOKEN_FILE"
fi

unset AZP_TOKEN

if [ -n "$AZP_WORK" ]; then
  mkdir -p "$AZP_WORK"
fi

export AGENT_ALLOW_RUNASROOT="1"

cleanup() {
  # If $AZP_PLACEHOLDER is set, skip cleanup
  if [ -n "$AZP_PLACEHOLDER" ]; then
    echo 'Running in placeholder mode, skipping cleanup'
    return
  fi
  if [ -e config.sh ]; then
    print_header "Cleanup. Removing Azure Pipelines agent..."

    # If the agent has some running jobs, the configuration removal process will fail.
    # So, give it some time to finish the job.
    while true; do
      ./config.sh remove --unattended --auth PAT --token $(cat "$AZP_TOKEN_FILE") && break

      echo "Retrying in 30 seconds..."
      sleep 30
    done
  fi
}

print_header() {
  lightcyan='\033[1;36m'
  nocolor='\033[0m'
  echo -e "${lightcyan}$1${nocolor}"
}

# Let the agent ignore the token env variables
export VSO_AGENT_IGNORE=AZP_TOKEN,AZP_TOKEN_FILE

print_header "1. Determining matching Azure Pipelines agent..."

AZP_AGENT_PACKAGES=$(curl -LsS \
    -u user:$(cat "$AZP_TOKEN_FILE") \
    -H 'Accept:application/json;' \
    "$AZP_URL/_apis/distributedtask/packages/agent?platform=$TARGETARCH&top=1")

AZP_AGENT_PACKAGE_LATEST_URL=$(echo "$AZP_AGENT_PACKAGES" | jq -r '.value[0].downloadUrl')

if [ -z "$AZP_AGENT_PACKAGE_LATEST_URL" -o "$AZP_AGENT_PACKAGE_LATEST_URL" == "null" ]; then
  echo 1>&2 "error: could not determine a matching Azure Pipelines agent"
  echo 1>&2 "check that account '$AZP_URL' is correct and the token is valid for that account"
  exit 1
fi

print_header "2. Downloading and extracting Azure Pipelines agent..."
echo "Agent package URL: $AZP_AGENT_PACKAGE_LATEST_URL"
curl -LsS $AZP_AGENT_PACKAGE_LATEST_URL | tar -xz & wait $!

source ./env.sh

trap 'cleanup; exit 0' EXIT
trap 'cleanup; exit 130' INT
trap 'cleanup; exit 143' TERM

print_header "3. Configuring Azure Pipelines agent..."

./config.sh --unattended \
  --agent "${AZP_AGENT_NAME:-$(hostname)}" \
  --url "$AZP_URL" \
  --auth PAT \
  --token $(cat "$AZP_TOKEN_FILE") \
  --pool "${AZP_POOL:-Default}" \
  --work "${AZP_WORK:-_work}" \
  --replace \
  --acceptTeeEula & wait $!

print_header "4. Running Azure Pipelines agent..."

trap 'cleanup; exit 0' EXIT
trap 'cleanup; exit 130' INT
trap 'cleanup; exit 143' TERM

chmod +x ./run.sh


# If $AZP_PLACEHOLDER is set, skipping running the agent
if [ -n "$AZP_PLACEHOLDER" ]; then
  echo 'Running in placeholder mode, skipping running the agent'
else
  # To be aware of TERM and INT signals call run.sh
  # Running it with the --once flag at the end will shut down the agent after the build is executed
  ./run.sh --once & wait $!
fi
```
> Hinweis: Prüft mittels Visual Studio Code die End of Line Sequens diese muss auf LF stehen **nicht** `CRLF`. Sonst kommt es zu fehlern.

#### Ziel des Skripts

Das Skript startet einen **Azure Pipelines Agent** in einem Docker-Container. Dieser Agent verbindet sich mit Azure DevOps und führt dort automatisch Aufgaben aus (z. B. Builds oder Deployments). Dazu muss er wissen:

- Wo sich der Azure DevOps Server befindet `AZP_URL`
- Welchen Zugangstoken er verwenden darf `AZP_TOKEN`
- In welchem Pool und Arbeitsverzeichnis er laufen soll `AZP_POOL`

#### Schritt-für-Schritt-Erklärung
**Fehlende Umgebungsvariablen prüfen**
```bash
if [ -z "$AZP_URL" ]; then
  echo 1>&2 "error: missing AZP_URL environment variable"
  exit 1
fi
```
- Wenn die Variable `AZP_URL` **nicht gesetzt** ist (also leer), bricht das Skript ab.
- `AZP_URL` ist die Adresse deiner Azure DevOps Instanz (z. B. `https://dev.azure.com/mycompany`).

---

**Zugangs-Token absichern**

```bash
if [ -z "$AZP_TOKEN_FILE" ]; then
  if [ -z "$AZP_TOKEN" ]; then
    echo 1>&2 "error: missing AZP_TOKEN environment variable"
    exit 1
  fi

  AZP_TOKEN_FILE=/azp/.token
  echo -n $AZP_TOKEN > "$AZP_TOKEN_FILE"
fi
```
- Der Agent braucht einen **Token**, um sich bei Azure DevOps anzumelden.
- Wenn nur `AZP_TOKEN` gesetzt ist, wird der Token in eine Datei geschrieben (`.token`).

```bash
unset AZP_TOKEN
```
- Aus Sicherheitsgründen wird der Token **aus dem Speicher gelöscht**, nachdem er in die Datei geschrieben wurde.

---

**Arbeitsverzeichnis vorbereiten**
```bash
if [ -n "$AZP_WORK" ]; then
  mkdir -p "$AZP_WORK"
fi
```
- Wenn ein Arbeitsverzeichnis angegeben wurde (`AZP_WORK`), wird es erstellt.

---

**Root-Nutzung erlauben**

```bash
export AGENT_ALLOW_RUNASROOT="1"
```
- Azure verbietet normalerweise, dass der Agent als „root“ läuft.
- Mit dieser Einstellung erlaubst du es trotzdem – notwendig im Container.

---

**Aufräumen vorbereiten**

```bash
cleanup() {
  ...
}
```
- Diese Funktion wird **automatisch aufgerufen**, wenn das Skript beendet wird (z. B. durch STRG+C).
- Sie entfernt den Agenten sauber von Azure DevOps – außer, wenn du im „Placeholder“-Modus arbeitest (`AZP_PLACEHOLDER` gesetzt).

---

**Passende Agent-Version finden**

```bash
print_header "1. Determining matching Azure Pipelines agent..."
AZP_AGENT_PACKAGES=$(curl ... )
```

- Das Skript fragt bei Azure DevOps nach, **welche Version des Azure Agents** zur aktuellen Plattform passt.
- Dafür wird der Token aus der Datei genutzt.

---

**Agent herunterladen & entpacken**

```bash
curl -LsS $AZP_AGENT_PACKAGE_LATEST_URL | tar -xz
```
- Der Agent wird direkt heruntergeladen und **entpackt** (kein Zwischenspeichern nötig).
---

**Agent konfigurieren**
```bash
./config.sh --unattended ...
```
Der Agent wird mit allen nötigen Infos **ohne Rückfragen** eingerichtet:

  - Agentname (Hostname oder manuell gesetzt)
  - Azure DevOps URL
  - Authentifizierung mit Token
  - Poolname
  - Arbeitsverzeichnis
  - Lizenzbedingungen akzeptieren
  - Existierenden Agent ersetzen, wenn nötig
---

**Agent starten**

```bash
./run.sh --once
```

- `run.sh` startet den Agenten.
- Mit `--once` wird der Agent **nach einem Durchlauf automatisch beendet** – perfekt für Container.

**Fazit**
Das `start.sh`-Skript sorgt dafür, dass sich der Container beim Start automatisch bei Azure DevOps anmeldet, den passenden Agent herunterlädt, konfiguriert und ausführt.

**Wichtig zu wissen**
- Die Kommunikation mit Azure DevOps basiert auf ein paar wenigen Umgebungsvariablen.
- Alles andere wird automatisch gemacht: Download, Einrichtung, Start.
- Der Agent lebt nur so lange wie der Container – beim nächsten Start passiert alles neu.

### Dockerfile
**agent\Dockerfile**
```Dockerfile
FROM ubuntu:20.04 AS builder
ENV DEBIAN_FRONTEND=noninteractive \
    ARCH=linux_amd64 \
    DB_VERSION=0.250.0 \
    YQ_VERSION=4.45.2 \
    TF_VERSION=1.11.4 \
    TF_DOCS_VERSION=0.20.0

WORKDIR /build

RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-transport-https \
    apt-utils \
    ca-certificates \
    curl \
    iputils-ping \
    lsb-release \
    software-properties-common \
    unzip \
    zip && \
    curl -fsSL "https://github.com/databricks/cli/releases/download/v${DB_VERSION}/databricks_cli_${DB_VERSION}_${ARCH}.zip" -o databricks.zip && \
    unzip databricks.zip && rm databricks.zip && chmod +x databricks && \
    curl -fsSL "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_${ARCH}" -o yq && chmod +x yq && \
    curl -fsSL "https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_${ARCH}.zip" -o terraform.zip && \
    unzip terraform.zip && rm terraform.zip && chmod +x terraform && \
    curl -fsSL https://github.com/terraform-docs/terraform-docs/releases/download/v${TF_DOCS_VERSION}/terraform-docs-v${TF_DOCS_VERSION}-linux-amd64.tar.gz | tar -xz terraform-docs && chmod +x terraform-docs && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

FROM ubuntu:20.04 AS agent-image
ENV DEBIAN_FRONTEND=noninteractive \
    TARGETARCH=linux-x64 \
    MKDOCS_VERSION=1.6.1 \
    MKDOCS_MATERIAL_VERSION=9.6.12 \
    PS_VERSION=7.5.1 \
    PS_PACKAGE=powershell_7.5.1-1.deb_amd64.deb

WORKDIR /azp

RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-transport-https \
    apt-utils \
    ca-certificates \
    curl \
    git \
    iputils-ping \
    jq \
    lsb-release \
    python3 \
    python3-pip \
    rsync \
    software-properties-common \
    unzip \
    wget \
    zip && \
    pip3 install --no-cache-dir mkdocs==${MKDOCS_VERSION} mkdocs-material==${MKDOCS_MATERIAL_VERSION} && \
    wget https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/${PS_PACKAGE} && \
    dpkg -i ${PS_PACKAGE} || apt-get install -f -y && rm ${PS_PACKAGE} && \
    curl -sL https://aka.ms/InstallAzureCLIDeb | bash && \
    rm -rf /usr/share/doc /usr/share/doc-base /usr/share/man /usr/share/locale /usr/share/zoneinfo && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

COPY --from=builder /build/databricks /usr/local/bin/databricks
COPY --from=builder /build/yq /usr/local/bin/yq
COPY --from=builder /build/terraform /usr/local/bin/terraform
COPY --from=builder /build/terraform-docs /usr/local/bin/terraform-docs
COPY azure-pipelines-agent/start.sh .
RUN chmod +x start.sh

ENTRYPOINT ["./start.sh"]
```

#### Was ist ein Dockerfile?
Ein **Dockerfile** ist eine Textdatei mit Anweisungen, wie ein sogenanntes **Docker-Image** gebaut werden soll. Ein Docker-Image ist so etwas wie ein Schnappschuss eines fertigen Systems (inkl. Software, Konfiguration, Skripten usw.), das dann in einem Container ausgeführt werden kann.

Ein **Container** ist eine Art „Mini-Computer“, der isoliert auf deinem System läuft – immer auf der Basis des Images.

**Besonderheit: Multi-Stage Build (vereinfacht erklärt)**

Dieses Dockerfile nutzt **zwei** Abschnitte, die jeweils mit `FROM ...` anfangen. Das ist ein **Multi-Stage Build**: Es hilft dabei, unnötige Dateien (z. B. Installationsdateien) aus dem finalen Container herauszuhalten. Der erste Teil dient als „Bauarbeiter“, der die Tools installiert. Der zweite Teil ist der eigentliche „Container“, den du später benutzt.


#### Schritt-für-Schritt-Erklärung
**Bauphase: Werkzeuge bauen & vorbereiten**

```dockerfile
FROM ubuntu:20.04 AS builder
```
- Wir starten mit einem Basis-Betriebssystem: **Ubuntu 20.04**. Das ist eine Linux-Variante.

---

```dockerfile
ENV DEBIAN_FRONTEND=noninteractive \
    ARCH=linux_amd64 \
    DB_VERSION=0.250.0 \
    YQ_VERSION=4.45.2 \
    TF_VERSION=1.11.4 \
    TF_DOCS_VERSION=0.20.0
```
- Diese `ENV`-Anweisungen setzen Umgebungsvariablen. Diese Werte (wie z. B. Versionsnummern) werden später beim Herunterladen von Tools verwendet.

---
```dockerfile
WORKDIR /build
```

- Wechselt in das Arbeitsverzeichnis `/build`. Alles, was ab jetzt installiert oder gespeichert wird, landet hier.
---

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-transport-https \
    apt-utils \
    ca-certificates \
    curl \
    iputils-ping \
    lsb-release \
    software-properties-common \
    unzip \
    zip && \
    curl -fsSL "https://github.com/databricks/cli/releases/download/v${DB_VERSION}/databricks_cli_${DB_VERSION}_${ARCH}.zip" -o databricks.zip && \
    unzip databricks.zip && rm databricks.zip && chmod +x databricks && \
    curl -fsSL "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_${ARCH}" -o yq && chmod +x yq && \
    curl -fsSL "https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_${ARCH}.zip" -o terraform.zip && \
    unzip terraform.zip && rm terraform.zip && chmod +x terraform && \
    curl -fsSL https://github.com/terraform-docs/terraform-docs/releases/download/v${TF_DOCS_VERSION}/terraform-docs-v${TF_DOCS_VERSION}-linux-amd64.tar.gz | tar -xz terraform-docs && chmod +x terraform-docs && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
```

Dieser `RUN`-Befehl installiert wichtige Tools aus dem Ubuntu-Softwarekatalog, z. B.:

- `curl`: zum Herunterladen von Dateien
- `unzip`, `zip`: um ZIP-Dateien zu entpacken
- `iputils-ping`: für Netzwerk-Checks

Danach lädt der Befehl verschiedene Programme von GitHub herunter und entpackt sie:

- **databricks CLI** (Tool für Databricks)
- **yq** (wie `jq`, aber für YAML)
- **terraform** (für Infrastructure-as-Code)
- **terraform-docs** (zum automatischen Dokumentieren von Terraform-Code)

Zum Schluss wird aufgeräumt: temporäre Dateien werden gelöscht.

---

**Finale Phase: Das tatsächliche Docker-Image**

```dockerfile
FROM ubuntu:20.04 AS agent-image
```
-  Noch einmal ein Ubuntu-Image. Hier startet die „zweite Phase“: der eigentliche Container, den du später benutzt.

---

```dockerfile
ENV DEBIAN_FRONTEND=noninteractive ...
```

Wieder werden Umgebungsvariablen gesetzt – diesmal für andere Tools wie:

- **MkDocs** (Dokumentationstool)
- **PowerShell**
- **Azure CLI**

---

``dockerfile
WORKDIR /azp
```

- Wechsel ins Verzeichnis `/azp`, wo später gearbeitet wird.

---

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-transport-https \
    apt-utils \
    ca-certificates \
    curl \
    git \
    iputils-ping \
    jq \
    lsb-release \
    python3 \
    python3-pip \
    rsync \
    software-properties-common \
    unzip \
    wget \
    zip && \
    pip3 install --no-cache-dir mkdocs==${MKDOCS_VERSION} mkdocs-material==${MKDOCS_MATERIAL_VERSION} && \
    wget https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/${PS_PACKAGE} && \
    dpkg -i ${PS_PACKAGE} || apt-get install -f -y && rm ${PS_PACKAGE} && \
    curl -sL https://aka.ms/InstallAzureCLIDeb | bash && \
    rm -rf /usr/share/doc /usr/share/doc-base /usr/share/man /usr/share/locale /usr/share/zoneinfo && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
```

Hier wird wieder Software installiert:
- **git**, **curl**, **wget**, **python3**, etc.
- Dann wird MkDocs installiert (über `pip`)
- PowerShell wird manuell installiert
- Azure CLI wird über ein Skript installiert

Danach wird wieder „aufgeräumt“ – unnötige Dateien entfernt, um das Image schlanker zu halten.

---

**Werkzeuge kopieren**

```dockerfile
COPY --from=builder /build/databricks /usr/local/bin/databricks
COPY --from=builder /build/yq /usr/local/bin/yq
COPY --from=builder /build/terraform /usr/local/bin/terraform
COPY --from=builder /build/terraform-docs /usr/local/bin/terraform-docs
```

- Jetzt werden die Tools, die im ersten Bau-Schritt erstellt wurden, **in das finale Image kopiert**. Das ist das zentrale Element eines Multi-Stage Builds: Man baut Tools in einer ersten „sauberen“ Umgebung und kopiert nur das Nötigste ins Endprodukt.

---

```dockerfile
COPY azure-pipelines-agent/start.sh .
RUN chmod +x start.sh
```

- Ein Skript namens `start.sh` wird in das Container-Verzeichnis kopiert und ausführbar gemacht. Dieses Skript startet später den Container.

---

**Start-Befehl**

```dockerfile
ENTRYPOINT ["./start.sh"]
```

- Dieser Befehl sagt: Wenn der Container gestartet wird, dann **führe das `start.sh`-Skript aus**. Das ist der Einstiegspunkt des Containers – quasi der „Startknopf“.

---

**Zusammenfassung**

Was macht dieses Dockerfile?

1. **Baut in einer ersten Phase** mehrere CLI-Tools (Databricks, yq, Terraform …).
1. **Erstellt ein sauberes Container-Image** mit allem, was gebraucht wird.
1. Installiert zusätzlich Dokumentationstools (MkDocs), Azure CLI, PowerShell usw.
1. **Kopiert nur das Nötige** vom Bau in das finale Image.
1. Führt beim Starten des Containers ein Skript aus (`start.sh`), um den Agent/Prozess zu starten.

---

**agent\BuildEnvironment.yml**
> Hinweis: Hier müssen die todo_ werte ersetzt werden.
```yml
pool:
  vmImage: ubuntu-latest
  
trigger: none

variables:
## Azure
- name: LOCATION
  value: todo_LOCATION  ## Enter your Azure location here
- name: RESOURCEGROUP
  value: todo_RESOURCEGROUP ## Enter your Azure Resourcegroup here -> Must be Unique
- name: CONTAINERREGISTRYNAME
  value: todo_CONTAINERREGISTRYNAME  ## Enter your Container Registry here -> Must be Unique
- name: ENVIRONMENT
  value: todo_ENVIRONMENT ## Enter your Container Environment here -> Must be Unique
- name: JOBNAME
  value: azure-pipelines-agent-job-we001  ## Enter your Jobname here
- name: PLACEHOLDERJOBNAME
  value: placeholder-agent-job-we001 ## Enter your Placeholder Job Name here
- name: CONTAINERIMAGENAME
  value: azure-pipelines-agent:1.0.$(Build.BuildId)
## Network
## https://learn.microsoft.com/en-us/azure/container-apps/vnet-custom-internal?tabs=bash%2Cazure-cli&pivots=azure-portal
- name: USESUBNET
  value: false
- name: INFRASTRUCTURESUBNETID
  value: todo_INFRASTRUCTURESUBNETID  ##/subscriptions/<tbd_subscription_id>/resourceGroups/<tbd_resourceGroupName>/providers/Microsoft.Network/virtualNetworks/<tbd_virtualNetworkName>/subnets/<tbd_subnetName>
- name: INTERNALROUTING
  value: true ## true or false
## DevOps  
- name: ORGANIZATIONURL
  value: todo_ORGANIZATIONURL  ## Enter your Azure Organization URL
- name: AZPPOOL
  value: SelfHostedContainerAgents
## Control
- name: isImageBuild
  value: $[eq(variables['imageBuild'],'true')]

steps:
- checkout: self
  path: docs
    
- task: AzureCLI@2
  displayName: az containerapp env create || az acr create
  condition: and(succeeded(),eq(variables.isImageBuild,false))
  enabled: true 
  inputs:
    azureSubscription: todo_ServiceConnection
    addSpnToEnvironment: true    
    scriptType: bash
    scriptLocation: inlineScript    
    inlineScript: |
      if [ "$USESUBNET" = "true" ]; then
        echo "##[group]az containerapp env create with network"
          az containerapp env create \
            --name "$ENVIRONMENT" \
            --resource-group "$RESOURCEGROUP" \
            --location "$LOCATION" \
            --infrastructure-subnet-resource-id "$INFRASTRUCTURESUBNETID" \
            --internal-only "$INTERNALROUTING" \
            --logs-destination none
        echo "##[endgroup]"
      else
        echo "##[group]az containerapp env create"
          az containerapp env create \
            --name "$ENVIRONMENT" \
            --resource-group "$RESOURCEGROUP" \
            --location "$LOCATION" \
            --logs-destination none
        echo "##[endgroup]"
      fi
      echo "##[group]az acr create"
        az acr create \
        --name "$CONTAINERREGISTRYNAME" \
        --resource-group "$RESOURCEGROUP" \
        --location "$LOCATION" \
        --sku Basic \
        --admin-enabled true
        echo "##[endgroup]"

- task: AzureCLI@2
  displayName: az acr build
  enabled: true 
  inputs:
    azureSubscription: todo_ServiceConnection
    addSpnToEnvironment: true    
    scriptType: bash
    workingDirectory: $(Pipeline.Workspace)/docs/docker/
    scriptLocation: inlineScript    
    inlineScript: |
      az acr build \
          --registry "$CONTAINERREGISTRYNAME" \
          --resource-group "$RESOURCEGROUP" \
          --image "$CONTAINERIMAGENAME" \
          --file "Dockerfile" \
          .

- task: AzureCLI@2
  displayName: az containerapp job create || az containerapp job start || az containerapp job delete
  condition: and(succeeded(),eq(variables.isImageBuild,false)) 
  enabled: true 
  inputs:
    azureSubscription: todo_ServiceConnection
    addSpnToEnvironment: true    
    scriptType: bash
    scriptLocation: inlineScript    
    inlineScript: |
      echo "##[group]az containerapp job create"
      az containerapp job create -n "$PLACEHOLDERJOBNAME" -g "$RESOURCEGROUP" --environment "$ENVIRONMENT" \
        --trigger-type Manual \
        --replica-timeout 300 \
        --replica-retry-limit 0 \
        --replica-completion-count 1 \
        --parallelism 1 \
        --image "$CONTAINERREGISTRYNAME.azurecr.io/$CONTAINERIMAGENAME" \
        --cpu "2.0" \
        --memory "4Gi" \
        --secrets "personal-access-token=$(AZP_TOKEN)" "organization-url=$ORGANIZATIONURL" \
        --env-vars "AZP_TOKEN=secretref:personal-access-token" "AZP_URL=secretref:organization-url" "AZP_POOL=$AZPPOOL" "AZP_PLACEHOLDER=1" "AZP_AGENT_NAME=placeholder-agent" \
        --registry-server "$CONTAINERREGISTRYNAME.azurecr.io"
      echo "##[endgroup]"
      echo "##[group]az containerapp job start"
      az containerapp job start -n "$PLACEHOLDERJOBNAME" -g "$RESOURCEGROUP"
      sleep 90s
      az containerapp job execution list \
        --name "$PLACEHOLDERJOBNAME" \
        --resource-group "$RESOURCEGROUP" \
        --output table \
        --query '[].{Status: properties.status, Name: name, StartTime: properties.startTime}'
      echo "##[endgroup]"
      echo "##[group]az containerapp job delete"   
      az containerapp job delete -n "$PLACEHOLDERJOBNAME" -g "$RESOURCEGROUP" --yes
      echo "##[endgroup]"

- task: AzureCLI@2
  displayName: az containerapp job create
  enabled: true 
  inputs:
    azureSubscription: todo_ServiceConnection
    addSpnToEnvironment: true    
    scriptType: bash
    scriptLocation: inlineScript    
    inlineScript: |
      az containerapp job create -n "$JOBNAME" -g "$RESOURCEGROUP" --environment "$ENVIRONMENT" \
        --trigger-type Event \
        --replica-timeout 1800 \
        --replica-retry-limit 0 \
        --replica-completion-count 1 \
        --parallelism 1 \
        --image "$CONTAINERREGISTRYNAME.azurecr.io/$CONTAINERIMAGENAME" \
        --min-executions 0 \
        --max-executions 10 \
        --polling-interval 30 \
        --scale-rule-name "azure-pipelines" \
        --scale-rule-type "azure-pipelines" \
        --scale-rule-metadata "poolName=$AZPPOOL" "targetPipelinesQueueLength=1" \
        --scale-rule-auth "personalAccessToken=personal-access-token" "organizationURL=organization-url" \
        --cpu "2.0" \
        --memory "4Gi" \
        --secrets "personal-access-token=$(AZP_TOKEN)" "organization-url=$ORGANIZATIONURL" \
        --env-vars "AZP_TOKEN=secretref:personal-access-token" "AZP_URL=secretref:organization-url" "AZP_POOL=$AZPPOOL" \
        --registry-server "$CONTAINERREGISTRYNAME.azurecr.io"
```

**agent\TestAgent.yml**
```yml
trigger:
- none

pool:
  name: SelfHostedContainerAgents

steps:
- pwsh: |
    $ErrorActionPreference = 'Stop'
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    try {
      Invoke-WebRequest -Method HEAD `
                        -Uri https://download.agent.dev.azure.com/agent/4.252.0/vsts-agent-win-x64-4.252.0.zip `
                        -TimeoutSec 5 `
                        -UseBasicParsing `
                        | Set-Variable response
      if ($response.StatusCode -lt 400) {
        Write-Host "Agent CDN is accessible. Status code: $($response.StatusCode)"
      } else {
        throw
      }
    } catch {
      Write-Host "##vso[task.logissue type=warning]Can't access download.agent.dev.azure.com. Please make sure the access is not blocked by a firewall."
      Write-Error "Agent CDN is inaccessible. Please make sure the access is not blocked by a firewall"
      $response | Format-List
    }
  displayName: 'Test download.agent.dev.azure.com access'
- script: |
    sleep 30
  displayName: 'Sleep'
  continueOnError: true
- script: |
    echo "Testing Azure CLI installation..."
    az --version
  displayName: 'Test Azure CLI'
  continueOnError: true
- script: |
    echo "Testing Databricks CLI installation..."
    databricks --version
  displayName: 'Test Databricks CLI'
  continueOnError: true
- script: |
    echo "Testing yq installation..."
    yq --version
  displayName: 'Test yq'
  continueOnError: true
- script: |
    echo "Testing Terraform installation..."
    terraform version
  displayName: 'Test Terraform'
  continueOnError: true
- script: |
    echo "Testing mkdocs installation..."
    mkdocs --version
  displayName: 'Test mkdocs'
  continueOnError: true
- script: |
    echo "Testing mkdocs-material extension..."
    pip3 list | grep mkdocs-material
  displayName: 'Test mkdocs-material'
  continueOnError: true
```

#### Pipelines erstellen
Erstelle zwei Pipelines in Azure DevOps:

1. **BuildEnvironment.yml** für die Pipeline `SetupBuildAgent` im Ordner `Orga`. 
  Füge zwei Variablen hinzu:
    - **imageBuild** mit dem Wert **false**. Wähle die Option (Die Benutzer können diesen Wert bei der Ausführung dieser Pipeline überschreiben.)
    - **AZP_TOKEN** mit dem Wert aus dem Abschnitt **Zugriffstoken** . Wähle die Option (Die Benutzer können diesen Wert bei der Ausführung dieser Pipeline überschreiben.)
1. **TestAgent.yml** für die Pipeline `TestBuildAgent` im Ordner `Orga`.

#### Variablen im BuildEnvironment.yml

Ersetze die `todo_` Werte mit den entsprechenden Informationen in Ihrer `BuildEnvironment.yml`-Datei.

| Name | Wert | Erklärung | Beispiel | 
| --- | --- | --- | --- |
| LOCATION | Zu definieren | Die Azure Region, in der die Containerlösung betrieben werden soll. | westeurope |
| RESOURCEGROUP | Zu definieren | Die Ressourcengruppe, in der die Lösung deployt wird. | shared |
| CONTAINERREGISTRYNAME | Zu definieren | Der Name der Container Registry, die eingerichtet werden soll. **Dieser Name muss Einzigartig sein.**| cragentwe001 |
| ENVIRONMENT | Zu definieren | Das zu erstellende Environment. Name muss eindeutig sein.  **Dieser Name muss Einzigartig sein.**| caeagentwe001 |
| JOBNAME | azure-pipelines-agent-job-we001 | Der Anzeigename des Container-Jobs. | azure-pipelines-agent-job-we001 |
| PLACEHOLDERJOBNAME | placeholder-agent-job-we001 | Der Anzeigename des Platzhalter-Jobs. | placeholder-agent-job-we001 |
| CONTAINERIMAGENAME | azure-pipelines-agent:1.0.$(Build.BuildId) | Der Name des in der Container Registry gebauten Images. | azure-pipelines-agent:1.0.$(Build.BuildId) |
| USESUBNET | false | Steuert die Netzwerkeinstellung; `false` für öffentlich, `true` für netzwerkisoliert. | false |
| INFRASTRUCTURESUBNETID | Zu definieren | Die Azure Netzwerk-ID, in die die Containerlösung integriert wird. [Networking in Azure Container Apps environment](https://learn.microsoft.com/en-us/azure/container-apps/networking)| /subscriptions/<tbd_subscription_id>/resourceGroups/<tbd_resourceGroupName>/providers/Microsoft.Network/virtualNetworks/<tbd_virtualNetworkName>/subnets/<tbd_subnetName> |
| INTERNALROUTING | true | Gibt an, dass die Umgebung nur über einen internen Load Balancer verfügt und keine öffentliche statische IP-Ressource besitzt. Muss `INFRASTRUCTURESUBNETID` angeben, wenn aktiviert. | true |
| ORGANIZATIONURL | Zu definieren | Die URL der Azure DevOps Organisation, in der die Agents eingerichtet werden. | https://dev.azure.com/<Organisation> |
| AZPPOOL | SelfHostedContainerAgents | Der Name des Agentpools. | SelfHostedContainerAgents |
| isImageBuild | Pipeline Variable | Gibt an, ob es sich um ein Initial-Setup (`false`) oder um den Bau einer neuen Image-Version (`true`) handelt. | An der Pipeline zu definieren |
| ServiceConnection | Zu definieren | Die eingerichtete Service-Connection in Azure DevOps. Kommt mehrfach vor. | ServiceConnection_Name |

#### Ausführen der Pipelines
1. Führe die **SetupBuildAgent** Pipeline aus und beobachte, ob alle Ressourcen in der Azure Resource Group angelegt werden.
1. Im Anschluss an die Ausführung der Pipeline **SetupBuildAgent** den Wert **imageBuild** auf `true` stellen.
1. Führe die `TestBuildAgent` Pipeline aus und überprüfe, ob ein selbst gehosteter Agent verwendet wird.