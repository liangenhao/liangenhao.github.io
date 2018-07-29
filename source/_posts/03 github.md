---
title: 03 github
date: 2018年7月29日
categories: Git
tags: [Git]
---

## 1 使用github作为远程库

零、【在github上创建一个远程仓库】：略

一、【创建远程库地址别名】：

[查看当前所有远程地址别名]：

```bash
git remote -v
```

[创建远程仓库地址别名]：

```bash
git remote add [别名] [远程地址]
```

> 例如：`git remote add origin https://github.com/liangenhao/test.git`

<!-- more -->

二、【推送】：

```bash
git push [别名] [分支名]
```

> 例如：`git push origin master`。推送到origin 远程仓库的master分支。

三、【克隆】：

```bash
git clone [远程地址]
```

> 例如：`git clone https://github.com/liangenhao/test.git`

克隆成功后，会完整的把远程库下载到本地，并初始化本地库，并创建origin远程地址别名。

> 注意：在github上，团队协作需要通过成员邀请的，否则修改的内容不能推送到远程库上。

四、【拉取】：

**[`pull`]：**

```bash
git pull [远程库地址的别名] [远程分支名]
```

> 注意：`pull = fetch + merge`

[`fetch`]：

```bash
git fetch [远程库地址的别名] [远程分支名]
```

> 例如：`git fetch origin master`

`fetch`操作不会修改工作区的内容。

[`merge`]：

```bash
git merge [远程库地址的别名/远程分支名]
```

> 例如：`git merge origin/master`

五、【解决冲突】：

要点：

1. 如果不是基于远程库的最新版所做的修改，不能直接推送（push），必须先拉去（pull）。
2. 拉去下来后如果进入冲突状态，则按照“分支冲突解决”操作进行解决。

## 2 github跨团队协作

一、【fork】：

首先进入别人的项目仓库，点击fork，相当于将这个仓库拷贝了一份到自己的仓库。

二、【pull request】：

fork完项目后，会在自己账号的仓库中看到这个项目。并可以对这个项目进行修改和推送。

修改完成后，就可以将修改推送给源项目。

使用pull request将修改的项目推送给源项目。

原作者在源项目下就可以看到这条pull request请求。并可以对代码进行审核，查看修改的代码。

审核完成后，就可以进行合并（merge pull request）。

## 3 SSH 登录

一、进入当前用户的家目录 

`$ cd ~ `

二、删除.ssh 目录

 `$ rm -rvf .ssh` 

三、运行命令生成.ssh 密钥目录 

`$ ssh-keygen -t rsa -C atguigu2018ybuq@aliyun.com`

> 注意： 这里-C 这个参数是大写的 C

四、进入.ssh 目录查看文件列表 

`$ cd .ssh` 

`$ ls -lF` 

五、查看 id_rsa.pub 文件内容    

`$ cat id_rsa.pub`
六、复制 id_rsa.pub 文件内容， 登录 GitHub，

点击用户头像→Settings→SSH and GPG keys、

New SSH Key 、

输入复制的密钥信息、

回到 Git bash 创建远程地址别名 `git remote add origin_ssh git@github.com:atguigu2018ybuq/huashan.git` 、  

推送文件进行测试    