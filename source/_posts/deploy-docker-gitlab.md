---
title: 部署Docker Gitlab
date: 2016-07-14 10:33:43
tags:
- Git
- Gitlab
- Docker
categories:
- Git
---

# Gitlab

　　由于公司现在 **Git** 私有服务使用的是 **Gitblit** ，虽然只有我们一个项目组在用 **Git** ，但使用下来问题不断，基本有以下几点：

1. 团队成员对 **Git** 的使用不熟悉，目前只有我一个人长期使用 **GitHub** 、 **Coding** 等服务。
2. 缺少 **Git** 工作流程的规范化。
3. 由第二点引起的缺少代码评审机制

> 对于第一点，引入 **SourceTree** 可以解决大部分问题，在操作上使用图形界面相对于 **Bash** 上敲命令容易的多。
>
> 对于第二点，起先打算引入 **Git Flow** 工作流，然而相对来说还是较为复杂，成员不一定能够理解其中奥妙。
>
> 对于第三点，如果使用 **Git Flow** 工作流的话实现不了强制性代码评审，直到后来看到了 **Gitlab** 。


<!-- more -->

　　基于以上几点，我在 Ubuntu 16.04 安装了 **Gitlab** ，具体安装过程我就不详述了，有兴趣的请点以下链接前往官网： [点我去官网](https://gitlab.com/) 使用过程中遇到了一个问题，在进入管理员页面时经常会出现 **502** 错误。详细排查之后才发现是内存不足的原因。

[![gitlab502](/images/20160714gitlab502.png)](https://jjandxa.github.io/images/20160714gitlab502.png)

试用过后认为有些优点是比较值得说的：

1. 界面美观
   [![gitlabindex](/images/20160714gitlabindex.png)](https://jjandxa.github.io/images/20160714gitlabindex.png)
2. **Gitlab Flow**
   类似于 **Github Flow** 的工作流，就问你怕没？
   **master** 分支作为保护分支，只有项目拥有者可以直接操作，成员只能在 **master** 基础上创建功能分支，开发完成后，提交 **Merge Request** ，项目拥有者同意后才能合并代码。
3. 强制性代码评审
   由上一条带来的好处就是， **Mereg Request** 时，项目拥有者可以和其他成员一起评审代码，对代码进行评论，在此期间，成员可以继续提交代码， **Merge Request** 会自动跟踪最新代码的情况。
4. **Gitlab CI** 集成
   暂时没用到，就不深入讨论了

具体而言， **Gitlab** 足够强大，然而问题来了，公司现在代码服务器在内网，系统为 **Windows Server 2008** ，而 **Gitlab** 只支持 Linux ，无奈只能等新服务器再使用 **Gitlab** 。

> 你以为这样就结束了？

# Docker

并没有，我在 Ubuntu 16.04 上安装了 **Gitlab** 后，想起对服务器来说，系统上装了很多服务，维护时总是在解决各种服务的冲突，另外服务不支持快速部署，对移植不友好等等。
由此我想到了 **Docker** ，在恶补了一两天，熟悉 **Docker** 基本使用后，便着手在 **Docker** 上部署 **Gitlab** ，由于在 **Docker** 上已经有成熟的 **Gitlab** 镜像，我们直接拿来用就可以了。[Docker-Gitlab](https://www.damagehead.com/docker-gitlab/) 在官网可以看到，部署方式特别简单，以下指出几点关键的地方：

**docker-compose**
作为一个将容器部署从复杂变为简单的工具，我一开始是不知道的，从 **Docker** 官网我们可以看到以下这两条安装命令：

```
$ curl -L https://github.com/docker/compose/releases/download/1.6.2/run.sh > /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

比较坑的是，国内运行这条命令来安装的速度，慢如乌龟。我一天都没下好，所以建议大家使用 python pip 来安装，运行如下命令：

```
//安装 pip
sudo apt-get install python-pip

//安装 docker-compose
pip install docker-compose

```

安装了 **docker-compose** 后，就需要部署 **Docker-Gitlab** 了：

```
//下载脚本
wget https://raw.githubusercontent.com/sameersbn/docker-gitlab/master/docker-compose.yml
//第一次运行，来编译，创建容器
docker-compose  up

//后面可以运行一下命令启动或停止容器
docker-compose start

docker-compose stop

```

最后，说一下如何备份 **Docker-Gitlab** 的数据，运行以下命令：

```
sudo docker run --name gitlab -it --rm \
--link [你的postgresql容器名称]:postgresql --link \ [你的redis容器名称]:redisio \
--net [docker-compose 初次运行时创建的网络名]  \
-e 'DB_ADAPTER=postgresql' \
-e 'DB_HOST=postgresql' \
-e 'DB_PORT=5432' \
-e 'DB_USER=gitlab' \
-e 'DB_PASS=password' \
-e 'DB_NAME=gitlabhq_production' \
-e 'REDIS_HOST=redis' \
-e 'REDIS_PORT=6379' \
--publish 10022:22 --publish 10080:80 \
--env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
--env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
--volume /srv/docker/gitlab/gitlab:/home/git/data \
sameersbn/gitlab:8.9.6 app:rake gitlab:backup:create

```

正常情况下你会看下以下画面：
[![docker-gitlab backup](/images/20160714dockergitlabbackup.png)](https://jjandxa.github.io/images/20160714dockergitlabbackup.png)

备份文件放在 **/srv/docker/gitlab/gitlab/backups** 目录下，更换服务器时，只需要将备份文件放到新服务器恢复，一切就完成啦

# End