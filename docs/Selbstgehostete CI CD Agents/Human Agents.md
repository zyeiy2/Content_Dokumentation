# Human Agents 

Mission: Selfhosted – Azure DevOps als Live-Spiel

## Ziel des Spiels

Erlebt interaktiv den Ablauf eines Selfhosted Build Agent-Szenarios in Azure DevOps. 
Die Spielenden übernehmen die Rollen realer Komponenten und simulieren den Build- und Deploymentprozess.

**Rollen mit Beschreibung & Spieltext**

|Rolle|Aufgabe|Signal|
|--|--|--|
|Dockerfile Master|Erstellt und übergibt das Dockerfile, fügt bei Fehlern fehlende Komponenten hinzu.|Hier ist die Bauanleitung!" / "Ich ergänze: pip install mkdocs!"|
|Container Registry|Baut das Image basierend auf dem Dockerfile und speichert es.|"Image erfolgreich gebaut und gespeichert!"|
|Placeholder Agent|Erkennt eingehende Azure DevOps Pipelines und startet den Job Agent.|"Pipeline erkannt – starte Job Agent!"|
|Job Agent| Holt das Image aus der Registry, registriert sich im Agent Pool, führt die Pipeline aus.|"Registriere mich im Agent Pool!" / "Installiere Komponente X!"|
|Agent Pool|Registriert Job Agents und gibt grünes Licht für die Ausführung.|"Agent registriert! Pipeline läuft!"|
|Container Instanz|Holt das Image, startet den Container, erkennt fehlende Komponenten.|"Container läuft!" / "Komponente MkDocs fehlt!"|


**Spielablauf**
**Phase 1 – Erster Build**
1. Dockerfile Master übergibt das Dockerfile (Zettel mit Anweisungen) an die Container Registry.
1. Container Registry baut das Image und speichert es.
1. Placeholder Agent erkennt: "Ein neuer Pipeline-Job will laufen!"
1. Placeholder aktiviert den Job Agent.
1. Container Instanz zieht das Image aus der Registry und startet es. Meldet: "Container läuft!"
1. Job Agent registriert sich im Agent Pool.
1. Azure DevOps Pipeline wird ausgeführt .

**Rollenkarten**
**Dockerfile Master**
- 📝 Aufgabe: Bauanleitung übergeben und korrigieren
- 🎙️ Signal: „Ich ergänze: pip install mkdocs!“

**Container Registry**
- 🏗️ Aufgabe: Image bauen und speichern
- 🎙️ Signal: „Image gespeichert!“

**Placeholder Agent**
- 🕹️ Aufgabe: Job Agent starten
- 🎙️ Signal: „Pipeline erkannt – starte Job Agent!“

**Job Agent**
- 🛠️ Aufgabe: Image holen, Pipeline ausführen
- 🎙️ Signal: „Registriere mich im Agent Pool!“

**Agent Pool**
- 🗂️ Aufgabe: Agent registrieren
- 🎙️ Signal: „Pipeline läuft!“

**Container Instanz**
- 📦 Aufgabe: Image starten, Fehler erkennen
- 🎙️ Signal: „MkDocs fehlt!“ / „Container läuft!“



