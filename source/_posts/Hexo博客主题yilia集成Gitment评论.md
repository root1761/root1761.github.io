---
title: Hexo博客主题yilia集成Gitment评论
date: 2020-05-05 20:56:54
comments: true
tags:
 - hexo
 - git
---
1.博客目录下使用npm下载gitment插件
``` bash
npm install --save gitment
```
2.git注册OAuth Application
地址:登陆git——>点击头像——>settings——>Developer settings ——>OAuth Apps——>New OAuth App
{% blockquote %}
Application name: 应用名
Homepage URL: 应用程序主页的完整URL
Application description: 应用说明
Authoriztion callback URL: 授权回调URL
{% endblockquote %}

<div style="width:50%;margin:auto">{% asset_img OAuth.png %}</div>
3.修改hexo主题yilia的_config配置
``` bash
gitment_owner: root1761      #你的 GitHub ID
gitment_repo: 'root1761.github.io'          #存储评论的 repo
gitment_oauth:
  client_id: ''           #client ID
  client_secret: ''       #client secret
```
<div style="width:50%;margin:auto">{% asset_img clientId.png %}</div>

4.重新编译运行
``` bash
hexo s -g

```



