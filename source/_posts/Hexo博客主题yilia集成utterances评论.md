---
title: Hexo博客主题yilia集成utterances评论
date: 2020-05-08 21:05:44
comments: true
tags:
  - git
  - hexo
---
utterances 是一款基于 GitHub issues 的评论工具。

相比同类的工具 gitment、gitalk 以及 disqus 评论工具，优点如下：
{% blockquote %}
1.极其轻量
2.加载非常快
3.配置比较简单
{% endblockquote %}

## 1.utterances 安装

首先安装这个 App ，选择要关联评论的仓库。可以选择所有仓库，也可以只选择一个仓库。选择一个仓库比较安全
<div style="width:50%;margin:auto">{% asset_img utterancesInstall.png %}</div>

## 2.utterances 使用

在themes/yilia/layout/_partial/article.ejs下添加以下代码：
```bash
<!-- 《utteranc评论：基于github issue的评论系统 -->
<% if (theme.utterance && theme.utterance.enable){ %>
    <section id="comments" class="comments">
      <style>
        .utterances{max-width: 100%;}
      </style>
        <script src="https://utteranc.es/client.js"
                repo="<%= theme.utterance.repo %>"
                issue-term="<%= theme.utterance.issue_term %>"
                theme="<%= theme.utterance.theme %>"
                crossorigin="anonymous"
                async>
        </script>
  </section>
<% } %>
<!-- utteranc评论》 -->
```
## 3.utterances配置

在themes/yilia/_config下添加以下代码：
```bash
#6.utteranc评论： https://utteranc.es (参数配置详见主页)
utterance:
  enable: true
  repo: root1761/root1761.github.io     #仓库名字,格式：你的用户ID/仓库名称，如：blog/utterance_repo
  issue_term: 'title'            #映射配置
  theme: github-light         #主题
```
{% blockquote %}
**enable：true使用utterances评论false关闭**
**repo： 你的用户ID/仓库名称，如：blog/utterance_repo**
**issue_term：映射配置,有如下选项：**
  1. pathname
  2. url
  3. title
  4. og:title
  5. issue-number
  6. specific-term
---
**theme：主题,有如下选项：**
  1. github-light
  2. github-dark
  3. github-dark-orange
  4. icy-dark
  5. dark-blue
  6. photon-dark
{% endblockquote %}

## 4.utterances

显示如下效果,说明配置成功

![](utterancesSuccess.png)
