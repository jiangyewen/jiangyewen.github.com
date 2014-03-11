---
layout: page
title: List
tagline: Every dog has its day
---


<ul class="posts">
  {% for post in site.posts %}
    
        <h2><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h2>
        <div class="title-desc">{{ post.description }}</div>
    
  {% endfor %}
</ul>



