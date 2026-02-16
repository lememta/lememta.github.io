---
layout: page
title: Blog
permalink: /blog/
---

<div class="posts">
  {% for post in site.posts %}
    <article class="post">
      <h2 class="post-title">
        <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
      </h2>

      <time datetime="{{ post.date | date_to_xmlschema }}" class="post-date">
        {{ post.date | date: "%B %d, %Y" }}
      </time>

      {% if post.description %}
        <p class="post-excerpt">{{ post.description }}</p>
      {% else %}
        <p class="post-excerpt">{{ post.excerpt | strip_html | truncate: 200 }}</p>
      {% endif %}

      {% if post.tags.size > 0 %}
        <div class="post-tags">
          {% for tag in post.tags %}
            <span class="tag">{{ tag }}</span>
          {% endfor %}
        </div>
      {% endif %}
    </article>
    
    {% unless forloop.last %}<hr class="post-separator">{% endunless %}
  {% endfor %}
</div>

{% if site.posts.size == 0 %}
  <p>No posts yet. Check back soon!</p>
{% endif %}
