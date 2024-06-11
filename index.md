---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
---

Ce référentiel contient les exercices de labo pratique pour le cours Microsoft [DP-420 Conception et implémentation d’applications natives cloud à l’aide de Microsoft Azure Cosmos DB][course-description] et les [modules auto-rythmés sur Microsoft Learn][learn-collection] équivalents. Les exercices sont conçus pour accompagner les supports de cours d'apprentissage et vous permettre de vous exercer à utiliser les technologies qu'ils décrivent.

> &#128221; Pour réaliser ces exercices, vous devez disposer d’un abonnement Microsoft Azure. Vous pouvez vous inscrire à un essai gratuit sur [https://azure.microsoft.com][azure].

## Laboratoires

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %}
| Module | Laboratoire |
| --- | --- |
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
