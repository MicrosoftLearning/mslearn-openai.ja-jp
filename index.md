---
title: Azure OpenAI 演習
permalink: index.html
layout: home
---

# Azure OpenAI 演習

次の演習は、[Microsoft Learn](https://learn.microsoft.com/training/browse/?terms=OpenAI) のモジュールをサポートするように設計されています。


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
