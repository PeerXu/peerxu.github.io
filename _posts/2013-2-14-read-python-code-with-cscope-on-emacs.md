---
layout: post
title: 利用cscope阅读python代码
---

本文是教你怎么使用 cscope+emacs来阅读 python代码, 网上已经有很多使用 cscope+emacs阅读 c/c++代码的教程. 但是没有找到python的, 故有了本文.

最近在阅读 quantum的代码, 必须有一个能够使用的代码阅读器. 正所谓工欲善其事,必先利其器.

[cscope][1]是一个代码索引数据库.

[xcscope][2]是 cscope在 emacs上面的一个插件, 提供构建 cscope数据库到代码跳转一条龙服务.

安装 cscope<br>

    $ sudo apt-get install cscope
	
修改 xcscope.el, 以提供 python-mode支持(我的xcscope.el在/usr/share/emacs/site-lisp/下)

    ...
    (add-hook 'c-mode-hook (function cscope:hook))
	(add-hook 'c++-mode-hook (function cscope:hook))
	(add-hook 'dired-mode-hook (function cscope:hook))
	*(add-hook 'python-mode-hook (function cscope:hook))* ; 添加 xcscope的 python-mode 支持
	(provide 'xcscope)
	...

启用 xcscope

    $ cat ~/.emacs
	...
	;;; xcscope plugin
	(load-file "/usr/share/emacs/site-lisp/xcscope.el")
	(require 'xcscope)

生成 cscope数据库

    $ cd ~/opt/openstack/quantum/ # quantum路径
	$ find . -name "*.py" | cscope -Rbq -i - # 生成 python文件的 cscope数据库

打开 emacs, enjoy it.

下面是一些常用的 cscope快捷键

	C-c s s         Find symbol.
	C-c s d         Find global definition.
	C-c s g         Find global definition (alternate binding).
	C-c s G         Find global definition without prompting.
	C-c s c         Find functions calling a function.
	C-c s C         Find called functions (list functions called from a function).
	C-c s t         Find text string.
	C-c s e         Find egrep pattern.
	C-c s f         Find a file.
	C-c s i         Find files #including a file.

	C-c s b         Display *cscope* buffer.
	C-c s B         Auto display *cscope* buffer toggle.
	C-c s n         Next symbol.
	C-c s N         Next file.
	C-c s p         Previous symbol.
	C-c s P         Previous file.
	C-c s u         Pop mark.

[1]: http://en.wikipedia.org/wiki/Cscope "cscope wiki"
[2]: http://linux.die.net/man/1/xcscope "xcscope 介绍"
