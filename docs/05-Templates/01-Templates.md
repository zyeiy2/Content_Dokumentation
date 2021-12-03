# Templates

Templates helfen Aufgaben, welche immer wieder ausgeführt werden müssen, wiederverwendbar zu machen. 

Mögliche Szenarien:   
- Installieren von Dependencies  
- Änderungen an Azure Security   
- White-listen von IP Adressen für den zugriff einer Pipeline  
- Herunterladen von Artefakten  
- Kompilieren von Code  
- Release Steuerung   


## Aufgabe
1. Legt bitte im Code Repository einen Ordner **templates** an.
2. Erstellt bitte ein File mit dem Namen **Invoke-Dokumentation.yml**
3. Kopiert den Inhalt in das File und speichert es
```yaml
steps:
- task: PowerShell@2
  displayName: DownloadContent
  inputs:
    targetType: 'inline'
    script: |
      ## Force to create a zip file 
      $ZipFile = '$(Pipeline.Workspace)\main.zip'
      New-Item $ZipFile -ItemType File -Force
      ##$RepositoryZipUrl = https://github.com/zyeiy2/Content_Dokumentation/archive/refs/heads/main.zip
      $RepositoryZipUrl = "https://api.github.com/repos/zyeiy2/Content_Dokumentation/zipball/main" 
      # download the zip 
      Write-Host 'Starting downloading the GitHub Repository'
      Invoke-RestMethod -Uri $RepositoryZipUrl -OutFile $ZipFile 
      Write-Host 'Download finished'
      #Extract Zip File
      Write-Host 'Starting unzipping the GitHub Repository locally'
      Expand-Archive -Path $ZipFile -DestinationPath $(Pipeline.Workspace) -Force
      Write-Host 'Unzip finished'     
      # remove the zip file
      Write-Host 'Starting delete .zip file'
      Remove-Item -Path $ZipFile -Force
      Write-Host 'Delete .zip file finished'    
      # rename zip file
      Write-Host 'Starting rename folder'
      $folder = Get-ChildItem -Path $(Pipeline.Workspace) | Where-Object {$_.Name -CMatch 'Content_Dokumentation' }
      $folderName = '$(Pipeline.Workspace)\'+$folder.name
      Rename-Item -Path $folderName -NewName "Content_Dokumentation"
      Write-Host 'Rename folder finished'    
```
## Was macht das Template?
Das Template lädt aus dem [GitHub Repository](https://github.com/zyeiy2/Content_Dokumentation) die aktuelle Struktur herunter. Entpackt das .zip File und legt unter dem Pfad ```$(Pipeline.Workspace)/Content_Dokumentation``` die Markdown Files ab.



## Pipeline umbau

### hinzufügen 
```yaml
- template: templates/Invoke-Dokumentation.yml@code
```
### entfernen
```yaml
- task: PowerShell@2
  displayName: Print Variable
  inputs:
    targetType: inline
    script: |
      echo $(Pipeline.Workspace)
      echo $(Agent.BuildDirectory)
```
```yaml
- task: PowerShell@2
  displayName: Print Tree Path
  inputs:
    targetType: inline
    script: |
      tree $(Pipeline.Workspace) /a
```

```yaml
- task: PowerShell@2
  displayName: Export Logs
  inputs:
    targetType: inline
    script: |
      $(Pipeline.Workspace)/code/scripts/Publish-Processes.ps1 -Name "Processes.txt" -Path $(Build.ArtifactStagingDirectory)
```
```yaml
- task: PublishPipelineArtifact@1
  displayName: Publish Pipeline Artifact
  inputs:
    targetPath: $(Build.ArtifactStagingDirectory)
    artifactName: logs
```
```yaml 
- task: PowerShell@2
  displayName: Print Secret
  enabled: true
  inputs:
    targetType: inline
    script: |
      Write-Host "Value of PipelineVar_mySecret $(PipelineVar_mySecret)"
      Write-Host "Value of PipelineVar_myName $(PipelineVar_myName)"
      Write-Host "Value of VariableGroup_EnvName $(VariableGroup_EnvName)"
      Write-Host "Value of VariableGroup_User $(VariableGroup_User)"
      Write-Host "Value of VariableGroup_Password $(VariableGroup_Password)"
```
### build.yml
```yaml
resources:
 repositories:
   - repository: code
     type: git
     name: code

variables:
- group: DEV

trigger: none

pool:
  vmImage: windows-latest

steps:
- checkout: self
  path: schulung
- checkout: code
  path: code
  
- template: templates/Invoke-Dokumentation.yml@code
```

## Links
[Templates - Azure Pipelines | Microsoft Docs
](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops)