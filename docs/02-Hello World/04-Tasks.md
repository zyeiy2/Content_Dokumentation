# Tasks

Tasks in der Azure DevOps Pipeline sind das Herzstück. Es ist möglich aus den folgenden Task-Kategorien zu wählen anbei einige beispiele:  
- Build  
- Utility   
- Test  
- Package  
- Deploy  
- Tool  

Wir werden für das **Publish-Processes.ps1** Powershell einen [PowerShell@2](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/powershell?view=azure-devops) Task nutzen. 


## Pipeline umbau
Ändert die **Build Pipeline** wie folgt:

### hinzufügen
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