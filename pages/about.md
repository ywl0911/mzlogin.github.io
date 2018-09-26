---
layout: page
title: About
description: Warning:nuclear silo detected.
comments: true
menu: 关于
permalink: /about/
---
I am a lover of machine learning especially in the field of NLP. I obtained a B.Sc. degree majoring in both management and finance from Zhongnan University of Economy and Law in June 2015. In the same year,
I was admitted to study for a M.Sc. degree in Harbin Institute of Technology. I was supervised by Prof. [Yunming Ye](http://homepage.hit.edu.cn/pages/yeyunming) and graduated in 2018.

If you have any questions about my blog, don't hesitate to email me. At last, really appreciate [Zhuang Ma](mazhuang.org) providing the blog template.

## Contact

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
