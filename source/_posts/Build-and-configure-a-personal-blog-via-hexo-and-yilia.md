---
title: Build and configure a personal blog via hexo and yilia on github
date: 2022-02-22 16:15:03
toc: true
declare: true
tags: 
  - hexo
  - blog
---

> Technical blog has a hundred benefits and no harm
>
> This blog records the process of building and customizing this personal blog from 0 to 1

> Unfortunately, the yilia theme has been no longer updating and it is too buggy right now. I switch to [other theme.](/2022/04/30/Switch-blog-theme-to-FLUID/)

<!-- more -->

## Preliminary

### Why personal blog

> Keeping a technical blog can be **a great way of documenting your growth as a developer**. This documentation can be particularly useful on a professional level. All software companies want to hire smart, thoughtful, communicative developers who can easily assimilate into a team, and who are ready to both teach and learn

### Static vs dynamic blog

there are 2 types of mainstream personal blog: static and dynamic.

Static is recommended considering its simplicity, 0 maintenance and 0 safety worry. 

|              | Static blog                                                  | Dynamic blog                                                 |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Price        | low, 0 cost when the traffic is relatively low               | High, server is needed, cloud server of  high performance is usually very expensive. |
| Features     | Limited, only third-party services can be used to complete certain "dynamic" functions, such as comments | Rich, in WordPress for example, basically any kind of plugins can be found. Featuers such as auto-resizing, media players, multiple authors, scheduled posts, user analysis can be easily realize. |
| Speed        | Fast                                                         | Slow                                                         |
| Maintainance | 0                                                            | Need to care about the sever                                 |
| Markdown     | Supported yet it's the only choice                           | not supported                                                |
| Geeky        | YES                                                          | No                                                           |

And I also chose static because I'm geeky (poor of money) and results-driven (lazy to spend time on maintaining).

## Build a static blog via hexo

### Set environment

1.check machine information: macOS on M1 MacBook

2.Install Nodejs, including node and npm

open  https://nodejs.org/en/download/ and click download

<img src="node js install.png" alt="node js install" style="zoom:50%;" />

3.Install Git

Aready installed

4.Open terminal, check node, npm and git versions

```shell
$ npm -v
$ node -v
$ git --version
8.3.1
v16.14.0
git version 2.32.0 (Apple Git-132)
```

### Initialize blog

1.install hexo via npm

```shell
$ sudo npm install -g hexo-cli
$ hexo -v
INFO  Validating config
hexo: 6.0.0
hexo-cli: 4.3.0
os: darwin 21.2.0 12.1

node: 16.14.0
v8: 9.4.146.24-node.20
uv: 1.43.0
zlib: 1.2.11
brotli: 1.0.9
ares: 1.18.1
modules: 93
nghttp2: 1.45.1
napi: 8
llhttp: 6.0.4
openssl: 1.1.1m+quic
cldr: 40.0
icu: 70.1
tz: 2021a3
unicode: 14.0
ngtcp2: 0.1.0-DEV
nghttp3: 0.1.0-DEV
```

2.create a new folder in terminal and initialize the blog

```shell
$ cd ~/Documents/
$ makedir self_blog
$ cd self_blog/
$ hexo init
INFO  Cloning hexo-starter https://github.com/hexojs/hexo-starter.gitINFO  Install dependencies???#########?????????????????????????????? ??? idealTree:hexo-front-matter: timing idealTree:node_modules/hexo-front-matter Completed in 212msINFO  Start blogging with Hexo! 
```

3.view the blog on localhost, s for start

```shell
$ hexo s
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/ . Press Ctrl+C to stop.
```


### Write first blog

1.write a new blog, n for new

```shell
$ hexo n 'Hello ShouRou'
INFO  Validating config
INFO  Created: ~/Documents/self_blog/source/_posts/Hello-ShouRou.md
```

the blog can be written on any editor, Typora in use.

```shell
open ~/Documents/self_blog/source/_posts/Hello-ShouRou.md
```

2.clean cache(not necessary)

```shell
$ hexo clean
```

3.generate the blog, g for generate

