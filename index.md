---
layout: page
title: home!
tagline:
title_cn : '首页'
---
{% include JB/setup %}

<div>
{% for page in site.posts %}
  {% if page.title != null %}
  <div class="article-desc">
    {% if forloop.first %}
    <h3 class="index-title">
    {% else %}
    <h3 class="index-title">
    {% endif %}
        <a href="{{ BASE_PATH }}{{page.url}}">
            {% if page.title_cn %} {{page.title_cn}} {% else %} {{page.title}} {% endif %}
            <span style="font-size:12px;" class="index-tagline">
                ({{ page.date | date : '%Y年%m月%-d日' }})
            </span>
        </a>
    </h3>
    
    {% unless page.categories == empty and page.tags == empty %}
    <ul class="tag_box inline" style="margin-bottom: 10px;font-size: 14px;">
        <li>分类：</li>
        <li><i class="glyphicon glyphicon-open"></i></li>
        {% assign categories_list = page.categories %}
        {% include custom/post_category %}
        <li style="margin-left: 30px;">标签：</li>
        <li><i class="glyphicon glyphicon-tags"></i></li>
        {% assign tags_list = page.tags %}
        {% include JB/tags_list %}
    </ul>
    {% endunless %}

    <p>{{ page.description }}<a href="{{ page.url }}">阅读全文...</a>
    </p>
    <!-- {{ page.content | truncate: 300, '......' }}     -->
    
    </div>
  {% endif %}
{% endfor %}
</div>