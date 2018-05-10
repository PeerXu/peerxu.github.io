---
layout: post
title: git备忘录
---

1. *删除远程分支*

    `git push <remote> :<branch>`

example: 删除origin仓库上的backup分支 `git push origin :backup`

2. *通过指定ssh端口号添加远程仓库*

    `git remote add ssh://<username>@<hostname>:<port>/<project_path>`

example: 添加用户名为pi, 端口为15002, 主机名为raspi.me, 项目路径为 /home/pi/project.git

    `git remote add ssh://pi@raspi.me:15002/home/pi/project.git`
