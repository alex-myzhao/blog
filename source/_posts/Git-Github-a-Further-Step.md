---
title: 'Git/Github, a Further Step'
tags:
  - git
photos: images/git-github/github-social-coding.jpg
date: 2017-06-12 15:02:25
---

Github 除了可以用作 git 的远程仓库，还可以很方便地进行项目管理。本博客以一个项目开发的前期流程作为 demo，介绍 GitHub 在这方面的使用方法。

<!--more-->

## Organization

Organization 是 GitHub 中的可以被个人免费创建的虚拟组织。

**为何需要 Organization？**
Organization 简化了对团队成员项目仓库的管理。通过创建 Organization，项目成员可以集中存放代码。以电影购票网站为例，开发过程中至少会存在项目前端、后端以及一个用于存放文档和管理进度的 dashboard 3个项目，此外还可能有一些临时开发出来的小工具或者脚本等等。当这些项目混在个人仓库中，查找管理很不方便，并且可能存在明明上的冲突（如果参加的多个项目都有一个叫做“Dashboard”的仓库）。
如下图所示：我们的 GoMovie 项目中有7名成员，整合了各种项目。
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/github-organization.png)

**成员权限管理**
每个团队成员都有三个权限级别：pull only, push & pull, push & pull & administative
分别对应：只读、可读写、管理员这三种不同层次的权限。
![](https://cloud.githubusercontent.com/assets/391331/6142028/a9de0054-b163-11e4-8adb-5edc9600ff97.png)

**Organization Fork**
有权限在 Organization 中创建仓库的成员在 fork 别的项目的时候，GitHub 会提供 fork 到个人仓库还是到 organization 中的提示。

我们在开发 GoMovie 项目的时候，将开发文档单独放到 Dashboard 仓库中进行管理和共享。

## Project

在每个 repository 中，可以创建 project，用来进行团队任务规划等协作工作。
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/github-project.png)
我们在开发过程中把这个功能当作时间推进表来使用，在一个 high-level 上对之后一段时间对任务进行规划，定下了一些关键点的 DDL，然后分为 todo, in progress 和 done 三列。如下图所示：
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/github-project-detail.png)

## Wiki

Wiki 中可以用来编写项目文档，包括 API 接口文档、常见问题等，通过团队协作，编写不同的 wiki 页，可以制作出可读性高且有层次结构的文档。为了方便查看，我们在项目中将查阅频率较高等内容放在了项目的 wiki 中。
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/github-wiki.png)

## Issue

Github 中的 Issue 同样是一个强大的功能。除了在别的开源项目中看到 Issue 用于反馈 bug 和提供改进思路之外，Issue 可以用来分派任务到团队成员，并打上各种标签以加强该任务的语义。
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/github-issue.png)
Github 还提供了一套标准提交 Issue 和发起 Pull Request 的[模版](https://github.com/blog/2111-issue-and-pull-request-templates)，用来帮助用户书写规范的信息。

## 总结

GitHub 完全可以当作团队协作工具进行使用，覆盖了规划制订、文件共享、任务分派等常用功能，再加上能和项目代码完美整合到一起，如果还在将 GitHub 仅仅作为一个远程 git 仓库使用会错过很多提升团队工作效率的机会。所以下次寻找团队协作软件的时候，尝试一下 Github 吧～

## Reference
- https://github.com/blog/674-introducing-organizations
