---
layout: home
---

<div class="index-content blog">
    <div class="section">
        <ul class="artical-cate">
            
			<li  class="on"><a href="/"><span>项目</span></a></li>
            <li style="text-align:center"><a href="/article"><span>文章</span></a></li>
			<li style="text-align:right"><a href="/about"><span>关于我</span></a></li>

        </ul>

        <div class="cate-bar"><span id="cateBar"></span></div>

        <ul class="artical-list">
        {% for post in site.categories.project %}
            <li>
                <div class="table-article">
                    <div class="col-title">
                        <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
                    </div>
                    <div class="col-date">
                        <p class="entry-date">{{ post.date|date:"%Y-%m-%d" }}</p>
                    </div>
                </div>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
    </div>
    <div class="aside">
    </div>

</div>
