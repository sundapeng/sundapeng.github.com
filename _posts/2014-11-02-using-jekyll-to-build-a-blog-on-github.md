---
layout: post
title: "利用jekyll在github上搭建博客"
description: "本文主要介绍了如何利用github搭建一个免费的博客"
category: jekyll 
tags: [jekyll, github建站]
---
{% include JB/setup %}

## 准备搭建环境

**由于博主使用的Ubuntu 14.10, 以下命令主要适用于Ubuntu, 其他Linux系统以及Mac系统请视情况而定**

- github 账号
- git
- ruby
- python

## 创建你的git repo

**在github上创建同名的repo  `USERNAME.github.com`**

**克隆一份jekyll的代码到本地, 并推送到你的repo上**

<pre class="prettyprint linenums">
git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com
cd USERNAME.github.com
git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git
git push origin master
</pre>

过几分钟，就可以看到 [http://USERNAME.github.io](http://USERNAME.github.io) 已经更新了

## 在本机安装jekyll server

**ruby1.9.1-dev中包含gem的安装包**

<pre class="prettyprint">
sudo apt-get install ruby1.9.1-dev
</pre>

**安装jekyll**

<pre class="prettyprint">
sudo gem1.9.1 install jekyll
</pre>

**test jekyll**

<pre class="prettyprint">
jekyll serve
</pre>

如果你启动jekyll时遇到`Could not find a JavaScript runtime. See https://github.com/sstephenson/exec`
请JavaScript引擎，如nodejs `sudo apt-get install nodejs`

**rake**

<pre class="prettyprint">
sudo gem1.9.1 install rake
</pre>

**theme**

<pre class="prettyprint">
rake theme:install git="https://github.com/jekyllbootstrap/theme-twitter.git"
</pre>

##配置jekyll

修改`_config.yml`文件

将一些基础信息配置成想要的内容

###添加文章
在`_posts`目录下新建一个`markdown`(`*.md`)文件,
文件命名规范是`yyyy-mm-dd-url`, 例如该文章的文件为`2012-01-01-test.md`


得到的访问路径却是
[/javascript/2012/01/01/test/](/javascript/2012/01/01/test/)  
其中`/javascript`是在markdown文件中配置的.

markdown文件头需要几个配置, 以下是该文章的头配置

<pre class="prettyprint linenums">
---
layout: post
title: 在github上搭建博客
category: javascript
tags: [github, bootstrap, jekyll, javascript]
---
</pre>

每个markdown必须在头部加上这段. 然后下面直接写markdown代码就行了.

##配置域名
> 新建一个`CNAME`文件, 里面直接写上所配置的域名, 例如`sundp.me`

然后上域名提供商上配置域名解析, `A`记录到`207.97.227.245`

等待域名解析完毕即可, 直接访问`http://sundapeng.github.io` 会跳转至 `http://sundp.me`

##duoshuo

`vim _config.yml`

{% highlight YAML %}
  comments :
    provider : duoshuo
    duoshuo :
      short_name : sdp-github
{% endhighlight %}

`vim ./_includes/JB/comments`

{% highlight ruby %}
{{ "{%" }} when "facebook" %}
  {{ "{%" }} include JB/comments-providers/facebook %}
{{ "{%" }} when "duoshuo" %}
  {{ "{%" }} include JB/comments-providers/duoshuo %}
{{ "{%" }} when "custom" %}
  {{ "{%" }} include custom/comments %}
{% endhighlight %}

`vim _includes/JB/comments-providers/duoshuo`

{% highlight javascript %}
<!-- Duoshuo Comment BEGIN -->
    <!-- 多说评论框 start -->
    <div id="comments">
      <div class="ds-thread" {{ "{%" }} if page.id %}data-thread-key="{{ "{{" }} page.id }}"{{ "{%" }} endif %}  data-title="{{ "{%" }} if page.title %}{{ "{{" }} page.title }} - {{ "{%" }} endif %} " {{"{{"}} site.title }}"></div>
    </div>
    <!-- 多说评论框 end -->
    <!-- 多说公共JS代码 start (一个网页只需插入一次) -->
    <script type="text/javascript">
      var duoshuoQuery = {short_name:"{{ site.JB.comments.duoshuo.short_name }}"};
      (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(ds);
      })();
    </script>
    <!-- 多说公共JS代码 end -->
<!-- Duoshuo Comment END -->
{% endhighlight %}

##baidu

`vim _config.yml`

{% highlight YAML %}
analytics :
    provider : baidu
    baidu :
        key : '112e9d845f2671e7bbe63d9c143ead10'
    google :
        tracking_id : 'UA-123-12'
{% endhighlight %}

`vim _includes/JB/analytics`

{% highlight ruby %}
{{ "{%" }} when "baidu" %}
  {{ "{%" }} include JB/analytics-providers/baidu %}
{% endhighlight %}

{% highlight ruby %}
{{ "{%" }} when "piwik" %}
  {{ "{%" }} include JB/analytics-providers/piwik %}
{{ "{%" }} when "baidu" %}
  {{ "{%" }} include JB/analytics-providers/baidu %}
{{ "{%" }} when "custom" %}
  {{ "{%" }} include custom/analytics %}
{{ "{%" }} endcase %}
{% endhighlight %}

`vim _includes/JB/analytics-providers/baidu`

{% highlight javascript %}
<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "//hm.baidu.com/hm.js?{{ "{{" }} site.JB.analytics.baidu.key }}";
  var s = document.getElementsByTagName("script")[0];
  s.parentNode.insertBefore(hm, s);
})();
</script>
{% endhighlight %}

##highlight

{% highlight bash %}
sudo apt-get install python-pip
pip install pygments --user
sudo gem1.9.1 install pygments.rb
sudo apt-get install python-pygments
{% endhighlight %}

`pygmentize -S default -f html > assets/themes/bootstrap-3/css/pygments-friendly.css`

`vim ./_includes/themes/bootstrap-3/default.html` 加入
{% highlight html %}
<link href="{{ ASSET_PATH }}/css/pygments.css" rel="stylesheet" type="text/css">
{% endhighlight %}

##优化
jquery wget到本地
