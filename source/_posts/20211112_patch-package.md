---
title: patch-package
date: 2021-11-12 17:05:00
comments: true
toc: true
banner_img: /images/posts/20211112_patch-package/banner.jpg
index_img: '/images/posts/20211112_patch-package/banner.jpg'
categories: '随笔'
tags:
	- node
---

## 场景

假设 packages 中的某一个依赖包【A】有 Bug，影响线上开发，应该怎么办？

此时有一个简单的方案，临时将修复文件纳入工作目录，可以解决这个问题

- 我们可以将该依赖包 clone 出来，命名为【_A】，然后进行修改。
- 将该包【_A】放到项目目录中，并纳入版本管理
- 修改项目中的依赖，指向我们修改后的依赖包

问题：

当我们所依赖的包更新了版本，并且我们需要同步更新版本以使用新的特性时，我们的依赖包已无法获取更新了。如果要使用的话只能重复上面的操作。

此时，我们的主角登场了

## patch-package

patch-package 会将你的修改进行记录，并记录到{workRoot}/patches/[package-name+version].patch 文件中，当你再次安装依赖的时候会通过该补丁文件对依赖包进行修复。

接下来我们来一起看一下实际效果：

本例使用 js-moment 举例说明

- 首先我们安装 patch-package

``` bash
npm install --save patch-package
```

- 修改 `package.json` 添加 `postinstall` 命令，注意此步骤不可忽略！

![添加scripts](/images/posts/20211112_patch-package/step1.jpg)

- 打开目标依赖文件，进行修改

本例中修改 README.md 文件

![修改依赖](/images/posts/20211112_patch-package/step2.gif)

- 执行 `patch-package` 创建补丁文件

``` bash
npx patch-package package-name
```

![创建补丁文件](/images/posts/20211112_patch-package/step2.gif)

此时根目录会生成patches目录，存放补丁文件

![补丁文件](/images/posts/20211112_patch-package/step2-2.jpg)

- 将补丁文件添加到版本管理，此步骤是为了保证补丁文件不会丢失

** 至此，我们的整个配置已经完成 **

测试：

首先我们删除 `node_modules` 文件，然后重新 `install`，检查新的依赖包是否有我们的修改

``` bash
rm -rf node_modules

npm install
```

![测试结果](/images/posts/20211112_patch-package/test.jpg)

如图，修改成功生效

## 参考文献

[patch-package](https://github.com/ds300/patch-package)  
[使用patch-package定制node_modules 中的依赖包](https://blog.csdn.net/qq_32429257/article/details/111051217)
