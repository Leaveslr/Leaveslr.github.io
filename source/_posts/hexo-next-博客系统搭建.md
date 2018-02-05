
---
title: hexo+next 博客系统搭建
date: 2018-02-02 16:36:35
tags: 
- hexo
categories:
- hexo
---
自己搭建博客的一些参考资料，记录防止以后用到。并非教程。未完成，待补充：discus评论系统，加入百度、google的统计。
<!-- more -->
# 博客的搭建
# 博客的个性化设置
## 文章阅读次数
本博客的阅读次数统计使用的是leanCloud账号，注册相对简单，具体步骤：

1. 创建LeanCloud账号[LeanCloud](https://leancloud.cn/)
2. 创建应用：注册并登录LeanCloud后，进入控制台，单击“创建应用”按钮进行应用的创建，输入新应用名称，选择开发版，单击“创建”按钮完成创建。  ![](https://upload-images.jianshu.io/upload_images/291600-a114303ec633d628.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
3. 创建Class:进入到刚刚创建的应用中，选择左侧导航栏的“存储”，然后点击“创建Class”，为了与NexT形成配置关系，将Class名称填为Counter，并选择无限制选项，然后单击“创建Class”按钮完成Class的创建，如下图所示：![](https://upload-images.jianshu.io/upload_images/291600-47b6f370eeb9384f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
 
    点击刚刚创建的Counter，其实质是一张结构表，用来记录文章的浏览量，如下图所示，这里的表可以直接对文章阅读次数进行修改，所以如果想要追求阅读次数的读者可以在表上直接进行修改。
     
4. 配置key:在左侧导航栏的设置界面，单击“应用Key”可以看到应用的App ID和App Key。复制ID和Key，然后将其配置到主题配置文件中，在文件中找到leancloud_visitors属性，将enable设置为true，然后将之前复制的ID和Key粘贴到相应的属性中。

## 分享
一开始用的 JiaThis, 后发现发不上去就不显示。后使用的needmoreshare2。

    # NeedMoreShare2
    # This plugin is a pure javascript sharing lib which is useful in China.
    # See: https://github.com/revir/need-more-share2
    # Also see: https://github.com/DzmVasileusky/needShareButton
    # iconStyle: default | box
    # boxForm: horizontal | vertical
    # position: top / middle / bottom + Left / Center / Right
    # networks: Weibo,Wechat,Douban,QQZone,Twitter,Linkedin,Mailto,Reddit,
    #           Delicious,StumbleUpon,Pinterest,Facebook,GooglePlus,Slashdot,
    #           Technorati,Posterous,Tumblr,GoogleBookmarks,Newsvine,
    #           Evernote,Friendfeed,Vkontakte,Odnoklassniki,Mailru
    needmoreshare2:
      enable: true
      postbottom:
        enable: true
        options:
          iconStyle: default
          boxForm: horizontal
          position: bottomCenter
          networks: Weibo,Wechat,Douban,QQZone,Twitter,Facebook
     float:
        enable: false
        options:
          iconStyle: box
          boxForm: horizontal

## 评论系统
一开始折腾了使用 [友言](http://www.uyan.cc/),发现一直出不来效果，就果断放弃了（后期看看加入好的评论系统）。然后转向了简单的[Valine](https://valine.js.org/).

Valine一款快速、简洁且高效的无后端评论系统,基于Leancloud开发。具体流程：

1. 创建LeanCloud账号[LeanCloud](https://leancloud.cn/) 并获取 App ID和App Key。
2. 配置：
	
		# Valine.
    	# You can get your appid and appkey from https://leancloud.cn
    	# more info please open https://valine.js.org
     	valine:
       	enable: true
       	appid: *****
       	appkey: ****
       	notify: false # mail notifier , https://github.com/xCss/Valine/wiki
       	verify: false # Verification code
       	placeholder: Just go go # comment box placeholder
       	avatar: mm # gravatar style
       	guest_info: nick,mail,link # custom comment header
       	pageSize: 10 # pagination size 
    
## 布局大小调整
一开始用的next默认的布局，但是看到[jmyblog](http://jmyblog.top)后，发现自己的布局可以调整，就尝试调整了一下，在官方文章中设置内容区域布局：NexT 对于内容的宽度的设定如下：700px，当屏幕宽度 < 1600px；900px，当屏幕宽度 >= 1600px；移动设备下，宽度自适应。

如果你需要修改内容的宽度，同样需要编辑样式文件。 编辑主题的 source/css/_variables/custom.styl 文件，新增变量：

	// 修改成你期望的宽度
    $content-desktop = 700px

    // 当视窗超过 1600px 后的宽度
    $content-desktop-large = 900px

但是对于Pisces Scheme，需要单独配置source/css/_variables/custom.styl：

    .header{ width: 90%; }
    .container .main-inner { width: 90%; }
    .content-wrap { width: calc(100% - 260px); }

## 侧边栏调整

## 文末版权信息
## 站点访问人数
具体实现方法，在\themes\next\layout\_partials\footer.swig文件中，在copyright前加以下代码：

    <script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>

在再合适的位置添加显示统计的代码：

    <div class="powered-by">
	<i class="fa fa-user-md"></i><span id="busuanzi_container_site_uv">
	  访客:<span id="busuanzi_value_site_uv"></span>
	  浏览:<span id="busuanzi_value_site_pv"></span>
	  </span>
	</div>

## 文章字数统计 
1. 切换到根目录下，然后运行如下代码 `$ npm install hexo-wordcount --save`
2. 然后在主题的配置文件中，配置如下：
    
    	# Post wordcount display settings
    	# Dependencies: https://github.com/willin/hexo-wordcount
     	post_wordcount:
     	item_text: true
     	wordcount: true
     	min2read: true

## 修改文章底部的那个带#号的标签
修改模板/themes/next/layout/_macro/post.swig，搜索 `rel="tag">#`，将 `#` 换成`<i class="fa fa-tag"></i>`

## 网站底部次数统计
需要使用hexo-wordcount 具体步骤：

1. 切换到根目录下，然后运行如下代码 `$ npm install hexo-wordcount --save`
2. 然后在/themes/next/layout/_partials/footer.swig文件尾部加上：

## 加入百度


# hexo+next 资料
1. next官方： http://theme-next.iissnan.com/
1.  hexo的next主题个性化教程：打造炫酷网站 http://blog.csdn.net/qq_33699981/article/details/72716951
2.  一个好的布局：http://jmyblog.top/
3.  布局优化：http://blog.csdn.net/heqiangflytosky/article/details/54863185

