---
title: 使用hexo搭建github Pages个人博客
date: 2019-10-31 15:54:00
tags:
- 其他
---
### **环境配置**
* node.js:需要使用 npm 下载
* github账号 以及 git

本文不再介绍 node.js 和 git 下载

### **安装Hexo**

可以使用 npm 或者 yarn ，建议使用yarn，npm容易出错
全局安装 hexo
```
npm install -g hexo
```
有 `yarn` 的可以使用 
```
yarn add hexo -S
```
#### 新建项目，进入项目路径初始化
```
hexo init
```

#### 安装依赖
```
npm install hexo-server --save
或者
yarn install
```
如果npm安装报错，可以使用 yarn

#### 启动本地服务
```
hexo s  或者 hexo server
```
默认使用 4000 端口，可以点链接进去看一下

#### 绑定github page
新建仓库，名称为 `你的用户.github.io`。（请替换 你的用户）
然后创建仓库

#### 配置 ssh 密钥
回到终端，查看本机是否有 ssh key
```
cd ~/.ssh
```
如果没有 
```
$ssh-keygen -t rsa -C "your_email@example.com"
#这将按照你提供的邮箱地址，创建一对密钥
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): [Press enter]
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]
```
三次回车就行
```
Your identification has been saved in /c/Users/you/.ssh/id_rsa.
Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
The key fingerprint is:
01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```
代表 ssh 密钥已经创建成功,然后将公钥添加到github中即可
```
clip < ~/.ssh/id_rsa.pub
```
将公钥中内容复制到粘贴板，然后 进入github，点开刚创建的仓库，进入settings 选择 ssh keys 粘贴密钥即可

#### 测试是否成功
```
ssh -T git@github.com
```
如果是
```
The authenticity of host 'github.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
```
输入 yes，然后配置个人信息
```
$ git config --global user.name "cym"//用户名
$ git config --global user.email  "liji.anchang@163.com"//填写自己的邮箱
```

#### 配置
打开项目，进入 `_config.yml`文件
```yml
# Site
title: Tony
subtitle:
description:
keywords:
author: Tony乙
language: en
timezone:
```
`title` 为网页title，`author`就是你的名字

找到 deploy 并配置,将repository改为你的
```yml
deploy:
  type: git
  repository: git@github.com:tony2051/tony2051.github.io.git
  branch: master
```
在文件最后添加
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
      categories: true
      tags: true
```
安装部署到github的插件
```
npm install hexo-deployer-git --save
```

#### hexo常用命令
```
hexo new "文档" // 创建名称为 文档的md文件，默认在 source/_posts下
hexo new page "tags" //创建 tags 页面，会在 source/下创建新文件夹 tags,以及index.md
hexo g 生成静态文件
hexo d 部署到绑定的github仓库
hexo s 启动本地预览
```
### 下载 yilia 模版

#### 安装
在项目路径下
```
git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```
会创建 themes 文件夹，这个就是 yilia 主题所在路径

#### 配置
进入 themes 进入 yilia 文件，找到 _config,yml 并进入，里面就是 主题的配置

```yml
menu:
  主页: /
  # 随笔: /tags/随笔/
  分类: /categories
  # 标签: /tags
  归档: /archives
```
menu 对应的是你的博客头像下的链接，以及在项目中的路径，比如分类你需要在你项目的 `source`目录下创建 `categories`目录(不要手动创建哦)

```yml
# SubNav
subnav:
  github: https://github.com/tony2051
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
  #mail: "mailto:litten225@qq.com"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"
```
这里是左侧的导航，有github，微博等，后面跟上链接就行

```yml
#你的头像url
avatar: /WechatIMG1.jpeg
```
这个是修改头像的 图片放到 yilia/source 下

```yml
smart_menu:
  innerArchive: '所有文章'
  friends: false
  aboutme: '关于我'
```
这里是图标上的菜单栏

在 yilia/layout/layout.ejs下添加下面代码
```js
<body>
  <div id="container" q-class="show:isCtnShow">
    <canvas id="anm-canvas" class="anm-canvas"></canvas>
    <div class="left-col" q-class="show:isShow">
      <%- partial('_partial/left-col', null, {cache: !config.relative_link}) %>
    </div>
    <div class="mid-col" q-class="show:isShow,hide:isShow|isFalse">
      <%- partial('_partial/mobile-nav', null, {cache: !config.relative_link}) %>
      <div id="wrapper" class="body-wrap">
        <div class="menu-l">
          <div class="canvas-wrap">
            <canvas data-colors="#eaeaea" data-sectionHeight="100" data-contentId="js-content" id="myCanvas1" class="anm-canvas"></canvas>
          </div>
          <div id="js-content" class="content-ll">
            <%- body %>
          </div>
        </div>
      </div>
      <%- partial('_partial/footer') %>
    </div>
    <%- partial('_partial/after-footer') %>
    <%- partial('_partial/tools') %>
    <%- partial('_partial/viewer') %>
  </div>
</body>
</html>
```

### **添加评论功能**
 `yilia/_config.yml` 里提供了5种评论插件接口，但是前两个都关闭了，所以用 gitment

 首先先注册个`OAuth Application`账号 地址<a href="https://github.com/settings/applications/new">https://github.com/settings/applications/new</a> 
 四个输入框,
 `Application name` 写你的名字，随便写
 `Homepage URL`和`Authorization callback URL`写博客地址，不是仓库地址

 打开 `yilia/_config.yml`,填写下面四项
 ```yml
 #5、Gitment
gitment_owner:     #你的 GitHub ID
gitment_repo:         #仓库地址
gitment_oauth:
  client_id: ''           #client ID
  client_secret: ''       #client secret

 ```
打开`yilia/layout/_partial/post/gitment.ejs`，替换为下面内容
```javascript
<div id="gitment-ctn"></div> 
<link rel="stylesheet" href="https://billts.site/extra_css/gitment.css">
<script src="https://billts.site/js/gitment.js"></script>
<script>
var gitment = new Gitment({
  id: "<%=page.title%>",
  owner: '<%=theme.gitment_owner%>',
  repo: '<%=theme.gitment_repo%>',
  oauth: {
    client_id: '<%=theme.gitment_oauth.client_id%>',
    client_secret: '<%=theme.gitment_oauth.client_secret%>',
  },
})
gitment.render('gitment-ctn')
</script>
```

最后使用 
```
hexo g
hexo d
```
就可以查看自己的博客啦