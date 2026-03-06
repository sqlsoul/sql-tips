---
layout: default
---

Short, practical SQL tips, query patterns, and database tooling insights designed to help developers and DBAs solve real database problems faster.

{% if site.posts.size > 0 %}
## Latest Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
{% else %}
No posts yet. Stay tuned!
{% endif %}
