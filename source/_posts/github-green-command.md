---
title: 【github】为了让自己变得更绿
tags: github
categories: git-command
copyright: ture
---

---

![github-yu](http://p95stksgt.bkt.clouddn.com/github-yu.jpeg)

## 问题

作为一个程序员，看着自己”绿油油”的github提交记录，一定是一件令人高兴的事。

然而，用了这么久的github，我今天才注意到，我之前的所有commit都没有被”绿”，只有一些创建项目时的记录，这让我很奇怪。不过github用户记录一定是与邮箱相绑定的。

于是，我查看了一下commit的详细记录:

![github](http://p95stksgt.bkt.clouddn.com/github.jpg)

好吧，原来是我在公司的电脑上使用的gitconfig绑定的是公司邮箱，然而github上是自己的谷歌邮箱。

so，let’s fix this problem！

## solution

1. 全局修改

```
vim ~/.gitconfig 修改配置

或者

git config --global user.email you@example.com


```

2. 局部修改

```
git config user.email you@example.com

```

3. 也可以修改提交的用户名和Email

```
git commit --amend --email='you@example.com'

```

## 总结

git config是用于进行一些配置设置，有三种不同的方式来指定这些配置适用的范围：

- git config 针对一个git仓库
- git config –global 针对一个用户
- sudo git config –system 针对一个系统，因为是针对整个系统的，所以必须使用sudo