---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Blog
---

# Blog Posts

<ul>
  {% for post in site.posts %}
    {% if post.hidden == null or post.hidden == false %}
      <li>
        <a href="/~dgeisler{{ post.url }}">{{ post.title }}</a>
      </li>
    {% endif %}
  {% endfor %}
</ul>