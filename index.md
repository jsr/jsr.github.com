---
layout: page
---
{% include JB/setup %}

{% for post in site.posts %}
  <h3><a href="{{ post.url }}"> {{ post.title }} </a></h3>
  <p>{{ post.excerpt }}</p>
{% endfor %} 

