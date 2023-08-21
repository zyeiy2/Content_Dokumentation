# ORAYLIS
Anbei findest du eine Übersicht von häufig verwendeten Technologien bei ORAYLIS und einen kurzen Überblick, was wichtig ist. Außerdem findest du immer noch am Ende ein paar wichtige Links dazu.

## Azure Databricks
Hier ist eine Schritt-für-Schritt-Anleitung, wie Sie Databricks-Notebooks aus einem Azure DevOps-Repository heraus in Azure Databricks bereitstellen können:

**Voraussetzungen:**
1. Du benötigst ein Azure DevOps-Konto mit Zugriff auf das gewünschte Repository.
2. Du musst über einen Azure Databricks-Arbeitsbereich verfügen.

**Schritte:**

1. **Erstelle Notebooks im Azure DevOps-Repository:**
   - Lege im Azure DevOps-Repository den Ordner oder das Projekt an, in dem du deine Databricks-Notebooks speichern möchtest.
   - Füge die gewünschten Databricks-Notebooks in diesen Ordner hinzu und achte darauf, dass sie im Repository versioniert werden.

2. **Erstelle eine Azure DevOps-Build-Pipeline:**
   - Gehe zu deinem Azure DevOps-Projekt und navigiere zur Übersicht der Pipelines.
   - Klicke auf "New Pipeline", um eine neue Build-Pipeline zu erstellen.
   - Wähle deine Quelle aus, z.B. das Git-Repository, das die Databricks-Notebooks enthält.
   - Konfiguriere die Build-Pipeline nach deinen Anforderungen. Du kannst hier Schritte wie das Klonen des Repositories und das Hochladen der Notebooks in den Build-Arbeitsordner einfügen.

3. **Füge das Databricks CLI-Skript zur Pipeline hinzu:**
   - Füge einen Schritt zur Build-Pipeline hinzu, der das Databricks CLI-Skript ausführt. Dieses Skript wird verwendet, um die Notebooks in Azure Databricks zu importieren.
   - Das Databricks CLI-Skript könnte zum Beispiel so aussehen:
     ```bash
     databricks workspace import_dir /pfad/zum/lokalen/ordner /zielordner
     ```
   - Stelle sicher, dass du die benötigten Zugriffstoken oder Authentifizierungsinformationen für den Zugriff auf den Databricks-Arbeitsbereich in der Pipeline hinterlegst.

4. **Konfiguriere Trigger und Ausführung:**
   - Lege Trigger für die Pipeline fest, um sie automatisch auszuführen, wenn sich etwas ändert, zum Beispiel im Git-Repository.
   - Starte die Build-Pipeline, um die Databricks-Notebooks in den Databricks-Arbeitsbereich zu importieren.

5. **Überprüfe in Databricks:**
   - Melde dich im Azure Databricks-Portal an.
   - Navigiere zum entsprechenden Arbeitsbereich und Ordner, in den die Notebooks importiert wurden.
   - Überprüfe, ob die Notebooks korrekt importiert wurden und ihre Inhalte stimmen.

Diese Anleitung gibt dir eine grundlegende Vorstellung davon, wie du Databricks-Notebooks aus einem Azure DevOps-Repository heraus in Azure Databricks bereitstellen kannst. Beachte, dass die genauen Schritte je nach deiner individuellen Umgebung, deinen Anforderungen und den verwendeten Authentifizierungsmethoden variieren können.

**Links für Azure Databricks**  
[Continuous integration and delivery on Azure Databricks using Azure DevOps](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/ci-cd/ci-cd-azure-devops)

## Azure Synapse 

Diese Anleitung zeigt, wie du die Bereitstellungsaufgabe (deployment task) in Azure Synapse mithilfe von Azure DevOps konfigurierst. Die Bereitstellungsaufgabe unterstützt drei Arten von Operationen: nur validieren, bereitstellen und validieren, sowie bereitstellen.

**Visual Studio Market Place Extension**
Verwende die Visual Studio Extension <a href="https://marketplace.visualstudio.com/items?itemName=AzureSynapseWorkspace.synapsecicd-deploy">Synapse workspace deployment</a>, um Elemente in deinem Azure Synapse-Arbeitsbereich bereitzustellen. Elemente, die du bereitstellen kannst, umfassen Datensätze, SQL-Skripte und Notizbücher, Spark-Jobdefinitionen, Integrationslaufzeit, Datenflüsse, Anmeldedaten und andere Artefakte im Arbeitsbereich.

