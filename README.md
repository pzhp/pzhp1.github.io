# About Theme
Jekyll theme by [Gaohaoyang](https://github.com/Gaohaoyang/gaohaoyang.github.io)**
This is a blog theme based on jekyll. You can use on your own blog. Before you use it, please click a star on [this respository](https://github.com/Gaohaoyang/gaohaoyang.github.io/). You will encourage me to do more great things!

# Attention

* Change information in _config.yml
* Delete statistics code in _includes/head.html.
like this:
```javascript
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "//hm.baidu.com/hm.js?**************************";
  var s = document.getElementsByTagName("script")[0]; 
  s.parentNode.insertBefore(hm, s);
})();
```

* Change the duoshuo comment code in _layouts/default.html.
like this:
```javascript
var duoshuoQuery = {short_name:"******"};
    (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(ds);
})(); 
```
And in post.html/post.html
```
<div class="ds-thread" data-thread-key="{{ page.id }}" data-title="{{ page.title }}" data-url="***.github.io{{ page.url }}"></div>
```
---

LICENSE

[MIT License](https://github.com/Gaohaoyang/gaohaoyang.github.io/blob/master/LICENSE.md)