```shell
$ hexo g
INFO  Validating config
INFO  Start processing
INFO  Files loaded in 61 ms
(node:10719) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(Use `node --trace-warnings ...` to show where the warning was created)
(node:10719) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:10719) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
(node:10719) Warning: Accessing non-existent property 'lineno' of module exports inside circular dependency
(node:10719) Warning: Accessing non-existent property 'column' of module exports inside circular dependency
(node:10719) Warning: Accessing non-existent property 'filename' of module exports inside circular dependency
INFO  Generated: archives/2022/index.html
INFO  Generated: archives/index.html
INFO  Generated: js/script.js
INFO  Generated: fancybox/jquery.fancybox.min.css
INFO  Generated: index.html
INFO  Generated: css/style.css
INFO  Generated: css/fonts/fontawesome-webfont.woff2
INFO  Generated: fancybox/jquery.fancybox.min.js
INFO  Generated: js/jquery-3.4.1.min.js
INFO  Generated: archives/2022/02/index.html
INFO  Generated: css/fonts/FontAwesome.otf
INFO  Generated: css/fonts/fontawesome-webfont.woff
INFO  Generated: css/fonts/fontawesome-webfont.eot
INFO  Generated: css/fonts/fontawesome-webfont.ttf
INFO  Generated: css/images/banner.jpg
INFO  Generated: 2022/02/22/hello-world/index.html
INFO  Generated: css/fonts/fontawesome-webfont.svg
INFO  Generated: 2022/02/22/Hello-ShouRou/index.html
INFO  18 files generated in 161 ms
```

### Deploy to remote (GitHub)

1.Create a new repository with the name of $username.github.io

<img src="github page.png" alt="github page" style="zoom:50%;" />

use default setting

2.open terminal, install plugin of deploying to git

```shell
$ npm install --save hexo-deployer-git
```

3.Open the `_config.yml` file in the blog `root` directory, add these lines afterwards

```yml
deploy:
 type: git
 repo: git@github.com:DaydreamAtNight/DaydreamAtNight.github.io.git
 branch: master
```

4.Go to the blog `root`, deploy the blog to remote, d for deploy

```shell
$ hexo clean
$ hexo g
$ hexo d
```

Open https://daydreamatnight.github.io/ to see if it works

## Change theme to yilia

default theme of hexo is called landscape and it's not beautiful enough to most of the people. Yilia is a fast, simple, elegant and popular theme. Thought it has not been updated since Nov 2017, it still a good choice for fresh bloggers.

### Download and deploy yilia

1.go to the blog `root`

``` shell
$ git clone https://github.com/litten/hexo-theme-yilia theme/yilia
```

2.eidt the `_config.yml` file, add

```yml
theme: yilia
```

3.clean and deploy hexo

```shell
$ hexo clean
$ hexo g
$ hexo d
```

### Basic customize yillia

#### Activate `aboutme` ???left slider??? button

1.go to terminal run

```shell
$ npm i hexo-generator-json-content --save
```

2.go to the blog `root` directory, add these lines to the `_config.yml` file

```yml
jsonContent:
    meta: false
    pages: false
    posts:
      title: true
      date: true
      path: true
      text: false
      raw: false
      content: false
      slug: false
      updated: false
      comments: false
      link: false
      permalink: false
      excerpt: false
      categories: false
      tags: true
```

#### Customize avatar

put the avatar file in directory  `themes/yilia/source/img`

> do not add to the public repository directly, or the img get cleaned every time running `hexo clean` , need to upload to the same dir again after this command.

#### Set favicon (icon on the tab of website)

put the favicon img in directory `themes/yilia/source/img`

