---
title: Online gehostete Anweisungen
permalink: index.html
layout: home
---

# Microsoft Learn: Praktische Übungen

Die folgenden praktischen Übungen dienen als Ergänzung zu [Microsoft Learn](https://docs.microsoft.com/training/)-Schulungen.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| |
| --- | --- | 
{% for activity in labs  %}| [{{ activity.lab.title }}{% if activity.lab.type %} – {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
