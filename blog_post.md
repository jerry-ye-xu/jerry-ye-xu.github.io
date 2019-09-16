---
layout: page
title: Blog Posts
category: Posts
tags: ['post', 'archive']
---

Here is an archive of all the blog posts on this site. It is grouped in descending chronological order based on category.

<h2 class="no_toc">Categories</h2>

* TOC
{:toc}

<hr class="shadow">

## Practical 
{% for post in site.posts %}
	{% if post.category == 'Practical' %}
  		<a href="{{ post.url }}"> - {{ post.date | date_to_string }}: {{ post.title }}</a><br>
  	{% endif %}
{% endfor %}
<hr class="shadow">

## Math/CS/Stats 
{% for post in site.posts %}
	{% if post.category == 'Theory' %}
  		<a href="{{ post.url }}"> - {{ post.date | date_to_string }}: {{ post.title }}</a><br>
  	{% endif %}
{% endfor %}
<hr class="shadow">

## Non-technical
{% for post in site.posts %}
	{% if post.category == 'Miscellaneous' %}
  		<a href="{{ post.url }}"> - {{ post.date | date_to_string }}: {{ post.title }}</a><br>
  	{% endif %}
{% endfor %}
<hr class="shadow">
