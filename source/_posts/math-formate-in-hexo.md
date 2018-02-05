---
title: hexo中编辑数学公式
date: 2018-02-03 22:28:15
tags:
- hexo
categories:
- hexo
---
hexo中使用数学公式的一个简单介绍（非教程），全当记录自己的想法，等自己探索差不多后会沉淀成教程。
<!-- more -->
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
});
</script>

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>


# 背景
这几天使用hexo搭建了自己的博客，在设定了一些个性化的配置后，就开始写自己的博客了，是不是很开心？然而我第一篇文章中就遇到问题了，怎么在博文中插入数学公式，并且本地编辑的时候能够可视化查看。基于这个问题，在网上查了半天终于搞定了。首先定义一下我们要解决的问题：

1. hexo博文中能够加入数学公式并能完整显示（基于markdown）。
2. 编辑本地markdown，并能可视化查看。

# hexo工作原理
其实下面讲了一通，在服务端可视化只要在hexo的配置中打开MathJax的开关就行了，同时对于marked中转义字符的问题参考[Hexo下mathjax的转义问题](http://shomy.top/2016/10/22/hexo-markdown-mathjax/)解决，想了解原理的可以继续看。

在开始讲解解决方案前，我们需要先了解一下hexo的工作原理，既hexo是如何通过我们编辑的makrdown文件生成一个漂亮的博客网站的。在[官网](https://hexo.io/zh-cn/docs/index.html)中如是描述hexo：
> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

换言之hexo是一个生成器或转换器，首先将source下的makrdown文件转换为可视化的html文件，然后根据我们的主题和相应的配置，渲染和组织博客，摘自[hexo是怎么工作的](http://coderunthings.com/2017/08/20/howhexoworks/)：

----------

简单来说，hexo中，从markdown到html的generate过程中做了两件事：模板渲染+模板渲染。如图所示
![](http://coderunthings.com/images/howhexoworks/render.png)
对应的source下文件
![](http://coderunthings.com/images/howhexoworks/post.png)
对上面表格和图的说明:

- hexo core在generate的过程中会产生一个对象，我们在这里把这个对象称为article。第一次渲染的主要目的就是给这个对象添加title,content等属性。其中:
    1. article.title, article.date, article.tags, article.categories等属性来自yml front的部分
	2. article.content是markdown文章解析后的html片段
- hexo项目目录下包含三个子目录，
	1. source目录，写博客的主要工作目录。这个目录下存放的是我们的markdown文章以及js, images, css
	2. themes目录，主题目录，定义了即将生成的html的layout, 和html中需要加载的css, js, images
	3. ublic目录, hexo generate的最终输出目录。里面包含了整个博客网站的html, css, js, images
- 第二次渲染，需要引入对应模板文件格式的插件，如.ejs文件就需要使用hexo-render-ejs插件，.jade文件需要使用hexo-render-jade插件，而.sass文件则需要hexo-render-sass插件来转换成css文件。hexo的这一设计有点类似webpack中的loader.`

----------
通过以上的介绍，大家对hexo的原理应该很清楚了。我们所关心的在文中插入数学公式，只要在在将markdown文件转换为html时，转换器识别其中的数学公式符号并有对应的渲染器渲染数学公式就OK了。因此，第一个问题转换为：
1. 转换器识别markdown数学公式符号
2. html中能够渲染数学公式

对于转换器(解析器)hexo中使用的是[hexo-renderer-marked](https://github.com/hexojs/hexo-renderer-marked)，将标准的markdown转换为html格式文件。对于网页中公式的渲染使用的是MathJax,在hexo的配置文件中可以设置开启，这样生成的网页就会加上对应Mathjax配置。简单认识下这个两个工具

## marked
[marked](https://github.com/chjj/marked):marked是一个基于Nodejs的Markdown解析引擎。Markdown不是HTML，目前还不能被浏览器解析，所以需要Markdown的解析器如marked，把Markdown翻译成浏览器认识的HTML文档。举个栗子，下文是一个markdown的文章，通过marked转换为html格式。
		
	Marked Demo
	======================
	
	这是一个Marked库使用的例子。 http://blog.fens.me/nodejs-markdown-marked/
	
	> A full-featured markdown parser and compiler, written in JavaScript. Built
	> for speed.
	
	[![NPM version](https://badge.fury.io/js/marked.png)][badge]
	
	## Install
	
	``` bash
	npm install marked --save
	```
	
	## 列表测试
	
	+ 列表测试，行1
	+ 列表测试，行2
	+ 列表测试，行3
	+ 列表测试，行4
	
	## 表格测试
	
	A | B | C
	--|--|--
	A1 | B1 | C1
	A2 | B2 | C2
	A3 | B3 | C3
	
转换后的html

    <h1 id="marked-demo">Marked Demo</h1>
	<p>这是一个Marked库使用的例子。 <a href="http://blog.fens.me/nodejs-markdown-marked/">http://blog.fens.me/nodejs-man-marked/</a></p>
	<blockquote>
	<p>A full-featured markdown parser and compiler, written in JavaScript. Built
	for speed.</p>
	</blockquote>
	<p>[<img src="https://badge.fury.io/js/marked.png" alt="NPM version">][badge]</p<
	<h2 id="install">Install</h2>
	<pre><code class="lang-bash">npm install marked --save
	</code></pre>
	<h2 id="-">列表测试</h2>
	<ul>
	<li>列表测试，行1</li>
	<li>列表测试，行2</li>
	<li>列表测试，行3</li>
	<li>列表测试，行4</li>
	</ul>
	<h2 id="-">表格测试</h2>
	<table>
	<thead>
	<tr>
	<th>A</th>
	<th>B</th>
	<th>C</th>
	</tr>
	</thead>
	<tbody>
	<tr>
	<td>A1</td>
	<td>B1</td>
	<td>C1</td>
	</tr>
	<tr>
	<td>A2</td>
	<td>B2</td>
	<td>C2</td>
	</tr>
	<tr>
	<td>A3</td>
	<td>B3</td>
	<td>C3</td>
	</tr>
	</tbody>
	</table>
	

## mathjax
[MathJax](https://mathjax-chinese-doc.readthedocs.io/en/latest/mathjax.html)是一个开源JavaScript库。它支持LaTeX、MathML、AsciiMath符号，可以运行于所有流行浏览器上。 它的设计目标是利用最新的web技术，构建一个支持math的web平台。支持主要的浏览器和操作系统,包括那些移动设备。使用MathJax显示数学公式是基于文本的，而非图片。它可以被搜索引擎使用，这意味着方程式和页面上的文字一样是可以被搜索的。 MathJax允许页面作者使用TeX、LaTeX符号和 MathML 或者 AsciiMath 去书写公式。 MathJax甚至可以将Tex格式转化为MathML格式，使其可以被原生支持MathML格式的浏览器更多的渲染。转化为MathML格式后你可以复制粘贴它们到其他程序中。

MathJax允许你在你的网页中包含公式，无论是使用LaTeX、MathML或者AsciiMath符号，这些公式都会被javascript处理为HTML、SVG或者MathML符号。具体栗子：
	
	<!DOCTYPE html>
	<html>
	<head>
	<title>MathJax TeX Test Page</title>
	<script type="text/x-mathjax-config">
	  MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
	</script>
	<script type="text/javascript"
	  src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
	</script>
	</head>
	<body>
	When $a \ne 0$, there are two solutions to \(ax^2 + bx + c = 0\) and they are
	$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$
	</body>
	</html>

在这个栗子中，分为几个部分：

1. 加载MathJax:使用MathJax的内容发布网络(CDN),这样就会从分发服务器中加载最新版本的MathJax，并配置它识别用Tex和MathML符号书写的公式。如果浏览器原生支持MathML格式，MathJax就会生成用MathML输出，不然的话就用HTML和CSS去显示公式。这是最常见的配置，它可以满足大部分人的需求.

	<script type="text/javascript"
	  src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
	</script>

2. 设置公式分隔符：使用 TeX 和 LaTeX 格式的公式使用 公式分隔符 环绕公式，即告知MathJax页面哪个部分代码公式和它的基本格式。 这里有两种形式的公式：
	1. 包含在段落之中的
	2. 独立于其他文字的
	
	默认的公式分隔符是 $$...$$ 和 \[...\] ，还有 \(...\) 常用于段落中的公式。请特别注意， \(...\) 分隔符 不是 默认使用的。美元符号$常常在其他情况下使用，这会导致本文被错误的当做公式解析了。

	例如，使用美元分隔符的情况下， ”... the cost is $2.50 for the first one, and $2.00 for each additional one ...” 会被处理为 “2.50 for the first one, and” 。因为介于美元符号之间的文字内容被当做公式处理了。基于这样的理由，如果你想使用美元分隔符，请在配置文件中显示声明
		
		<script type="text/x-mathjax-config">
		MathJax.Hub.Config({
		  tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
		});
		</script>
		<script type="text/javascript" src="path-to-mathjax/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

3. 写公式：
	
		When $a \ne 0$, there are two solutions to \(ax^2 + bx + c = 0\) and they are
		$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$


以上基本摘自MathJax的文档.

##	Marked+MathJax+Markdown
好了上面我们介绍了Marked和MathJax，是不是markdown到html并能顺利展示已经解决了，貌似也不用费什么心思了。但是使用marked.js去解析我们写的markdown，比如一些符号，`_`代表斜体会被处理为`<em>`标签，比如在`x_i`开始被渲染的时候，处理为x<em>i</em>，这个时候mathjax就无法渲染成下标了。很多符号都有这个问题，比如粗体*,也是无法在mathjax渲染出来的，好在有替代的乘法等,包括\\同理。因此问题就是Marked在解析转换markdown时没有考虑数学公式的问题.这里有个解决方法[Hexo下mathjax的转义问题](http://shomy.top/2016/10/22/hexo-markdown-mathjax/)

> 2018-02-05 为了彻底解决问题，免去各种配置问题，我直接转为Pandoc引擎进行markdown文件的渲染。

# 本地可视化编辑
本地化编辑我是用的MarkdownPad,在这个编辑器中公式不能直接浏览，需要在网页中查看。相关配置：在markdown文件中内容部分直接加入：

	---
	title: math formate in hexo
	date: 2018-02-03 22:28:15
	tags:
	- hexo
	categories:
	- hexo
	---
	hexo中使用数学公式的一个解决方法和相应的公式符号。
	<!-- more -->
	<script type="text/x-mathjax-config">
	MathJax.Hub.Config({
	tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
	});
	</script>
	
	<script type="text/javascript" async
	  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
	</script>

然后在“工具->在浏览器中浏览”就可以了，当然为了省去每次在在文件中粘贴的麻烦，可以在“工具->选项->高级->自定义Html Head”中加入即可。

    <script type="text/x-mathjax-config">
	MathJax.Hub.Config({
	tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
	});
	</script>
	
	<script type="text/javascript" async
	  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
	</script>

当然有很多其他直接浏览的工具，如cmd Markdown, stackEdit等。按需自取吧。

# 简单公式
可以直接参考[Easy Copy MathJax](http://easy-copy-mathjax.xxxx7.com/) 中数学公式的MathJax识别的符号。下面是自己经常用的一些记录。
## 行中公式
对于一个线性方程 \\( y=a*x+b\\)

$$f'(x\_0)=\lim_{\Delta x\to 0} \frac{f(x\_0+\Delta x) - f(x\_0)}{\Delta x}$$

# 资料
1. [如何在Hexo博客中插入数学公式](http://daniellaah.github.io/2016/Mathmatical-Formula-within-Markdown.html)
2. Markdown+math 编辑器[StackEdit](https://github.com/benweet/stackedit/)
3. [Pandoc markdown语法](http://pages.tzengyuxio.me/pandoc/)