---
title: Grant's GIS Projects
---
# GIS Projects

Here are some GIS projects I've done.

- [Determining Release Locations for Rehabilitated Birds](./bird_release)

## GIS-related blog posts

<ul>
  {% for post in site.categories.gis %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> ({{ post.date | date_to_string }})
    </li>
  {% endfor %}
</ul>
