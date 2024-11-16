---
title: Instruções online hospedadas
permalink: index.html
layout: home
---

# Exercícios de administração de banco de dados

Estes exercícios dão suporte ao curso da Microsoft [DP-300: como administrar soluções do Microsoft SQL do Azure](https://docs.microsoft.com/training/courses/dp-300t00).

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Módulo | Exercício |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} — {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