[Bitbug](https://www.bitbug.net/) is a way of converting image into .ico file.

#### Other configuration

Set file of yillia is in `themes/yilia/_config.yml` as:

```yml
# Header
author: Ryan LI
subtitle: 'Daydreaming at night'
menu:
  main: /
  archives: /archives/index.html
  learn: /tags/learn/

# SubNav
subnav:
  github: "https://github.com/DaydreamAtNight"
  # weibo: "#"
  # rss: "#"
  # zhihu: "#"
  #qq: "#"
  #weixin: "#"
  #jianshu: "#"
  #douban: "#"
  #segmentfault: "#"
  #bilibili: "#"
  #acfun: "#"
  mail: "mailto:lishoushou2019@gmail.com"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"

rss: /atom.xml

# ?????????????????? root ??????
# ???????????????????????????????????????????????? http://yoursite.com/blog???
# ???????????? url ?????? http://yoursite.com/blog ?????? root ?????? /blog/???
root: /

# Content

# ?????????????????????????????????
# excerpt_link: more
# ?????????????????????????????????????????????????????????false
show_all_link: 'show all'
# ????????????
mathjax: false
# ??????????????????????????????
open_in_new: false

# ??????
# ??????type?????????0-??????????????? 1-???????????????md????????????reward:true???????????????????????? 2-????????????????????????
reward_type: 0
# # ??????wording
# reward_wording: '????????????????????????'
# # ????????????????????????????????????????????????????????????????????????/assets/img/alipay.jpg
# alipay: 
# # ???????????????????????????
# weixin: 

# ??????
# ???????????????0-?????????????????? 1-???????????????md????????????toc:true???????????????????????? 2-???????????????????????????
toc: 1
# ?????????????????????????????????????????????????????????????????????????????????true????????????hexo??????????????????????????????false
toc_hide_index: true
# ????????????????????????
toc_empty_wording: 'directery none exist'

# ????????????????????????????????????
top: true

# Miscellaneous
baidu_analytics: ''
google_analytics: ''
favicon: /img/favicon.ico

#????????????url
avatar: /img/avatar.jpeg

#??????????????????
# share_jia: true

# #?????????1????????????2?????????????????????3????????????4???Disqus???5???Gitment
# #??????????????????????????????????????????false???????????????
# #???????????????wiki???https://github.com/litten/hexo-theme-yilia/wiki/

# #1?????????
# duoshuo: false

# #2??????????????????
# wangyiyun: false

# #3?????????
# changyan_appid: false
# changyan_conf: false

# #4???Disqus ???hexo????????????config?????????disqus_shortname?????????????????????yilia???
# disqus: false

# #5???Gitment
# gitment_owner: false      #?????? GitHub ID
# gitment_repo: ''          #??????????????? repo
# gitment_oauth:
#   client_id: ''           #client ID
#   client_secret: ''       #client secret

# ???????????? - ?????????????????????????????????????????????????????????
style:
  # ???????????????????????????
  header: '#ece0cf'
  # ??????????????????
  slider: 'linear-gradient(45deg,#b4a698,#ece0cf)'

# slider?????????
slider:
  # ??????????????????tags??????
  showTags: false

# ????????????
# ????????????????????????????????????false
# ??????
#smart_menu:
#  friends: false
smart_menu:
  innerArchive: 'All articles'
  # friends: '??????'
  aboutme: 'About me'

# friends:
#   ????????????1: http://localhost:4000/
#   ????????????2: http://localhost:4000/
#   ????????????3: http://localhost:4000/
#   ????????????4: http://localhost:4000/
#   ????????????5: http://localhost:4000/
#   ????????????6: http://localhost:4000/

aboutme: Stay hungry, stay fullish
```

## Advance customize

### Stop visit litten.me:9005

Sometimes the user's client information is collected, see [here](https://github.com/litten/hexo-theme-yilia/issues/528) for details. 

Stop reporting by clear the contents in `themes/yilia/source-src/js/report.js`

### Limit display numbers on the main page

Simply insert `<! -- more -->` to show only what comes before it while collapse the afterwards,  click on the article title to read it in full.

### Easily add pics to blogs via hexo-renderer-marked plugin

1.find `post_asset_folder ` in `_config.yml` file in the blog `root` directory, set to be true

```yml
post_asset_folder:true
```

2.Install plugin

```shell
npm install hexo-renderer-marked --save
```

3.change `_config.yml` in blog `root` directory as

```yml
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

then img can be easily add with `![img description](img.png)` after add the image to the folder with the same name as the article in `/source/_posts/`

4.change Typora pereference as

<img src="typora setting.png" alt="typora setting" style="zoom:50%;" />

img can drag into typro, yet blogname need to be deleted before deploying

### Show number of articles and words on the left panel

1.add wordcount plugin in terminal

```shell
npm i --save hexo-wordcount
```

2.change `themes/yilia/layout/_partial/left-col.ejs` 

after

```html
<nav class="header-menu">
  <ul>
    <% for (var i in theme.menu){ %>
      <li><a href="<%- url_for(theme.menu[i]) %>"><%= i %></a></li>
    <%}%>
  </ul>
</nav>
```

add

```html
<span class="post-count"><%=site.posts.length%> articles
			<span><%= totalcount(site, '0,0.0a') %></span> words</span>
```

add style sheet in `themes/yilia/source/main.0cf68a.css`

```
.post-count{
  font-size: 12px;
  color: #696969;
}
```

### Show number of visits in the footer

[busuanzi](https://busuanzi.ibruce.info/) is in use, which is super easy to deploy

change `themes/yilia/layout/_partial/footer.ejs` as

```html
<footer id="footer">
  <div class="outer">
    <div id="footer-info">
    	<div class="footer-left">
    		<!-- total visits number -->
          <% if (theme.busuanzi && theme.busuanzi.enable){ %>
            <!-- busuanzi statistics -->
            <span id="busuanzi_value_site_pv"></span>&nbsp;visits in total
            <script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
          <% } %>
        <!-- end -->
    	</div>
      	<div class="footer-right">
      		&copy; <%= date(new Date(), 'YYYY') %> <%= config.author || config.title %>
      	</div>
    </div>
  </div>
</footer>
```

and add 

```yml
busuanzi:
  enable: true
```

### Add button of hiding the left panel

Refer to  [hexo yilia????????????????????????????????????](https://cqh-i.github.io/2019/08/07/hexo-yilia%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E9%9A%90%E8%97%8F%E5%B7%A6%E8%BE%B9%E6%A0%8F%E7%9B%AE%E6%8C%89%E9%92%AE/ )  

1.add style list to file ` /themes/yilia/source/main.0cf68a.css `

```css
/*stylesheet for hide the left panel*/
.mymenucontainer {
	display:block;
	cursor:pointer;
	left:0;
	top:0;
	width:35px;
	height:35px;
	z-index:9999;
	position:fixed;
}
.bar1 {
	width:35px;
	height:3px;
	border-radius:3px;
	background-color:#8E6D51;
	margin:6px 0;
	transition:0.1s;
	-webkit-transform:rotate(-45deg) translate(-8px,8px);
	transform:rotate(-45deg) translate(-8px,8px);
}
.bar2 {
	width:35px;
	height:3px;
	border-radius:3px;
	background-color:#8E6D51;
	margin:6px 0;
	transition:0.1s;
	opacity:0;
}
.bar3 {
	width:35px;
	height:3px;
	border-radius:3px;
	background-color:#8E6D51;
	margin:6px 0;
	transition:0.1s;
	-webkit-transform:rotate(45deg) translate(-4px,-6px);
	transform:rotate(45deg) translate(-4px,-6px);
}
.change .bar1 {
	-webkit-transform:rotate(0deg) translate(0px,0px);
	transform:rotate(0deg) translate(0px,0px);
}
.change .bar2 {
	opacity:1;
}
.change .bar3 {
	-webkit-transform:rotate(0deg) translate(0px,0px);
	transform:rotate(0deg) translate(0px,0px);
}
/*stylesheet for hide the left panel end*/
```

2.go to ` /themes/yilia/layout/layout.ejs ` add before `  <div class="left-col"  ` 

```html
<div class="mymenucontainer" onclick="myFunction(this)">
  <div class="bar1"></div>
  <div class="bar2"></div>
  <div class="bar3"></div>
</div>
```

3.add between ` </body> ` and ` </html> ` 

```js
<script>
    var hide = false;
    function myFunction(x) {
        x.classList.toggle("change");
        if(hide == false){
            $(".left-col").css('display', 'none');
            $(".mid-col").css("left", 6);
            $(".tools-col").css('display', 'none');
            $(".tools-col.hide").css('display', 'none');
            hide = true;
        }else{
            $(".left-col").css('display', '');
            $(".mid-col").css("left", 300);
            $(".tools-col").css('display', '');
            $(".tools-col.hide").css('display', '');
            hide = false;
        }
    }
</script>
```





### Beautiful contents navigation in articles

Default navigator is kind of ugly so found a more beautiful version, to use default version, simply change `toc: 2` in file `themes/yilia/_config.yml`

1.add this block at the end of `themes/yilia/source/main.0cf68a.css`

```css
/* navigator */
#container .show-toc-btn,#container .toc-article{display:block}
.toc-article{z-index:100;background:#fff;border:1px solid #ccc;max-width:250px;min-width:150px;max-height:500px;overflow-y:auto;-webkit-box-shadow:5px 5px 2px #ccc;box-shadow:5px 5px 2px #ccc;font-size:12px;padding:10px;position:fixed;right:35px;top:129px}.toc-article .toc-close{font-weight:700;font-size:20px;cursor:pointer;float:right;color:#ccc}.toc-article .toc-close:hover{color:#000}.toc-article .toc{font-size:12px;padding:0;line-height:20px}.toc-article .toc .toc-number{color:#333}.toc-article .toc .toc-text:hover{text-decoration:underline;color:#2a6496}.toc-article li{list-style-type:none}.toc-article .toc-level-1{margin:4px 0}.toc-article .toc-child{}@-moz-keyframes cd-bounce-1{0%{opacity:0;-o-transform:scale(1);-webkit-transform:scale(1);-moz-transform:scale(1);-ms-transform:scale(1);transform:scale(1)}60%{opacity:1;-o-transform:scale(1.01);-webkit-transform:scale(1.01);-moz-transform:scale(1.01);-ms-transform:scale(1.01);transform:scale(1.01)}100%{-o-transform:scale(1);-webkit-transform:scale(1);-moz-transform:scale(1);-ms-transform:scale(1);transform:scale(1)}}@-webkit-keyframes cd-bounce-1{0%{opacity:0;-o-transform:scale(1);-webkit-transform:scale(1);-moz-transform:scale(1);-ms-transform:scale(1);transform:scale(1)}60%{opacity:1;-o-transform:scale(1.01);-webkit-transform:scale(1.01);-moz-transform:scale(1.01);-ms-transform:scale(1.01);transform:scale(1.01)}100%{-o-transform:scale(1);-webkit-transform:scale(1);-moz-transform:scale(1);-ms-transform:scale(1);transform:scale(1)}}@-o-keyframes cd-bounce-1{0%{opacity:0;-o-transform:scale(1);-webkit-transform:scale(1);-moz-transform:scale(1);-ms-transform:scale(1);transform:scale(1)}60%{opacity:1;-o-transform:scale(1.01);-webkit-transform:scale(1.01);-moz-transform:scale(1.01);-ms-transform:scale(1.01);transform:scale(1.01)}100%{-o-transform:scale(1);-webkit-transform:scale(1);-moz-transform:scale(1);-ms-transform:scale(1);transform:scale(1)}}@keyframes cd-bounce-1{0%{opacity:0;-o-transform:scale(1);-webkit-transform:scale(1);-moz-transform:scale(1);-ms-transform:scale(1);transform:scale(1)}60%{opacity:1;-o-transform:scale(1.01);-webkit-transform:scale(1.01);-moz-transform:scale(1.01);-ms-transform:scale(1.01);transform:scale(1.01)}100%{-o-transform:scale(1);-webkit-transform:scale(1);-moz-transform:scale(1);-ms-transform:scale(1);transform:scale(1)}}.show-toc-btn{display:none;z-index:10;width:30px;min-height:14px;overflow:hidden;padding:4px 6px 8px 5px;border:1px solid #ddd;border-right:none;position:fixed;right:40px;text-align:center;background-color:#f9f9f9}.show-toc-btn .btn-bg{margin-top:2px;display:block;width:16px;height:14px;background:url(http://7xtawy.com1.z0.glb.clouddn.com/show.png) no-repeat;-webkit-background-size:100%;-moz-background-size:100%;background-size:100%}.show-toc-btn .btn-text{color:#999;font-size:12px}.show-toc-btn:hover{cursor:pointer}.show-toc-btn:hover .btn-bg{background-position:0 -16px}.show-toc-btn:hover .btn-text{font-size:12px;color:#ea8010}
.toc-article li ol, .toc-article li ul {
    margin-left: 30px;
}
.toc-article ol, .toc-article ul {
    margin: 10px 0;
}
```

2.after `</header><% } %>` in file `themes/yilia/layout/_partial/article.ejs` add

```html
    <!-- navigator -->
    <% if (!index && post.toc){ %>
      <p class="show-toc-btn" id="show-toc-btn" onclick="showToc();" style="display:none">
            <span class="btn-bg"></span>
            <span class="btn-text">...</span>
            </p>
      <div id="toc-article" class="toc-article">
          <span id="toc-close" class="toc-close" title="hide navigator" onclick="showBtn();">??</span>
          <strong class="toc-title">navigator</strong>
            <%- toc(post.content) %>
          </div>
    <script type="text/javascript">
      function showToc(){
          var toc_article = document.getElementById("toc-article");
          var show_toc_btn = document.getElementById("show-toc-btn");
          toc_article.setAttribute("style","display:block");
          show_toc_btn.setAttribute("style","display:none");
          };
      function showBtn(){
          var toc_article = document.getElementById("toc-article");
          var show_toc_btn = document.getElementById("show-toc-btn");
          toc_article.setAttribute("style","display:none");
          show_toc_btn.setAttribute("style","display:block");
          };
    </script>
        <% } %>
    <!-- navigator end -->
```

3.add `toc:true` to the articles that need the navigator.

### Add custormize header to articles

when run `hexo new` to initiate a new blog, a defaul head would generate, change it by

change the `scaffolds/post.md` in the `root` directory

```txt
---
title: {{ title }}
date: {{ date }}
author: daydreamatnight
toc: true
declare: true
tags:
---
```

#### more headers to choose when writing a blog

before a blog, more paras can be chosen to add

```txt
--- 
title: #????????????????????? 
toc: ture #toc 
date: 2020-09-07 09:25:00 #???????????? 
author: GavenLee #?????? 
img: /source/images/xxx.jpg #?????? 
top: true #???????????? 
cover: true #???????????????????????? 
coverImg: /images/1.jpg #???????????? 
password: #??????????????????????????? 
mathjax: false #mathjax 
summary: ??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? 
categories: Markdown #?????? 
tags: #?????? 
abbrlink: HexoLearn #?????? 
---
```

### Disable auto wrap in code block

locate and delete `white-space:pre-wrap` in file `themes/yilia/source/main.0cf68a.css` 

### Add copy button to code block

1.create a `clipboard_use.js` file in directory `themes/yilia/source` 

```js
$(".highlight").wrap("<div class='code-wrapper' style='position:relative'></div>");
/*create copy button after page loaded*/
!function (e, t, a) {
    /* code */
    var initCopyCode = function () {
        var copyHtml = '';
        copyHtml += '<button class="btn-copy" data-clipboard-snippet="">';
        copyHtml += '  <i class="fa fa-clipboard"></i><span>copy</span>';
        copyHtml += '</button>';
        $(".highlight .code").before(copyHtml);
        var clipboard = new ClipboardJS('.btn-copy', {
            target: function (trigger) {
                return trigger.nextElementSibling;
            }
        });
        clipboard.on('success', function (e) {
            e.trigger.innerHTML = "<i class='fa fa-check' style='color:green'></i><span style='color:green'>copy success</span>"
            setTimeout(function () {
                e.trigger.innerHTML = "<i class='fa fa-clipboard'></i><span>copy</span>"
            }, 1000)
            e.clearSelection();
        });
        clipboard.on('error', function (e) {
            e.trigger.innerHTML = "<i class='fa fa-exclamation' style='color:red'></i><span style='color:red'>copy success</span>"
            setTimeout(function () {
                e.trigger.innerHTML = "<i class='fa fa-clipboard'></i><span>copy</span>"
            }, 1000)
            e.clearSelection();
        });
    }
    initCopyCode();
}(window, document);
```

2.load .js file, edit `themes/yilia/layout/layout.ejs` file, add before `</body>`. 

```html
<!-- copy button in code block-->
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/clipboard@2.0.4/dist/clipboard.js"></script>
<script type="text/javascript" src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
<script type="text/javascript" src="/clipboard_use.js"></script>
```

3.add stylesheet the end of `themes/yilia/source/main.0cf68a.css`

```css
/* code copy button */
.btn-copy {
  display: inline-block;
  cursor: pointer;
  background-color: #eee;
  background-image: linear-gradient(#fcfcfc, #eee);
  border: 1px solid #d5d5d5;
  border-radius: 3px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  -webkit-appearance: none;
  font-size: 13px;
  font-weight: 700;
  line-height: 20px;
  color: #333;
  -webkit-transition: opacity .3s ease-in-out;
  -o-transition: opacity .3s ease-in-out;
  transition: opacity .3s ease-in-out;
  padding: 2px 6px;
  position: absolute;
  right: 5px;
  top: 5px;
  opacity: 0;
}
.btn-copy span {
  margin-left: 5px;
}
.highlight:hover .btn-copy {
  opacity: 1;
}
/* code copy button end */
```

4.add copy button icon, edit `themes/yilia/layout/_partia/head.ejs` add before `</head>` 

```html
<!-- copy button icon -->
<link rel="stylesheet" type="text/css" href="//cdn.bootcss.com/font-awesome/4.6.3/css/font-awesome.min.css">
```

### Allow search engines to index this Blog

#### index Google to this Blog

check if google can find you, enter `site:daydreamatnight.github.io` to see

<img src="check google search.png" alt="check google search" style="zoom:50%;" />

##### Add url to goole search console

1.open [google console](https://search.google.com/search-console/welcome) , add URL link of the blog (https://daydreamatnight.github.io), in the `URL prefix` block, click `CONTINUE`

<img src="google search console.png" alt="google console" style="zoom:50%;" />

2.upload the html file to the blog `root` directory and deploy the website, then clicke verify.

<img src="google console varification.png" alt="google console varification" style="zoom:50%;" />

little buggy here, see [**don???t upload** the file **using hexo** command](https://hilyy.xyz/how-to-allow-google-to-index-your-hexo-blog-website-google-search-console-verification-methods/)

##### add sitemap for google

add sitemap for google and baidu together

> A *sitemap* is a file where you provide information about the pages, videos, and other files on your site, and the relationships between them. Search engines like Google read this file to more intelligently crawl your site. A sitemap tells Google which pages and files you think are important in your site, and also provides valuable information about these files: for example, for pages, when the page was last updated, how often the page is changed, and any alternate language versions of a page.

1.install sitemap plugins	

```shell
$ npm install hexo-generator-sitemap --save
$ npm install hexo-generator-baidu-sitemap --save
```

2.add to the `_config.yml` in the blog `root` 

```yml
# hexo sitemap
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml
```

3.Deploy the blog, go to  https://daydreamatnight.github.io/sitemap.xml and https://daydreamatnight.github.io/baidusitemap.xml to see if sitemaps are uploaded

4.Go to Google Search Console, in the left panel, click `Sitemaps`, enter your sitemap URL `sitemap.xml` 

<img src="can't fetch sitemap.png" alt="can't fetch sitemap" style="zoom:50%;" />

Googlebot won't download the sitemap immediately. Give it time. 

##### add robots.txt

> A robots.txt file tells search engine crawlers which URLs the crawler can access on your site. This is used mainly to avoid overloading your site with requests; **it is not a mechanism for keeping a web page out of Google**. To keep a web page out of Google, [block indexing with `noindex`](https://developers.google.com/search/docs/advanced/crawling/block-indexing) or password-protect the page.
>
> A robots.txt file is used primarily to manage crawler traffic to your site, and *usually* to keep a file off Google, depending on the file type:

```txt
User-agent: *
Allow: /
Allow: /archives/
Allow: /tags/
Allow: /categories/
Allow: /about/
Allow: /guestbook/
Allow: /others/


Disallow: /js/
Disallow: /css/
Disallow: /lib/

Sitemap: https://daydreamatnight.github.io/sitemap.xml
Sitemap: https://daydreamatnight.github.io/baidusitemap.xml
```

deploy the blog and wait.

##### check if sitemap is available

After uploaded several updates, my sitemap still didn't fetched by google. So I went to check, it turns out my url setting in `_config.yml` is wrong.  So I changed it to be my home url. And check it with  [URL Inspection Tool](https://www.jcchouinard.com/url-inspection-tool/). 

1.Open [google search console](https://search.google.com/search-console), add the url of sitemap in the upper url inspecting box.

<img src="Google%20sitemap%20inspect.png" alt="Google sitemap inspect URL is not on Google " style="zoom:22%;" />

<img src="Google%20sitemap%20inspect%202.png" alt="Google sitemap inspect 2" style="zoom:22%;" />

It's normal it shows `URL is not on Google` because it shouldn't as a sitemap.

2.click `live test` to check the availability.

<img src="Google%20sitemap%20inspect%203.png" alt="Google sitemap inspect 3" style="zoom:75%;" />

It should be available, then just wait.

#### index Bing to this Blog

1.go to [Bing webmaster](https://www.bing.com/webmasters/) and login

2.connect with google webmaster.

<img src="bing%20sitemap.png" alt="bing sitemap" style="zoom:75%;" />

  

<img src="being%20sitemap%20connect%20google.png" alt="being sitemap connect google" style="zoom:75%;" />

  

<img src="bing%20search%20console%20success.png" alt="bing search console success" style="zoom:75%;" />

#### index baidu to this Blog(not possibly working)

go to the [baidu search console](https://ziyuan.baidu.com/site/index) , 

<img src="baidu console.png" alt="baidu console" style="zoom:50%;" />

Click `????????????` and input every thing, do similar thing

<img src="baidu console varification.png" alt="baidu console varification" style="zoom:50%;" />

add sitemap

<img src="baidu console sitemap.png" alt="baidu console sitemap" style="zoom:50%;" />

just wait forever, this could take 2000 years, so give up

### Add copyright statement

1.open file `themes/yilia/layout/_partial/article.ejs` add before `<% if ((theme.reward_type === 2 || (theme.reward_type === 1 && post.reward)) && !index){ %>`

```html
<!-- add copyright statement -->
<% if(theme.declare){%>
    <%- partial('post/declare') %>
<% } %>
<!-- end -->
```

2.create new file `declare.ejs` under `themes/yilia/layout/_partial/post/` with:

```html
<!--add copyright statement https://github.com/JoeyBling/hexo-theme-yilia-plus/commit/c1215e132f6d5621c5fea83d3c4f7ccbcca074a3-->
<%
  var sUrl = url.replace(/index\.html$/, '');
  sUrl = /^(http:|https:)\/\//.test(sUrl) ? sUrl : 'https:' + sUrl;
%>

<!-- #copyright setting???0-close statement; 1-declare statement if declare: true in the article header; 2-always declare the copyright -->
<% if ((theme.declare.declare_type === 2 || (theme.declare.declare_type === 1 && post.declare)) && !index){ %>
  <div class="declare">
    <strong class="author">author: </strong>
    <% if(config.author != undefined){ %>
      <%= config.author%>
    <% }else{%>
      <font color="red">please add right "author" name in "_config.yml" in the blog root</font>
    <%}%>
    <br>
    <strong class="create-time">posting date: </strong>
    <%- date(post.date, 'YYYY-MM-DD HH:MM:SS') %>
    <br>
    <strong class="update-time">last update: </strong>
    <%- date(post.updated, 'YYYY-MM-DD HH:MM:SS') %>
    <br>
    <strong class="article-titles">article title: </strong>
    <a href="<%= config.url %>/<%= post.path %>" title="<%= post.title %>" target="_blank"><%= post.title %></a>
    <br>
    <strong class="article-url">article link: </strong>
    <a href="<%= config.url %>/<%= post.path %>" title="<%= post.title %>" target="_blank"><%= config.url %>/<%= post.path %></a>
    <br>
    <strong class="copyright">copyright:</strong>
    This work is licensed under a
    <a rel="license" href="<%= theme.declare.licensee_url%>" title="<%= theme.declare.licensee_alias %>"><%= theme.declare.licensee_name%></a>
    licience 
    <% if(theme.declare.licensee_img != undefined){ %>
      <a rel="license" href="<%= theme.declare.licensee_url%>"><img alt="????????????????????????" style="border-width:0" src="<%= theme.declare.licensee_img%>"/></a>
    <% } %>
  </div>
<% } else {%>
  <div class="declare" hidden="hidden"></div>
<% } %>
<!-- add copyright statement -->
```

3.add stylesheet the end of `themes/yilia/source/main.0cf68a.css` 

```css
/*stylesheet for the delcare*/
.declare {
  background-color: #eaeaea;
  margin-top: 2em;
  border-left: 3px solid #ff1700;
  padding: .5em 1em; 
}
/*stylesheet for the delcare end*/
```

4.add at the end of `themes/yilia/_config.yml` file:

```
declare:
  declare_type: 1
  licensee_url: http://creativecommons.org/licenses/by-nc-sa/4.0/      
  licensee_name: 'CC BY-NC-SA 4.0'                              
  licensee_alias: 'CC BY-NC-SA 4.0'     
  licensee_img: https://i.creativecommons.org/l/by-nc-sa/4.0/80x15.png
```

### Add mind-map support

```shell
npm install hexo-markmap
```

Detailed in its [Github](https://github.com/MaxChang3/hexo-markmap)

Example:

```
{% markmap 300px %}
- Testa
  - test1
  - test2
- Testb
  - test1
  - test2
{%endmarkmap%}
```

### Add Latex math support

Change the renderer to the more powerful pandoc:

1.Install pandoc on macOS:

```
copybrew install pandoc
```

2.in the blog root directory uninstall the default renderer then install the pandoc renderer:

```
copynpm uninstall hexo-renderer-marked --save
npm install hexo-renderer-pandoc --save
```

3.install the hexo math plugin

```
copynpm install hexo-math --save
```

4.add these lines to the hexo `_config` file

```
copymarkdown:
  plugins:
    - markdown-it-footnote
    - markdown-it-sup
    - markdown-it-sub
    - markdown-it-abbr
    - markdown-it-emoji
    - hexo-math
```

5.add these lines to the theme `_config` file

```
copy# MathJax Support
mathjax:
  enable: true
  per_page: true
```

6.rebuild the to blog see changes

7.Examples: $this_{is}an\frac{inline}{equation}$
$$
\begin{equation}
    \mathbf{K}_\mathbf{1}=\frac{1}{\Delta r}\ \left[\begin{matrix}\begin{matrix}-1&1\\-1&1\\\end{matrix}&\ &\ \\\begin{matrix}\ &\ddots\\\end{matrix}&\begin{matrix}\ddots&\ \\\end{matrix}&\ \\\ &-1\ &1\\\end{matrix}\right],\ \ {\ \mathbf{K}}_\mathbf{2}=\frac{1}{\Delta r}\ \left[\begin{matrix}\begin{matrix}-1&1\\\ &\ddots\\\end{matrix}&\begin{matrix}\\\ddots\\\end{matrix}&\ \\\begin{matrix}\ &\ \\\end{matrix}&-1&1\ \\\ &-1\ &1\\\end{matrix}\right]
    \label{K2}
\end{equation}
$$

### The last snapshot

Ok, never spend time on a no-longer maintained project. Here's the last figure of it.

<img src="Last snapshot.png" alt="Last snapshot" style="zoom:80%;" />

## Reference

https://flatironschool.com/blog/the-benefits-of-blogging-how-and-why-to-keep-a-technical-blog/

https://weblog.masukomi.org/2015/10/18/static-vs-dynamic-blogging/

https://www.cnblogs.com/aoguai/p/11781505.html

https://www.kblog.top/post/30452.html

https://wkzqn.gitee.io/2020/02/16/typora%E7%BC%96%E5%86%99hexo%E5%8D%9A%E5%AE%A2%E6%97%B6%E7%9A%84%E5%9B%BE%E7%89%87%E6%98%BE%E7%A4%BA/

https://segmentfault.com/a/1190000009478837#articleHeader5

https://hilyy.xyz/how-to-allow-google-to-index-your-hexo-blog-website-google-search-console-verification-methods/

https://busuanzi.ibruce.info/

https://creativecommons.org/choose/results-one?license_code=by-nc-sa&amp;jurisdiction=&amp;version=4.0&amp;lang=en

https://www.jcchouinard.com/sitemap-could-not-be-read-couldnt-fetch-in-google-search-console/

https://cqh-i.github.io/2019/08/07/hexo-yilia%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E9%9A%90%E8%97%8F%E5%B7%A6%E8%BE%B9%E6%A0%8F%E7%9B%AE%E6%8C%89%E9%92%AE/