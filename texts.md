---
layout: page
title: Materials
---

This collection contains articles, guides, and information about S4RC and our work in sustainable, secure scientific software and reproducible computing.

## Available Materials

{% for item in site.texts %}
- [{{ item.title }}]({{ site.baseurl }}{{ item.url }}){% if item.author %} by {{ item.author }}{% endif %}
{% endfor %}


