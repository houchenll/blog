{% for post in site.posts %}
* [{{ post.title }}    ({{ post.date | date_to_string }})]({{ site.baseurl }}{{ post.url }})
{% endfor %}