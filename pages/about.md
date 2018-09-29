---
layout: page
title: About
description: Warning:nuclear silo detected.
comments: true
menu: 关于
permalink: /about/
---

I am a lover of machine learning especially for the field of NLP. In June 2015, I obtained my B.Sc. degree majoring in both management and finance from Zhongnan University of Economy and Law.
I received my M.Sc. degree in computer science from Harbin Institute of Technology, Shenzhen in Jan 2018. If you have any questions about me or my blog, don't hesitate to email me.


At last, really appreciate [Zhuang Ma](http://mazhuang.org) providing the blog template.



## Contact

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}


{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
<br>
<br>
</div>
{% endfor %}
