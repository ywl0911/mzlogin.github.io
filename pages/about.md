---
layout: page
title: About
description: Warning:nuclear silo detected.
comments: true
menu: 关于
permalink: /about/
---

I obtained a double B.Sc. degree majoring in management and finance from Zhongnan University of Economy and Law in June 2015. In the same year,
I was admitted to study for a M.Sc. degree in School of Computer Science and Technology, Harbin Institute of Technology. I was supervised by Prof. [Yunming Ye](http://www.hitsz.edu.cn/teacher/view/id-237.html#) and graduated in 2018.


If you have any questions about my blog, don't hesitate to get in touch with me through my email.

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
