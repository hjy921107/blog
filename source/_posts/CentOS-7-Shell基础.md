---
title: CentOS_7_Shell基础
date: 2017-09-17 23:05:27
category: Linux
tags: CentOS 7
---

<!-- toc -->

---

# 1. Shell 简介

# 2. HelloWorld

## 2.1 新建 Shell 脚本

脚本文件 hello.sh

```shell
#!/bin/bash
#This is the first program

echo "Hello Shell";
```

说明：

- Linux 中并没有扩展名的概念，但为了便于管理，建议使用 .sh 作为脚本文件的后缀
- 注释：shell 脚本中使用 # 进行注释
- `#!/bin/bash` 是固定写法，不是注释，放在文件首行，说明下面的内容遵循 bash 的基本语法。对于简单的 shell 脚本可以省略，但不建议；对于脚本中使用了其他语言的脚本，不能省略。

## 2.2 执行 Shell 脚本

- 方法一：直接使用 `bash hello.sh` 命令，但通常不这样做，建议使用方法二
+ 方法二：
	1. 修改脚本文件的执行权限 `chmod 755 hell.sh`
	2. 使用相对/绝对路径访问脚本文件 `./hello.sh` 或 `/root/hello.sh`
