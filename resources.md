---
layout: page
title: Resources
---

A collection of resources for sustainable, secure scientific software and reproducible computing.

## Tools and Technologies

Resources and tools that support our work in scientific software development.

## Documentation

Guides, tutorials, and documentation to help improve scientific software practices.

{% for item in site.resources %}
- [{{ item.title }}]({{ site.baseurl }}{{ item.url }})
{% endfor %}

## External Links

Links to relevant organizations, projects, and resources in the scientific computing community.

