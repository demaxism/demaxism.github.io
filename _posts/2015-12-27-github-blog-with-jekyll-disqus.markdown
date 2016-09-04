---
layout: post
title:  "用Jekyll在GitHub上做基于markdown的博客"
date:   2015-12-27 23:51:14 +0900
categories: tech
---

Github是很无敌的代码库，其实还可以做免费的博客空间。 这里其实用到的是GitHub的Pages功能，可以用markdown写简洁高雅的技术博。

### GitHub Pages

首先你得有个GitHub账号，这里省去一万字关于开通账号和怎么在上面件代码库项目的诸多云云。
跑到https://pages.github.com/， 这上面介绍了怎么做Page, 简单说就是可以创建一个特定的代码库，GitHub会自动解析这个代码库里的文件并公开成一个Web页面，于是就成了个博客空间。 

如果GitHub用户名是peter, 就创建一个repository取名叫`peter.github.io`, 这个不能写错因为GitHub只会找这个repository来解析成Page。

代码库里写个index.html然后推上去后，用浏览器打开http://peter.github.io 就能看到自己写的index.html显示出来了：

```shell-session
$ cd peter.github.io
$ $echo "Hello World" > index.html
$ git add --all
$ git commit -m "Initial commit"
$ git push -u origin master
```
### 用Jekyll写markdown格式的博客

博客空间已经有了接下来就考虑做博客的内容。
Jekyll是一个能将markdown文本转化成漂亮的静态页网页的ruby库。

于是先安装这个库，在我的mac上:

```shell-session
$ cd peter.github.io
$ gem install jekyll bundler
```

对了gem是Ruby的一个包管理器，如果没装的话就搜一下怎么安装gem.

jekyll装好后就可以用jekyll这个命令新建网站项目了，可以这样`jekyll new my-awesome-site`新建一个网站项目的文件夹，但我打算就在peter.github.io目录里生成项目，先把刚刚测试用的index.html删掉，然后:

```shell-session
$ cd peter.github.io
$ jekyll new .
```

文件夹里出现了_config.yml, _post, css等文件和文件夹，可以先本地跑一下：

```shell-session
$ bundle install
$ bundle exec jekyll serve
```

浏览器打开http://localhost:4000可以确认。

以下是几个说明：

* _config.yml  ［要改］这是主要的配置文件，里面设网站的标题，描述，或定义一些模块用的变量
* _posts/ ［要改］这个文件夹里用来放博客的条目，以后每发一次博文就在里面加一个markdown文件，jakyll运行时会编译这些条目
* _site/ ［别碰］这是jekilly根据_config.yml和_posts里的文本生成的网页，每次生成这个文件夹都会被覆盖掉，所以不用碰。

由于_site/是会被生成覆盖的，而且GitHub里内置有jekyll编译生成机制，不需要上传_site/, 于是把_site/放在.gitignore里：

```shell-session
$ $echo "_site/" > .gitignore
$ git add --all
$ git commit -m "Jekyll"
$ git push -u origin master
```

打开http://peter.github.io确认更新了。

### 借助Disqus实现读者评论

光能写博别人无法留言那肯定很寂寞，刚才提过GitHub的Pages只能host静态页面，想搞读者评论得有动态API连数据库吧。 
其实这年头留言评语系统也不需要做动态页和数据库了，javascript就能搞定。

这里我用Disqus的留言javascript API实现评论。 其实Disqus不但提供评论的API，而且能将流量转成现金给发布者，这也是他们的广告的业务。具体参见他们的网页https://disqus.com/how/

* 首先创建一个Disqus的账户，比如用户名peter。

* 注册完后点左上角的Disqus logo，跳到页面问'I want to comment on sites'和'I want to install Disqus on my site', 选第二个，跳到创建网站页面https://disqus.com/admin/create/

* 在'Create a new site'界面里，Website Name里一个任意的便于识别的名字比如petergithub, Catagory下拉框里选个类别后就能点'Create Site'下一步了。

* 于是进入了Admin控制页面 https://mygoodsite.disqus.com/admin/install/， 左边菜单有 
    1. Accept Policy (确保它打勾了) 
    2. Install Disqus
        - Select Platform (现在在这儿)
        - Install Instructions
    3. Configure Disqus

* 先进Configure Disqus(https://mygoodsite.disqus.com/admin/install/settings/), 这里很重要的一步是在Website URL里填上GitHub Page的URL： `http://peter.github.io`

* 然后回到Install Disqus, 里面问What platform is your site on? 里面有Wordpress, Blogger之类的选项，我用的是GitHub不在里面，于是点最下面的‘I don't see my platform listed...’

* 进入了Universal Code install instructions页面(https://mygoodsite.disqus.com/admin/install/platforms/universalcode/), 这里的第一段 'Place the following code where you'd like Disqus to load'里写的HTML/JavaScript就是我们要的代码。

添加路径和文件`_includes/disqus.html`, 参照上面页面里的Javascript加入以下代码，注意根据自己的实际URL替换掉`s.src = '//https-peter-github-io.disqus.com/embed.js'`:

```html
{% if site.disqus %}
<div class="comments">
    <div id="disqus_thread"></div>
<script>
(function() { // DON'T EDIT BELOW THIS LINE
    var d = document, s = d.createElement('script');
    s.src = '//https-peter-github-io.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a>
</noscript>
</div>
{% endif %}
```

* 添加路径和文件`_layouts/post.html`:

```html
---
layout: default
---
<article class="post">
  <h1>{{ page.title }}</h1>

  <div class="entry">
    {{ content }}
  </div>

  <div class="date">
    Written on {{ page.date | date: "%B %e, %Y" }}
  </div>
{% raw %}
  {% include disqus.html %}
{% endraw %}
</article>
```

* 在`_config.yml`里加上这些：

```yaml
comments: true
disqus: peter
url: "https://peter.github.io" # the base hostname & protocol for your site
```

这里disqus后面写的是Disqus的用户名，在setting页面(https://disqus.com/home/settings/account/)里的Username

把这些修改传上去后，就能看到每条post的最下面出现Disqus的评论插件了。


### 其他参考链接
* Build A Blog With Jekyll And GitHub Pages https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/
* Setting up Github Pages  https://davidwinter.me/setting-up-github-pages/
