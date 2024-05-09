---
layout: page
title: Posts
nav-menu: true
show_tile: false
---

<!-- Main -->
<div id="main">

<!-- One -->
<section id="one">
	<div class="inner">
		<header class="major">
			<h2>Posts</h2>
		</header>
        <div class="row">
            <ul>
            {% for post in site.posts %}
                <li>
                <a href="{{ post.url }}">{{ post.title }}</a>
                </li>
            {% endfor %}
            </ul>
        </div>
    </div>
</section>

</div>