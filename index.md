---
title: GenAIOps-Übungen
permalink: index.html
layout: home
---

# Operationalisieren von Anwendungen mit generativer KI

Die folgenden Schnellstartübungen sind darauf ausgelegt, Ihnen eine praktische Lernerfahrung zu bieten, in der Sie allgemeine Aufgaben erkunden, die zum Operationalisieren einer Workload für generative KI in Microsoft Azure erforderlich sind.

> **Hinweis**: Um die Übungen durchführen zu können, benötigen Sie ein Azure-Abonnement, das Ihnen ausreichende Berechtigungen und Kontingente für die Bereitstellung der erforderlichen Azure-Ressourcen und generativen KI-Modelle bietet. Wenn Sie noch kein Konto haben, können Sie sich für ein [Azure-Konto](https://azure.microsoft.com/free) anmelden. Für neue Benutzende gibt es dort eine kostenlose Testoption, die Guthaben für die ersten 30 Tage beinhaltet.

## Schnellstartübungen

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

> **Hinweis**: Diese Übungen können zwar eigenständig absolviert werden, sie sind jedoch als Ergänzung zu den Modulen auf [Microsoft Learn](https://learn.microsoft.com/training/paths/operationalize-gen-ai-apps/) gedacht, in denen Sie tiefer in einige der zugrunde liegenden Konzepte eintauchen können, auf denen diese Übungen basieren.
