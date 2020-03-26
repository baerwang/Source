# Share.js 字体图标组件

Share.js是一款一键转发工具，它可以一键分享到新浪微博、微信、QQ空间、QQ好友、腾讯微博、豆瓣、Facebook组件、Twitter、Linkedin、Google+、点点等社交网站，使用字体图标。

以下步骤可以实现微博和微信分享

（1）修改pages/gathering/item/_id.vue的脚本部分。以下代码用于引入外部的js .我们这里的js采用cdn方式引入  地址为：

https://cdn.bootcss.com/social-share.js/1.0.16/js/social-share.min.js

所需要的样式： https://cdn.bootcss.com/social-share.js/1.0.16/css/share.min.css

```js
    head: {
        script: [
            { src: 'https://cdn.bootcss.com/social-share.js/1.0.16/js/social-share.min.js' }
        ],
        link: [
            { rel: 'stylesheet', href: 'https://cdn.bootcss.com/social-share.js/1.0.16/css/share.min.css' }
        ]
    }
```

（2）修改pages/gathering/_id.vue的页面部分，在合适的位置添加分享按钮

```html
<div class="social-share"  
     data-sites="weibo,wechat" 
     data-url="http://www.itheima.com" 
     :data-title="item.name">
</div> 
```

选项：

```txt
url                 : '', // 网址，默认使用 window.location.href
source              : '', // 来源（QQ空间会用到）, 默认读取head标签：<meta name="site" content="http://overtrue" />
title               : '', // 标题，默认读取 document.title 或者 <meta name="title" content="share.js" />
description         : '', // 描述, 默认读取head标签：<meta name="description" content="PHP弱类型的实现原理分析" />
image               : '', // 图片, 默认取网页中第一个img标签
sites               : ['qzone', 'qq', 'weibo','wechat', 'douban'], // 启用的站点
disabled            : ['google', 'facebook', 'twitter'], // 禁用的站点
wechatQrcodeTitle   : '微信扫一扫：分享', // 微信二维码提示文字
wechatQrcodeHelper  : '<p>微信里点“发现”，扫一下</p><p>二维码便可将本文分享至朋友圈。</p>'
```

以上选项均可通过标签 `data-xxx` 来设置