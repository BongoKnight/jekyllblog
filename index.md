---
title: Bienvenue
layout: default
---
Bienvenue sur ce petit blog, l'id√e est de recenser, des articles, des tweets, des exp√riences personelles ou simplement des petites astuces autour de la s√curit√© informatique eng√n√ral...
Voici si dessous une liste des diff√rents articles.
<ul>
  {% for post in site.posts %}
	<li>
            <a href="{{ post.url }}">{{ post.title }}</a>
	</li>
  {% endfor %}
</ul>
