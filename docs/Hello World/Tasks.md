# Tasks

Im Kontext von Azure DevOps Pipelines bezieht sich ein "Task" auf eine einzelne Arbeitseinheit oder Anweisung, die in einer Build- oder Release-Pipeline ausgeführt wird. Tasks repräsentieren die Schritte, die zur Durchführung einer bestimmten Aktion während des Build- oder Bereitstellungsprozesses erforderlich sind. Diese können Befehle, Skripte, Buildschritte, Bereitstellungsschritte oder andere Aktionen sein.

Jede Task führt eine spezifische Aufgabe innerhalb des Pipelines aus. Beispiele für Tasks könnten sein:

1. **Build-Task:** Kompilieren des Quellcodes in ausführbare Artefakte.
2. **Copy-Files-Task:** Kopieren von Dateien an einen bestimmten Speicherort.
3. **Publish-Test-Results-Task:** Veröffentlichen von Testergebnissen.
4. **Deploy-Task:** Bereitstellen von Anwendungen oder Artefakten auf eine Ziellumgebung.

Azure DevOps Pipelines bietet eine Vielzahl von vordefinierten Tasks für häufige Szenarien. Darüber hinaus können Sie auch benutzerdefinierte Tasks erstellen oder vorhandene Erweiterungen aus dem Azure DevOps Marketplace verwenden, um zusätzliche Funktionalitäten zu integrieren.

Tasks werden in einer YAML-Datei oder visuell in der Benutzeroberfläche von Azure DevOps definiert. Sie sind die grundlegenden Bausteine einer Pipeline und werden in der Reihenfolge ausgeführt, in der sie in der Konfiguration angegeben sind. Durch die Verkettung von Aufgaben können Sie komplexe Build- und Bereitstellungsworkflows erstellen.

Wir werden für das **Publish-Processes.ps1** Powershell einen [PowerShell@2](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/powershell?view=azure-devops) Task nutzen. 


## Pipeline umbau
Ändert die **Build Pipeline** wie folgt:

### ➕ hinzufügen
```yaml
- task: PowerShell@2
  displayName: Export Logs
  inputs:
    targetType: inline
    script: |
      $(Pipeline.Workspace)/code/scripts/Publish-Processes.ps1 -Name "Processes.txt" -Path $(Build.ArtifactStagingDirectory)
```

### build.yml

```yaml
resources:
 repositories:
   - repository: code
     type: git
     name: code

trigger: none

pool:
  vmImage: windows-latest

steps:
- checkout: self
  path: schulung
- checkout: code
  path: code

- task: PowerShell@2
  displayName: Print Variable
  inputs:
    targetType: inline
    script: |
      echo $(Pipeline.Workspace)
      echo $(Agent.BuildDirectory)

- task: PowerShell@2
  displayName: Print Tree Path
  inputs:
    targetType: inline
    script: |
      tree $(Pipeline.Workspace) /a

- task: PowerShell@2
  displayName: Export Logs
  inputs:
    targetType: inline
    script: |
      $(Pipeline.Workspace)/code/scripts/Publish-Processes.ps1 -Name "Processes.txt" -Path $(Build.ArtifactStagingDirectory)
```

## Links
[Catalog of the built-in tasks for build-release and Azure Pipelines & TFS - Azure Pipelines | Microsoft Docs ](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/?view=azure-devops)  
[PowerShell@2](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/powershell?view=azure-devops)  
[Logging commands - Azure Pipelines | Microsoft Docs](https://docs.microsoft.com/en-us/azure/devops/pipelines/scripts/logging-commands?view=azure-devops&tabs=bash)