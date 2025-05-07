# Einordnung in Azure DevOps 

**Einordnung: Selbstgehostete Agents in Azure DevOps**

In Azure DevOps dienen **Build- und Release-Agents** dazu, automatisierte Pipelines auszuführen – sei es für Builds, Tests oder Deployments. Standardmäßig stellt Microsoft sogenannte **Microsoft-hosted Agents** zur Verfügung. Für viele Standardanwendungen reicht das aus.

**Selbstgehostete Agents** bieten jedoch mehr Kontrolle und Flexibilität. Sie sind besonders dann sinnvoll, wenn:

- **spezielle Abhängigkeiten oder Tools** benötigt werden, die nicht auf Microsoft-hosted Agents vorinstalliert sind.
- eine **konstante Umgebung** erforderlich ist (z. B. für Legacy-Projekte).
- **Performance** und **Kostenkontrolle** wichtig sind – etwa bei langen Builds oder hohem Durchsatz.
- der Zugriff auf **interne Ressourcen** nötig ist, z. B. private Netzwerke oder nicht öffentlich Resourcen.

Ein selbstgehosteter Agent ist im Grunde ein Dienst, der auf einer eigenen VM, einem physischen Server oder auch einem Container läuft. Er verbindet sich über einen sicheren Kanal mit Azure DevOps und wartet auf Aufgaben aus einer Pipeline. Dabei hat das Team die volle Kontrolle über Betriebssystem, installierte Software, Hardware-Ressourcen und Sicherheitsrichtlinien.

**Warum ist das wichtig?**

- Ihr könnt lokale Entwicklungsumgebungen sehr genau nachbilden.
- Ihr reduziert unnötige Fehler durch Umgebungsunterschiede.
- Ihr beschleunigt Feedback-Zyklen, weil Builds parallel oder gezielt auf spezialisierter Hardware laufen können.
- Ihr könnt eure Infrastruktur besser an CI/CD-Prozesse anpassen und automatisieren.

> Hinweis: Mit der Kontrolle kommt auch Verantwortung – etwa für Updates, Sicherheitspatches, Agent-Verfügbarkeit und Skalierung. 

**Vergleich: Microsoft-hosted vs. Selbstgehostete Agents**

| Merkmal| Microsoft-hosted Agent| Selbstgehosteter Agent|
| -- | -- | -- |
| **Bereitstellung**| Automatisch durch Microsoft| Manuell durch das Team|
| **Lebensdauer**| Kurzlebig (wird nach jedem Job gelöscht)| Kurzlebig (wird nach jedem Job gelöscht)|
| **Verwaltung**| Microsoft übernimmt Wartung und Updates| Eigenverantwortlich für Wartung und Updates|
| **Anpassbarkeit**| Eingeschränkt (vordefinierte Images)| Vollständig anpassbar (eigene Software & Konfiguration) |
| **Leistung**| Variabel (abhängig von Systemauslastung)| Kontrollierbar (abhängig von eigener Konfiguration)|
| **Kosten**| Abhängig von Nutzung und Parallelität| Eigene Infrastrukturkosten, aber keine Nutzungskosten|                                                
| **Netzwerkzugriff** | Eingeschränkt (kein Zugriff auf interne Netzwerke) | Voller Zugriff auf interne Ressourcen|
  
---  
  
**Wann sollte man welchen Agent-Typ wählen?**

**Microsoft-hosted Agents** sind ideal für:

  - Schnelles Setup und einfache Projekte
  - Standardisierte Builds ohne spezielle Anforderungen
  - Teams, die keine eigene Infrastruktur verwalten möchten oder müssen

**Selbstgehostete Agents** eignen sich besonders für:

  - Projekte mit speziellen Softwareanforderungen
  - Notwendigkeit des Zugriffs auf interne Netzwerke oder Ressourcen
  - Kontrolle über die Build-Umgebung und bessere Performance



