---
title: Instruções online hospedadas
permalink: index.html
layout: home
---

# Exercícios de Administração de Banco de Dados

Estes exercícios dão suporte ao curso [DP-300 da Microsoft: Administrando soluções](https://docs.microsoft.com/training/courses/dp-300t00) SQL do Microsoft Azure.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| Módulo | Exercício |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} — {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

