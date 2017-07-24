---
layout: page
title: Topics 
permalink: /Topics/
---

<ul>
{% for category in site.categories %}
	<li>
		<a name="{{ category | first }}">{{ category | first }}</a>
		<ul>
			{% for posts in category %}
				{% for post in posts %}
					{% assign thesize = post.title.size %}
					{% if thesize > 0 %}
						<li><a href="{{ post.url }}">{{ post.title }}</a></li>
					{% endif %}
				{% endfor %}
			{% endfor %}
		</ul>
	</li>
{% endfor %}
</ul>

