---
title: "Jekyll Recently Updated"
author: hcbak
date: 2025-02-20 17:31:34 +0900
categories: [Blog]
---

## Recently Updated

Jekyll에 Chirpy 테마를 끼워서 사용하고 있는데 오른쪽에 **Recently Updated**의 존재감이 상당했다.

최근 게시글을 보여주는 것 같아서 그냥 그런가보다 하고 쓰는데 뭔가 이상했다. 새로 글을 올려도 저기에 올라가지 않는 경우가 많았기 때문이다.

그러다가 결국 버티지 못하고 뭐하는 친구인지 찾아봤다.

### _layouts\default.html

{% raw %}
```html
<!-- panel -->
<aside aria-label="Panel" id="panel-wrapper" class="col-xl-3 ps-2 mb-5 text-muted">
  <div class="access">
    {% include_cached update-list.html lang=lang %}
    {% include_cached trending-tags.html lang=lang %}
  </div>

  {% for _include in layout.panel_includes %}
    {% assign _include_path = _include | append: '.html' %}
    {% include {{ _include_path }} lang=lang %}
  {% endfor %}
</aside>
```
{% endraw %}

update-list.html을 가져온다고 한다!

### _includes\update-list.html

{% raw %}
```html
{% for post in site.posts %}
  {% if post.last_modified_at and post.last_modified_at != post.date %}
    {% capture elem %}
      {{- post.last_modified_at | date: "%Y%m%d%H%M%S" -}}::{{- forloop.index0 -}}
    {% endcapture %}
    {% assign all_list = all_list | push: elem %}
  {% endif %}
{% endfor %}
```
{% endraw %}

**last_modified_at**을 보니 아무래도 수정당한 최근 파일을 보여주는 것 같다. 내가 원한 것은 이것이 아니므로 아래와 같이 바꿔줬다.

{% raw %}
```html
{% for post in site.posts %}
  {% if post.date %}
    {% capture elem %}
      {{- post.date | date: "%Y%m%d%H%M%S" -}}::{{- forloop.index0 -}}
    {% endcapture %}
    {% assign all_list = all_list | push: elem %}
  {% endif %}
{% endfor %}
```
{% endraw %}

### _data\locales\en.yml

덤으로 이름도 바꿔줬다. **lastmod** 부분을 변경하면 된다.

```yml
panel:
  lastmod: Recently Posted
  trending_tags: Trending Tags
  toc: Contents
```
