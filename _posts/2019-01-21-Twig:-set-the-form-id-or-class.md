---
layout: single
title: Twig - set the form id or class
excerpt: "Sometimes, when we are rendering a form in Twig templates, we need to set a custom ‘id’ or ‘class’ to a form."
date: 2019-01-21
classes: wide
header:
  teaser: /assets/images/form_id.png
  teaser_home_page: true
categories:
  - Research
tags:
  - Symfony
  - Twig  
---

Sometimes, when we are rendering a form in Twig templates, we need to set a custom ‘id’ or ‘class’ to a form. 
This is how we can achieve that:

Setting an id: 
{% raw %}
```twig
{% form_start( form, { 'attr': { 'id': 'form_person_edit'}) %}
```
{% endraw %}

{% raw %}
Setting a class: 
```twig
{% form_start(form,{'attr':{'class':'your-class'}}) %}
```
{% endraw %}