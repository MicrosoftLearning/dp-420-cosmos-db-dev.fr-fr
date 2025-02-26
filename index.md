---
title: Instructions hébergées en ligne
permalink: index.html
layout: home
---

Ce référentiel contient les exercices de labo pratique pour le cours Microsoft [DP-420 Conception et implémentation d’applications natives cloud à l’aide de Microsoft Azure Cosmos DB][course-description] et les [modules auto-rythmés sur Microsoft Learn][learn-collection] équivalents. Les exercices sont conçus pour accompagner les supports de cours d'apprentissage et vous permettre de vous exercer à utiliser les technologies qu'ils décrivent.

> &#128221; Pour réaliser ces exercices, vous devez disposer d’un abonnement Microsoft Azure. Vous pouvez vous inscrire à un essai gratuit sur [https://azure.microsoft.com][azure].

# Laboratoires

## Labos C#

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign csharp_setup_labs = "" | split: "," %} {% assign csharp_regular_labs = "" | split: "," %} {% assign genai_setup_labs = "" | split: "," %} {% assign genai_python_labs = "" | split: "," %} {% assign genai_javascript_labs = "" | split: "," %}

{% for activity in labs %} {% assign segments = activity.url | split: "/" %}

  {% if segments[1] == "instructions" and segments.size == 3 %} {% if activity.lab.module contains "Setup" %} {% assign csharp_setup_labs = csharp_setup_labs | push: activity %} {% else %} {% assign csharp_regular_labs = csharp_regular_labs | push: activity %} {% endif %}
  
  {% elsif activity.url contains '/gen-ai/python/instructions' %} {% assign genai_python_labs = genai_python_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/javascript/instructions' %} {% assign genai_javascript_labs = genai_javascript_labs | push: activity %}
  
  {% elsif activity.url contains '/gen-ai/common/instructions' and activity.lab.module contains "Setup" %} {% assign genai_setup_labs = genai_setup_labs | push: activity %} {% endif %} {% endfor %}

---

### **Configurer des labos**

| Module | Laboratoire |
| --- | --- |
{% for activity in csharp_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Labos**

| Module | Laboratoire |
| --- | --- |
{% for activity in csharp_regular_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

## **Labos d’IA générative**

### **Labos de configuration courants**

| Module | Laboratoire |
| --- | --- |
{% for activity in genai_setup_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Labos JavaScript**

| Module | Laboratoire |
| --- | --- |
{% for activity in genai_javascript_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

---

### **Labos Python**

| Module | Laboratoire |
| --- | --- |
{% for activity in genai_python_labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |  
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
