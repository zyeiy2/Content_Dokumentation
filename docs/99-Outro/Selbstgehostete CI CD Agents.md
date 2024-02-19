# Selbstgehostete CI CD Agents

![IaasVSPaaS](VMPaaS.png)

## Einleitung

In unserer schnelllebigen IT-Landschaft, wo Effizienz und Kostenoptimierung an oberster Stelle stehen, habe ich, wie viele Entwickler, die Herausforderung, eine flexible und kosteneffiziente Infrastruktur für unsere Entwicklungs- und Betriebsprozesse zu schaffen. Besonders in Cloud-Umgebungen wie Azure, die eine breite Palette von Diensten und Tools für die Entwicklung und Bereitstellung von Anwendungen bieten, stoße ich oft auf ein Dilemma: die Wahl zwischen der Bequemlichkeit von Platform-as-a-Service (PaaS)-Angeboten und der Notwendigkeit, Kontrolle und Flexibilität durch selbstgehostete Lösungen zu bewahren.

Ein spezifisches Beispiel für diese Herausforderung ist der Einsatz von Azure Databricks in isolierten Umgebungen. Hier scheint die konventionelle Lösung oft darin zu bestehen, einen selbstgehosteten Agenten zu nutzen. Diese Methode kann jedoch schnell zu erhöhten Kosten führen, vor allem, wenn eine virtuelle Maschine (VM) dauerhaft betrieben wird. Nicht zu vergessen sind die Aufwände für das Patchmanagement des Betriebssystems.
Zudem kann die Notwendigkeit, eine VM kontinuierlich laufen zu lassen, in vielen unserer Projekte der Strategie, sich auf PaaS-Lösungen zu stützen, entgegenwirken.

Stell dir nun vor, es gäbe eine Lösung, die nicht nur die Kosten minimiert, indem sie Ressourcen nur bei Bedarf verbraucht, sondern auch das zeitaufwendige Management des Betriebssystems überflüssig macht, während der PaaS-Ansatz beibehalten wird. Die Verwendung selbstgehosteter CI/CD-Agents in Kombination mit Azure Container Apps bietet genau diese Vorteile. Diese innovative Lösung ermöglicht es, dass Kosten nur dann anfallen, wenn Deployments ausgeführt werden, und eliminiert gleichzeitig die Notwendigkeit für das Patchen des Betriebssystems.

In diesem Leitfaden werde ich dir zeigen, wie du diese Lösung effektiv implementieren kannst, um deine Entwicklungs- und Bereitstellungsprozesse zu optimieren.




## Links
- [Tutorial: Bereitstellen von selbstgehosteten CI/CD-Runnern und -Agents mit Azure Container Apps-Aufträgen](https://learn.microsoft.com/de-de/azure/container-apps/tutorial-ci-cd-runners-jobs?tabs=bash&pivots=container-apps-jobs-self-hosted-ci-cd-azure-pipelines)
