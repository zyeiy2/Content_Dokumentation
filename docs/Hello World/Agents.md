#  Agents 

## Software

Der Azure Pipelines-Agent-Pool bietet mehrere VM-Images zur Auswahl, die jeweils eine breite Palette von Tools und Software enthalten.

| Image | YAML Label | Included Software | 
| --- | --- | --- | 
| Ubuntu 22.04 | `ubuntu-latest` or `ubuntu-22.04` | [ubuntu-22.04] |
| Ubuntu 20.04 | `ubuntu-20.04` | [ubuntu-20.04] | 
| macOS 13 [beta] | `macos-13` or `macos-13-xl`| [macOS-13] | 
| macOS 12 | `macos-latest`, `macos-latest-xl`, `macos-12`, or `macos-12-xl`| [macOS-12] | 
| macOS 11 | `macos-11`| [macOS-11] | 
| Windows Server 2022 | `windows-latest` or `windows-2022` | [windows-2022] | 
| Windows Server 2019 | `windows-2019` | [windows-2019] | 


[ubuntu-22.04]: https://github.com/actions/runner-images/blob/main/images/ubuntu/Ubuntu2204-Readme.md
[ubuntu-20.04]: https://github.com/actions/runner-images/blob/main/images/ubuntu/Ubuntu2004-Readme.md
[windows-2022]: https://github.com/actions/runner-images/blob/main/images/windows/Windows2022-Readme.md
[windows-2019]: https://github.com/actions/runner-images/blob/main/images/windows/Windows2019-Readme.md
[macOS-11]: https://github.com/actions/runner-images/blob/main/images/macos/macos-11-Readme.md
[macOS-12]: https://github.com/actions/runner-images/blob/main/images/macos/macos-12-Readme.md
[macOS-13]: https://github.com/actions/runner-images/blob/main/images/macos/macos-13-Readme.md


## Pipeline umbau
Ändert die **Build Pipeline** wie folgt:

### 🏗️ ändern 
```yaml
trigger: none
```
```yaml
pool:
  vmImage: windows-latest
```

### build.yml
```yaml
trigger: none

pool:
  vmImage: windows-latest

steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'
```


## Links
[Microsoft-hosted agents for Azure Pipelines - Azure Pipelines | Microsoft Docs](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#use-a-microsoft-hosted-agent)    
[Azure DevOps Services – Preise | Microsoft Azure](https://azure.microsoft.com/de-de/pricing/details/devops/azure-devops-services/)  
[Runner Images - Actions | GitHub](https://github.com/actions/runner-images/tree/main)