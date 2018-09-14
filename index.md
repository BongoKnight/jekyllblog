---
title: Bienvenue
layout: main
---
Bienvenue sur ce petit blog, l'idée est de recenser, des articles, des tweets, des expériences personnelles ou simplement des petites astuces autour de la sécurité informatique en général...
Voici-ci dessous une liste des différents articles:
<ul>
  {% for post in site.posts %}
	<li>
            <a href="{{ post.url }}">{{ post.title }}</a>
	</li>
  {% endfor %}
</ul>
