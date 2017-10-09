---
layout: post
title: 服务器上搭建 Git 以及客户端操作
tags: 
  - program
  - git
---

实验室要在自己的服务器上搭建一个 Git 仓库，方便大家共享数据文档，于是写了一些简要操作供大家参考。

## 服务器上搭建 Git

- 在 Ubuntu 服务器上添加 Git 用户
  ```shell
  sudo adduser git
  ```
- 安装 ssh（如果未安装的话）
  ```shell
  sudo apt-get install ssh
  ```
- 在 Git 用户 home 目录下创建裸仓库（用 `git init` 的话，后面客户机 push 操作的时候可能会出问题）
  ```shell
  git --bare init BigdataGroupResouces.git
  ```
  创建好的 BigdataGroupResouces.git 就是我们的公共仓库，以后所有用户提交的共享文件都存放在这个仓库中。

## 客户端操作步骤

- 安装 Git 
  Linux 下执行 `sudo apt-get install git`，windows 下自行搜索 git for windows 下载安装即可。
- 克隆远程仓库到本地
  打开 gitbash，切换到任意工作目录下，执行以下命令:
  ```shell
  git clone git@bigdata20:/home/git/BigdataGroupResources.git
  ```
  看到提示输入密码后，输入 Git 账户的密码 `git`，就成功将远程仓库克隆到本地了。
  *注意：要在 hosts 文件里将 bigdata20 映射到相应的 IP 地址，否则就要将上面命令中的 bigdata20 替换成相应 IP 地址。*
- 提交或修改自己本地文件到服务器上
  进入到刚刚克隆到本地的仓库目录中（如果已克隆到了本地，请先执行第三步更新本地仓库），然后将自己修改过或新添加的文件放到仓库中（文档类文件放到 documentResources 文件夹下，代码类文件放到自建文件夹），执行以下命令：
  ```shell
  git add yourfilename
  git commit -m "your comment"
  git push origin master
  ```
  对本地仓库的一些其他基本操作如 `git status`、`git rm` 等，自行查看帮助文档即可。
  *注意：第一次使用 Git 会提示设置邮箱和用户名，输入以下命令：*
  ```shell
  git config --global user.email "youremailaddress"
  git config --global user.name "yourusername"
  ```
- 更新本地仓库（即从服务器上获取更新存到本地）
  ```shell
  git pull
  ```