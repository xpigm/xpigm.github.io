---
layout: post
title:  从一个Git仓库提交代码到另一个Git仓库
categories: tool
tags: [git]
no-post-nav: true
comments: true
---

# 从一个Git仓库提交代码到另一个Git仓库

1. 在Git上创建一个新的仓库 **repo_new**

2. 将需要同步代码的仓库 **repo**  `clone`至本地

3. （按照惯例）修改默认remote名称 origin -> upstream

   `git remote rename origin upstream`

4. 将新建仓库**repo_new**的URL添加至本地工作仓库的remote中

   `git remote add origin ANOTHER_REPO_URL`

   添加后，可以使用`git remote -v`查看详情

5. 提交master分支至新建仓库

   `git push origin master`


如有后续的push 可直接运行`git pull upstream master || git push origin master`



*相关资料*

1. [Git - git-remote Documentation](https://www.git-scm.com/docs/git-remote)

