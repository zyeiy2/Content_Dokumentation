# HandsOn




### Voraussetzungen

Stellen sicher, dass die folgende Voraussetzungen erfüllt sind:

- Azure Umgebung
- Netzwerk
- Azure DevOps Instanz
- Agentpools
- Azure DevOps-Zugriffstokens
- Variablen im `BuildEnvironment.yml`

#### Azure Umgebung

Erstelle ein [kostenloses Azure-Konto](https://azure.microsoft.com/de-de/free/) wenn keins vorhanden ist.

##### Azure Resource Provider

In der Subscription, wo die Lösung deployt wird, müssen die folgenden Ressourcen-Provider registriert sein:  
- Microsoft.App  
- Microsoft.OperationalInsights  

Im ,[Azure portal | Register resource provider](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider-1) findest du eine Anleitung, wie die Azure Resource Provider registriert werden können.

##### Netzwerk 
Das Präfix der Netzwerk-Subnetzadresse erfordert einen minimalen CIDR-Bereich /23 für die Verwendung mit Container-Apps. 

#### Azure DevOps Instanz
Erstelle eine [kostenlose Azure-DevOps Instanz](https://azure.microsoft.com/de-de/free/devops/) wenn keine vorhanden ist.

##### Zugriffstoken

1. `todo_ORGANIZATION_URL` -> die URL des Azure DevOps Projekts (`https://dev.azure.com/ProjektName`)
2. `todo_AZP_TOKEN` -> persönliches Zugriffstoken (PAT)

Um ein PAT zu erstellen:

1. Wähle in Azure DevOps die Benutzereinstellungen neben Ihrem Profilbild in der oberen rechten Ecke aus.  
2. Wähle "Personal Access Tokens" aus.  
3. Wähle auf der Seite "Persönliche Zugriffstoken" "Neues Token" aus und gebe die folgenden Werte ein:  

| Einstellung| Wert|
| --- | --- |
| Name| Gib einen Namen für den Token an. |
| Organisation| Wähle die Organisation aus. |
| Bereiche| Wähle **Benutzerdefiniert** aus. |
| Alle Bereiche anzeigen | Wähle  **Alle Bereiche anzeigen** aus. |
| Agentpools (Lesen und Verwalten) | Wähle Agentpools (Lesen und Verwalten) aus. |

4. Klicke auf Erstellen und kopiereden Tokenwert an einen sicheren Speicherort.

##### Agentpools

1. Erweiter im Azure DevOps-Projekt die linke Navigationsleiste und wähle "Projekteinstellungen" aus.
2. Wähle unter dem Abschnitt "Pipelines" die Option "Agentpools" aus.
3. Wähle "Pool hinzufügen" aus und gebe folgende Werte ein:

| Einstellung| Wert|
| -- | -- |
| Pool, der verknüpft werden soll| Wähle **Neu** aus.|
| Pooltyp| Wähle **selbst gehostet** aus. |
| Name| Gebe  **SelfHostedContainerAgents** ein. |
| Gewähren der Zugriffsberechtigung für alle Pipelines | Aktiviere dieses Kontrollkästchen. |

##### Service Connection 

###### App Registration
1. Erstellen einer [App-Registration](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app) und anlegen eines [Client Secrets](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app#add-a-client-secret). 
1. Vergeben von Contributor Rechten auf der Ressourcengruppe oder Subscription. 
1. Analge einer mit Service Connection `Client Secret` oder `Federated Credentials`.

###### Service Connection - Client Secret

1. App Registration als Service Connection in Azure DevOps anlegen.[Create an Azure Resource Manager service connection that uses an existing service principal](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure?view=azure-devops#create-an-azure-resource-manager-service-connection-that-uses-an-existing-service-principal)  
