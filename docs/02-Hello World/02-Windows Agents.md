#  Agents 

## Software

Der Azure Pipelines-Agent-Pool bietet mehrere VM-Images zur Auswahl, die jeweils eine breite Palette von Tools und Software enthalten.

 Image | Spezifikation des klassischen Editor-Agents | YAML-VM-Imagebezeichnung | Enthaltene Software |
 --- | --- | --- | --- 
Windows Server 2022 mit Visual Studio 2022 |  windows-2022 | ```windows-2022``` | [Link](https://github.com/actions/virtual-environments/blob/main/images/win/Windows2022-Readme.md) 
 Windows Server 2019 mit Visual Studio 2019|windows-2019 | ```windows-latest``` ODER ```windows-2019``` | [Link](https://github.com/actions/virtual-environments/blob/main/images/win/Windows2019-Readme.md) 
Ubuntu 20.04 |ubuntu-20.04 |```ubuntu-latest``` ODER ```ubuntu-20.04``` | [Link](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu2004-README.md) 
Ubuntu 18.04 |ubuntu-18.04 |```ubuntu-18.04```| [Link](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu1804-README.md) 
macOS 11 Big Sur |macOS-11 |```macOS-11``` | [Link](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-11-Readme.md) 
macOS X Mojave 10.14 |macOS-10.14 |```macOS-10.14``` | [Link](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-10.14-Readme.md) 
macOS X Catalina 10.15 |macOS-10.15 |```macOS-latest``` ODER ```macOS-10.15``` | [Link](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-10.15-Readme.md) 



## Pipeline umbau
Ändert die **Build Pipeline** wie folgt:

### ändern
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
[Microsoft-hosted agents for Azure Pipelines - Azure Pipelines | Microsoft Docs
](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#use-a-microsoft-hosted-agent)