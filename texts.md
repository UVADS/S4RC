---
layout: page
title: Texts
---

# Texts Collection

This collection contains articles, guides, and information about S4RC and our work in sustainable, secure scientific software and reproducible computing.

## Available Texts

{% for item in site.texts %}
- [{{ item.title }}]({{ site.baseurl }}{{ item.url }}){% if item.author %} by {{ item.author }}{% endif %}
{% endfor %}

---

*The Ed theme is designed for minimal editions of texts, perfect for scholarly and scientific publications.*
