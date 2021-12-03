# YAML Schemareferenz

## Pipeline structure

Eine Pipeline umfasst eine oder mehrere Stages, die einen CI/CD-Prozess beschreiben. Stages sind die Hauptbereiche in einer Pipeline. Die Stages "Build this app", "Run these tests" und "Deploy to preproduction" sind gute Beispiele.

Eine Phase sind ein oder mehrere Jobs, bei denen es sich um Arbeitseinheiten handelt, die demselben Computer zugewiesen werden können. Sie können Stages und Jobs in Abhängigkeitsdiagrammen anordnen. Beispiele hierfür sind "Diese Phase vor dieser Phase ausführen" und "Dieser Job hängt von der Ausgabe dieses Jobs ab".

Ein Job ist eine lineare Abfolge von Steps. Steps können Aufgaben, Skripts oder Verweise auf externe Vorlagen sein.

This hierarchy is reflected in the structure of a YAML file like:

- Pipeline
    - Stage A
        - Job 1
            - Step 1.1
            - Step 1.2
            - ...
        - Job 2
            - Step 2.1
            - Step 2.2
            - ...
    - Stage B
        - ...

Für einfache Pipelines sind nicht alle diese Ebenen erforderlich. Beispielsweise können Sie in einem Einzelauftragsbuild die Container für Phasen und Aufträge weglassen, da es nur Steps gibt. Da viele in diesem Abschnitt gezeigte Optionen nicht erforderlich sind und über gute Standardwerte verfügen, ist es unwahrscheinlich, dass Ihre YAML-Definitionen alle enthalten.

**Beispiel einer Pipeline:** 
![Azure DevOps](Bild11.png)



### Trigger
```yaml
trigger: 
- main
```

Ein Trigger gibt an wann eine Pipeline gestartet wird. 
Möglichkeiten sind:
- Push trigger
- Pull request trigger
- Scheduled trigger
- Pipeline trigger

###  Pool
```yaml
pool:
  vmImage: ubuntu-latest
```

Ein Pool gibt an, wo und auf welchem OS die Phase ausgeführt wird. Möglichkeiten sind: 
- Windows-latest
- ubuntu-latest
- macOS-latest
- Selfhosted

### Steps

```yaml
steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'

```

Steps sind eine Abfolge von Vorgängen. 
Alle Tasks unterstützen die folgenden Eigenschaften:   
- displayName  
- condition  
- continueonerror  
- enabled  
- env  
- timeoutInMinutes  

Eine Aufgabe ist das Herzstück der Pipeline, sie führt die gewünschte Funktion aus.
Möglichkeiten sind:  
- [PowerShell Task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/powershell?view=azure-devops)  
- [Copy Files Task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/copy-files?view=azure-devops&tabs=yaml)  
- [Azure PowerShell task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/deploy/azure-powershell?view=azure-devops)  
- [Publish Build Artifacts task](https://docs.microsoft.com/en-us/azure/devops/pipelines/artifacts/pipeline-artifacts?view=azure-devops&tabs=yaml)  
- [Python Script task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/python-script?view=azure-devops)  


## Links
[Catalog of the built-in tasks for build-release and Azure Pipelines & TFS - Azure Pipelines | Microsoft Docs
](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/?view=azure-devops)

[YAML schema - Azure Pipelines | Microsoft Docs
](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema) 

## Glossar

Begriff | Erklärung 
--- |---
Stage | Eine Phase ist eine Sammlung von zusammenhängenden Aufträgen. Standardmäßig werden Schritte sequentiell ausgeführt. Jede Stufe beginnt erst, wenn die vorhergehende Stufe abgeschlossen ist, es sei denn, über die Eigenschaft dependsOn wird etwas anderes angegeben.
Job | Ein Job ist eine Sammlung von Schritten, die von einem Agenten oder auf einem Server ausgeführt werden. Aufträge können unter bestimmten Bedingungen laufen und von früheren Aufträgen abhängen.
Step | Ein Step ist eine lineare Abfolge von Vorgängen, aus denen ein Auftrag besteht. Jeder Step läuft in einem eigenen Prozess auf einem Agenten und hat Zugriff auf den Pipeline-Arbeitsbereich auf einer lokalen Festplatte.
Task | Tasks sind die Bausteine einer Pipeline.
Resources | Eine Ressource ist ein externer Dienst, der als Teil der Pipeline genutzt wird.
Triggers | Ein Trigger legt fest, welche Zweige oder Zeiten einen Build  auslösen.
Publish | Das Schlüsselwort publish ist eine Abkürzung für die Aufgabe Publish Pipeline Artifact. Mit dieser Aufgabe wird eine Datei oder ein Ordner als Pipeline-Artefakt veröffentlicht (hochgeladen), das von anderen Aufträgen und Pipelines verwendet werden kann.
Download | Das Schlüsselwort download ist eine Abkürzung für die Aufgabe Pipeline-Artefakte herunterladen. Die Aufgabe lädt Artefakte herunter, welche mit dem aktuellen Lauf oder von einer anderen Azure-Pipeline, die als Pipeline-Ressource zugeordnet ist, verbunden sind.
Checkout | Aufträge, welche nicht der Bereitstellung dienen, checken automatisch den Quellcode aus. 

