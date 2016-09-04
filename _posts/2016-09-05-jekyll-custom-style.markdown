---
layout: post
title:  "自定义Jekyll的风格"
date:   2016-09-05 00:36:27 +0900
categories: tech
---

### 修改minima主题

之前的post里介绍的jekyll设置是用了minima主题，见`_config.yml`里：

```yaml
theme: minima
```

于是就去jekyll的GitHub上找到这个主题的代码库 https://github.com/jekyll/minima

把它拉下来，我们要参考的是_includes, _layouts, _sass这三个目录。

把这三个目录拷到自己项目里，然后记得在`_config.yml`里把`theme: minima`注释掉。

### 添加资源

在项目的目录下新建一个任意名称的目录比如`asset`，里面放上要用的资源文件`asset/header-bg.png`，用`jekyll serve`重新编译启动jekyll时会自动把这个asset文件夹拷到`_site`里。

比如要在header里添上这个图片资源的话， 修改`_includes/header.html`如下：

```html
<header class="site-header" role="banner" style="background-image: url({% raw %}{{ site.url }}{% endraw %}/asset/header-bg.png); background-repeat: repeat-x;">
```

这里的`site.url`会被解析成`_config.yml`里定义的`url`。

### 使用 raw, endraw 表述带保留字段的文字

比如想在博文里显示诸如 `{% raw %}{{ site.url }}{% endraw %}` 或 `{% raw %}{% ??? %}{% endraw %}`之类的特殊字符，但又不想然jekyll在编译过程中解析处理这些特殊字符的话，要用raw, endraw标记：

```
{% raw %}{% raw %}{% endraw %}
{% raw %}{{ site.url }}{% endraw %}
{% raw %}{% endraw{% endraw %}{% raw %} %}{% endraw %}

{% raw %}{% raw %}{% endraw %}
{% raw %}{% ??? %}{% endraw %}
{% raw %}{% endraw{% endraw %}{% raw %} %}{% endraw %}
```