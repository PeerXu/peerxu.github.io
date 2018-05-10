---
layout: post
title: 用树莓派与github构建自己的DDNS服务(上)
---

*[DDNS][1]* -- 动态域名解析服务, 是一种把互联网域名指向可变IP地址的系统.
这次我们的域名提供商选择了[github][2].

github上提供了一个静态页面服务[gh-pages][7], 我们就是利用这个服务来达到DDNS的目的的.

你需要一下几样东西:

* Raspberry Pi
* 有基础的[git][6]使用经验
* github 帐号
* 在Raspbeery Pi上设定github的SSH keys, 并提交代码时不需要输入密码. **如果你的SSH key有密码的话,可以使用[ssh-agent][4]配置免密码提交**

开始我们的旅程吧!

### server端配置

1. 登录github, fork [slipper][3]
1. clone该项目到 `~/.slipper` 目录, `git clone git@github.com/<你的github帐号>/slipper.git ~/.slipper`.
1. 进入到 `.slipper` 目录. `cd ~/.slipper`.
1. 创建一个名为*gh-pages*的分支, `git checkout -b gh-pages`
1. 复制`server/update.sh`到当前目录`~/.slipper`. `cp server/update.sh .`
1. 进入[cron][5]定时任务配置, `crontab -e`
1. 将update.sh脚本配置到定时执行中 `*/5 * * * * ~/.slipper/update.sh`, 保存退出

server端配置已经完毕, 过几分钟, 你用curl获取你的Rpi运行IP `curl http://<你的github帐号>.github.com/slipper/addr`

下一篇将会为大家介绍client端的设置和应用场景. 如自建博客域名解析, ssh反向连接等.

[1]: http://zh.wikipedia.org/wiki/%E5%8B%95%E6%85%8BDNS "动态DNS“
[2]: https://www.github.com/ "github"
[3]: https://www.github.com/peerxu/slipper "DDNS服务项目”
[4]: http://clonn.blogspot.com/2012/03/ssh-agent.html "ssh-agent教程 需翻墙"
[5]: http://linux.vbird.org/linux_basic/0430cron.php "cron"
[6]: http://www-cs-students.stanford.edu/~blynn/gitmagic/intl/zh_cn/ "git教程"
[7]: http://pages.github.com/ "github pages"
