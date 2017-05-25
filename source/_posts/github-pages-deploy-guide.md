---
title: Github Pages & Hexo 博客搭建教程
date: 2017-03-24 22:31:57
tags:
- Git
- Hexo
- Github Pages
categories:
- Git
typora-root-url: ../../source
---

# 为什么要写博客？

　　之所以以这个标题开头，正是因为这篇文章主要讲的便是基于　Github Pages 服务与 Hexo 静态博客框架搭建博客的教程。博客是什么？在现在这个信息爆炸的时代，我们每天接收到的信息多不胜数。在这海量的信息里有大部分是无用的，我认为在吸收这些信息的同时，能够表达出自身想法、整合并输出优秀的内容正是我们所欠缺的。所以便在这里献上这篇文章，拥有一个博客也是你表达自身想法的第一步。权当抛砖引玉了 ^_^

<!-- more -->

# 环境要求

> 1. Github 账户
> 2. Git 客户端
> 3. NodeJS
> 4. 文本编辑器

# 使用 Github Pages 服务

1. 创建一个以自己 Github 用户名加上 github.io 的仓库

   > 例：jjandxa 是我的 Github 用户名，所以需要创建一个名为 jjandxa.github.io 的仓库

2. 测试 Github Pages 服务是否正常

   往该仓库推送一个 index.html 页面，并访问 用户名.github.io 域名

   > 这里我以一个测试账户来测试

   ![1](/images/hexo-github-pages/1.png)

   ![2](/images/hexo-github-pages/2.png)

# 安装 Hexo

1. 安装 NodeJS ，这一步比较简单，这里就不再赘述了。

   > 这里有一个需要注意的点， Windows 下安装 NodeJS 时需要勾选 Add Path 。如果忘记勾选了，就需要自己配置环境变量。

2. NPM 是 NodeJS 的包管理器，但是 NPM 官方仓库的速度很慢。建议安装淘宝的 CNPM ，在命令行输入：

   ```
   npm install cnpm -g
   ```

3. 使用 CNPM 安装 Hexo

   ```
   cnpm install hexo -g
   ```

4. 使用 hexo 创建一个博客

   ```
   hexo init blog
   ```

   > 创建博客时可能需要等很久，这是因为在安装 hexo 需要的依赖。当看到提示 **Install Dependencies** 时就可以 **Ctrl + C** 取消安装了

5. 安装依赖

   ```
   cd blog
   cnpm install
   ```

6. 启动服务

   ```
   hexo s
   ```

7. 访问 Hexo ，打开浏览器访问 http://localhost:4000

   ![3](/images/hexo-github-pages/3.png)

# 部署博客到 Github Pages

1. 安装 hexo git 部署依赖

   ```
   npm install hexo-deployer-git --save
   ```

2. 修改博客配置文件 **_config.yml**

   ```
   deploy:
     type: git
     repo: <repository url>
   ```

   > 该配置默认会部署到 master 分支

3. 执行部署命令

   ```
   hexo d
   ```

4. 完成后即可直接以你的仓库名称为域名来访问你的博客啦！

# 其他

关于如何配置博客的页面标题、作者、描述、菜单等等请浏览 Hexo 的[官方网站](https://hexo.io/zh-cn/)。

关于如何配置主题请在 Hexo 官网的主题列表挑选一款主题，然后在查看该主题的文档来配置。这里以 Next 主题为例，请访问 Next 的[官方网站](http://theme-next.iissnan.com/)查看文档。