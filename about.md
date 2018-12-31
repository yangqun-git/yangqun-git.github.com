---
layout: page
title: 关于
permalink: about.html
image: /public/images/bluegirl.jpeg
order: 5
---

{% if site.author.photo %}
![{{ site.author.name }}]({{ site.author.photo | prepend: site.cdnurl }}){:.me}
{% endif %}

`ekko` 是我一直在使用的英文 ID，因为很喜欢英雄联盟中艾克这个英雄。

目前还没什么其他可说的，如果想看我的代码可以去我的 [Github](https://github.com/yangqun-git){:target="_blank"}。

## 编程理念

仰慕「优雅编码的艺术」，追崇实践 + 理论得真知。

## 版权说明

我坚信着开放、自由和乐于分享是推动计算机技术发展的动力之一。所以本站所有内容均采用[署名 4.0 国际（CC BY
4.0）](http://creativecommons.org/licenses/by/4.0/deed.zh)创作共享协议。通俗地讲，只要在使用时署名，那么使用者可以对本站所有内容进行转载、节选、二次创作，并且允许商业性使用。

## 其他

之前的文章全删了，感觉写的不好，正好从新来过吧。

爱你呦![miao](/public/images/2018/weisuo.jpg)

<!-- Add Disqus Comments -->
{% if site.disqus %}
{% include disqus.html %}
{% endif %}