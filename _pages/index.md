---
layout: page
title: Home
id: home
permalink: /
---

# Welcome to My Digital Garden! 🌱

<p style="padding: 3em 1em; background: #f5f7ff; border-radius: 4px;">
  Take a look at <span style="font-weight: bold">[[Minimal-technical-requirement.md|기본 제약조건 보러가기]]</span> to get started on your exploration.
</p>

Hello! I'm a student at 42seoul, and this site is my digital space for sharing:

- 🖥️ My 42 curriculum projects
- 🚀 Personal coding projects
- 📝 Learning notes and personal insights

Here, you can follow my learning journey and explore my experiences and insights into coding and technology.

## Recently Updated Notes

<ul>
  {% assign recent_notes = site.notes | sort: "last_modified_at_timestamp" | reverse %}
  {% for note in recent_notes limit: 5 %}
    <li>
      {{ note.last_modified_at | date: "%Y-%m-%d" }} — <a class="internal-link" href="{{ site.baseurl }}{{ note.url }}">{{ note.title }}</a>
    </li>
  {% endfor %}
</ul>

This digital garden template is free, open-source, and [available on GitHub here](https://github.com/maximevaillancourt/digital-garden-jekyll-template).

Feel free to explore and don't hesitate to reach out if you have any questions about my projects or notes!

<style>
  .wrapper {
    max-width: 46em;
  }
</style>