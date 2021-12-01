# Lösung

## Invoke-Dokumentation.yml
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

## build.yml
```yaml
resources:
 repositories:
   - repository: code
     type: git
     name: code

trigger: none

pool:
  vmImage: ubuntu-latest

steps:
- checkout: code
  path: code

- template: templates/Invoke-Dokumentation.yml@code

- task: UsePythonVersion@0
  displayName: 'Install latest Python Version'
  inputs:
    versionSpec: '3.x'
    addToPath: true
    architecture: 'x64'

- task: PowerShell@2
  displayName: 'Install mkDocs and relevant modules'
  inputs:
    targetType: 'inline'
    script: |
      python -m pip install mkdocs --user
      python -m pip install mkdocs-material --user

- task: PowerShell@2
  displayName: 'Build Site'
  inputs:
    targetType: 'inline'
    script: |
      python -m mkdocs build --clean
    workingDirectory: $(Pipeline.Workspace)/Content_Dokumentation

- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: site'
  inputs:
    artifactName: 'site'
    PathtoPublish: $(Pipeline.Workspace)/Content_Dokumentation/site
```


## release.yml
Variables

Name | Value 
---|---
container | **`$web**
sasToken | Value of SAS Token
storageAccount | Value of Storage Account Name

```yaml
resources:
 pipelines:
   - pipeline: build
     project: Schulung
     source: build
     branch: main
     trigger:
      branches:
         include:
           - main
           
trigger: none

pool:
  vmImage: windows-latest

steps:
- checkout: none
- download: build

- task: PowerShell@2
  displayName: Export to Azure Storage
  inputs:
    targetType: inline
    script: |
      Write-Host "Start delte https://$(storageAccount).blob.core.windows.net/$(container)"
      azcopy rm "https://$(storageAccount).blob.core.windows.net/$(container)/$(sasToken)" --recursive=true
      Write-Host "Finish delte https://$(storageAccount).blob.core.windows.net/$(container)"
      Write-Host "Start upload content https://$(storageAccount).blob.core.windows.net/$(container)"
      azcopy copy $(Pipeline.Workspace)/build/site/* "https://$(storageAccount).blob.core.windows.net/$(container)/$(sasToken)" --recursive=true
      Write-Host "Finish upload content https://$(storageAccount).blob.core.windows.net/$(container)"
```
