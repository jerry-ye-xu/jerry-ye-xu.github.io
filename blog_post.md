---
layout: page
title: Blog Posts
category: Posts
tags: ['post', 'archive']
---

Here is an archive of all the blog posts on this site. It is grouped in ascending chronological order based on category.

<h2 class="no_toc">Categories</h2>

* TOC
{:toc}

<hr class="shadow">

## Data Science 
{% for post in site.posts %}
	{% if post.category == 'Data Science' %}
  		<a href="{{ post.url }}"> - {{ post.date | date_to_string }}: {{ post.title }}</a><br>
  	{% endif %}
{% endfor %}
<hr class="shadow">

## Computer Science 
{% for post in site.posts %}
	{% if post.category == 'Computer Science' %}
  		<a href="{{ post.url }}"> - {{ post.date | date_to_string }}: {{ post.title }}</a><br>
  	{% endif %}
{% endfor %}
<hr class="shadow">

## Miscellaneous
{% for post in site.posts %}
	{% if post.category == 'Miscellaneous' %}
  		<a href="{{ post.url }}"> - {{ post.date | date_to_string }}: {{ post.title }}</a>
  	{% endif %}
{% endfor %}
<hr class="shadow">