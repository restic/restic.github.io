---
layout: page
title: Blog
redirect_to: https://restic.net/blog/
permalink: /blog/
---

## Latest Blog Posts

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}

<div class="rss">
  <i class="icon-rss"></i> <a href="/feed.xml">Atom Feed</a>
</div>
