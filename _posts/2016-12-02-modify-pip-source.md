---
layout: post
title: 修改 pip 源以加快 Python 模块安装速度
tags:
  - program
  - python
---

用 pip 安装 Python 模块时经常出现访问速度非常慢的情况，是因为 pip 默认镜像源是 `https://pypi.Python.org/simple/`，把这个镜像源修改成我们国内的就好了。

##  国内镜像源

目前可用的国内镜像源有：

- `http://pypi.mirrors.ustc.edu.cn/simple/` (中科大)
- `http://mirrors.aliyun.com/pypi/simple/` (阿里云)

## 修改方式

### 手动指定

第一种安装镜像源的方式是手动指定：

`pip -i http://mirrors.aliyun.com/pypi/simple/ install Flask -- trusted-host mirrors.aliyun.com`

*PS：这种方式在每次执行 pip 安装命令时都要手动指定一次。*

### 添加配置文件

第二种方式是添加配置文件。

1. Linux 或 OSX 系统
   执行以下命令：
   ```shell
   mkdir ~/.pip
   cd ~/.pip
   touch pip.conf
   ```
   在 pip.conf 文件中填写以下内容：
   ```
   [global]
   trusted-host=mirrors.aliyun.com
   index-url=http://mirrors.aliyun.com/pypi/simple/
   ```
2. Windows 系统
   在个人用户目录（`C:\Users\username\`）下创建 pip 文件夹，并在该文件夹内创建 pip.ini 文件，在文件内填入以上内容即可。

## 参考资料

- [http://blog.csdn.net/u012592062/article/details/51966649](http://blog.csdn.net/u012592062/article/details/51966649) 