**Hinweis:** Diese Erweiterung für die <a href="https://marketplace.visualstudio.com/items?itemName=AzureSynapseWorkspace.synapsecicd-deploy">Synapse workspace deployment</a> ist nicht rückwärtskompatibel. Stelle sicher, dass die neueste Version installiert und verwendet wird. Du kannst unter <a href="https://marketplace.visualstudio.com/items?itemName=AzureSynapseWorkspace.synapsecicd-deploy">Release notes/ roadmap</a> im auge behalten.

1. **Validieren:** Dieser Schritt validiert die Synapse-Artefakte im Branch mit der Aufgabe und generiert die Arbeitsbereichsvorlagen- und Parameter-Vorlagen-Dateien. Diese Validierungsoperation funktioniert nur in der YAML-Pipeline. Hier ist ein Beispiel für eine YAML-Datei:

```yaml
pool:
  vmImage: ubuntu-latest

resources:
  repositories:
  - repository: <Repositoriumsname>
    type: git
    name: <Name>
    ref: <Benutzer/Kollaborations-Branch>

steps:
  - checkout: <Name>
  - task: Synapse workspace deployment@2
    continueOnError: true    
    inputs:
      operation: 'validate'
      ArtifactsFolder: '$(System.DefaultWorkingDirectory)/ArtifactFolder'
      TargetWorkspaceName: '<Zielarbeitsbereichsname>'
```

2. **Validieren und Bereitstellen:** Mit dieser Option kannst du den Arbeitsbereich direkt aus dem Branch mit dem Artefakt-Stammverzeichnis bereitstellen.

**Bereitstellen:** Die Eingaben für die Bereitstellungsoperation umfassen die Synapse-Arbeitsbereichsvorlage und die Parameter-Vorlage. Diese können nach der Veröffentlichung im Branch oder nach der Validierung erstellt werden. Die Vorgehensweise ist wie in Version 1.x.

**Anleitung:**

1. Wähle in dem Task den Operationstyp "Bereitstellen" aus.

2. Gebe den Pfad zum Template an um das Template auszuwählen.

3. Gebe den Pfad zu den Template Parametern.

4. Gebe die Details des Synapse Workspace an.

5. Vor und nach dem Deployment müssen die Trigger entsprechend behandelt werden.

```
  - task: AzurePowerShell@5
    displayName: "Disable Trigger"
    enabled: true
    continueOnError: false
    inputs:
      scriptType: "inlineScript"
      azureSubscription: ${{ parameters.serviceconnectionname }}
      azurePowerShellVersion: latestVersion
      inline: |
        foreach ($trigger in Get-AzSynapseTrigger -WorkspaceName $(WorkspaceName) -ErrorAction Stop) {
          Stop-AzSynapseTrigger -WorkspaceName $(WorkspaceName) -Name $trigger.Name -ErrorAction Stop
        }
        
  - task: AzureSynapseWorkspace.synapsecicd-deploy.synapse-deploy.Synapse workspace deployment@1
    displayName: 'Deploy Synapase Workspace'
    enabled: true
    continueOnError: false
    inputs:
      TemplateFile: $(Pipeline.Workspace)\build\files\arm\TemplateForWorkspace.json
      ParametersFile: $(Pipeline.Workspace)\build\files\arm\TemplateParametersForWorkspace.json
      azureSubscription: ${{ parameters.serviceconnectionname }}
      ResourceGroupName: $(ResourceGroupName)
      TargetWorkspaceName: $(WorkspaceName)
      DeleteArtifactsNotInTemplate: $(FullLoad)
      OverrideArmParameters: >-
        -workspaceName $WorkspaceName)
        -properties_typeProperties_tenant $(ServicePrincial_TenantId)
        -properties_typeProperties_servicePrincipalId $(ServicePrincial_ApplicationId)
        -$(KeyVault)_properties_typeProperties_baseUrl https://$(KeyVault).vault.azure.net/
        -$(WorkspaceName)-WorkspaceDefaultStorage_properties_typeProperties_url https://$(StorageAccountNamePrimary).dfs.core.windows.net

  - task: AzurePowerShell@5
    displayName: "Enable Trigger"
    enabled: true
    continueOnError: false
    inputs:
      azureSubscription: ${{ parameters.rmserviceconnectionname }} 
      azurePowerShellVersion: latestVersion
      scriptType: 'filePath' 
      scriptPath: $(Pipeline.Workspace)\release\azurePipelines\scripts\Invoke-EnableSynapseTrigger.ps1
      scriptArguments: > 
        -WorkspaceName $(WorkspaceName)
        -PathTrigger $(Pipeline.Workspace)\build\files\trigger
```

**Links für Azure Synapse**  
[Continuous integration and delivery for an Azure Synapse Analytics workspace](https://learn.microsoft.com/en-us/azure/synapse-analytics/cicd/continuous-integration-delivery)

