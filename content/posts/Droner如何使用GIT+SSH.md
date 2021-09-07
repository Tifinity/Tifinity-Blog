---
title: "Drone如何使用GIT+SSH"
date: 2021-09-07T10:21:24+08:00
draft: false
categories: ["未分类"]
tags: [""]
featured_image: 
author: "Tifinity"
---

# Drone如何使用GIT+SSH

> 慎终如始，则无败事。 ——《道德经》

在本文开始之前，我已经按官网的说明将 drone server 和 runner 都成功的部署在自己的服务器上。部署完之后创建了一条更新 github.io 的流水线，然而现在 github 的 https 通道弱不禁风，跑十次“失联”九次，所以需要换成用 ssh 来 git clone 和 push。

我们都知道 CI 流水线的第一步一般是 在 drone 的官方文档中并没有很直接的说明如何使用 ssh 来连接 git，只说明可以禁用默认的 clone 步骤并[自定义 clone 流程](https://docs.drone.io/pipeline/docker/syntax/cloning/)，但并没看到有 ssh key 相关的参数或者环境变量之类的，不是很清楚怎么使用。

随后找到一篇博客，muxueqz 大佬的[静态博客持续集成大法](https://muxueqz.top/my-blog-ci.html)，看到他是自己写了 drone 的插件来定义参数和调用命令的，不过该博客写于2019年，于是我坚信这个步骤应该有并且可能两年时间已经优化了 ssh 的更简单的使用方法，于是又去找别的办法。

在 [Drone官方讨论区](https://discourse.drone.io/) 搜索了一下 git ssh，找到一些帖子：

这两个是 19 年的提问以及解决，还有一些更久远的就没看了

[提问1](https://discourse.drone.io/t/clone-by-ssh-not-http-git-server-port-override/5408/2)  [提问2](https://discourse.drone.io/t/custom-clone-with-ssh/6149)

官方在 [github](https://github.com/drone/drone/blob/782688872970bc7dea53f167ce4181458892369e/.drone.sh#L9:L17) 上给出的脚本:

~~~bash
# write the ssh key.
mkdir /root/.ssh
echo -n "$SSH_KEY" > /root/.ssh/id_rsa
chmod 600 /root/.ssh/id_rsa

# add github.com to our known hosts.
touch /root/.ssh/known_hosts
chmod 600 /root/.ssh/known_hosts
ssh-keyscan -H github.com > /etc/ssh/ssh_known_hosts 2> /dev/null
~~~

基本解决方法就是自己定义一个流水线环境变量然后用 from_secret 传入，然后当我使用的时候出现了一个问题：

![截屏2021-09-07 15.27.44](https://i.loli.net/2021/09/07/DcHvCOB2YUmZgVf.png)

报错私钥的格式不对，第一反应是换行，文件结束符之类的问题，因为 ssh key 是在 web 页面上直接粘贴进去的。再三确认我的 ssh key 是准确无误并具有权限的，然后找了很多关于这个报错的信息，试了很多解决办法，包括：

1. 将 ssh 手动用 \n 连接在一起再复制到 secret 的 value 中。
2. 用 drone-cli 来写入 secret：`drone secret add --repository muxueqz/muxueqz-blog --name GIT_PUSH_SSH_KEY --data @/tmp/iam_logs/pages_id_rsa`，其中@是文件语法，这也是 muxueqz 使用的方法，但在我这里还是报错“invalid format”。
3. 用了一些 drone 的 git 插件，包括 plugins/gh-pages，plugins/git-action:1，muxueqz/drone-git-push-by-bash，无一例外均报错。
4. 用官方给出的写入命令。

[提问3](https://discourse.drone.io/t/i-cant-use-drone-secret/9411)，这个是21年1月的一个老哥，跟我一样的问题，最终他换了另外的 CI，有点尴尬。

最后发现，echo -n 会把 ssh key 写成一行，于是格式错误，不知道为何官网会用这个参数，改成 -e 后无论是直接粘贴还是用 /n 连接都可行。

|                      | 不带参数         | -e   | -n               |
| -------------------- | ---------------- | ---- | ---------------- |
| 直接粘贴             | ✅                | ✅    | ❌ invalid format |
| 用/n连接为一行字符串 | ❌ invalid format | ✅    | ❌ invalid format |

以下为我的 .drone.yml 文件中 clone 的部分：

~~~yaml
kind: pipeline
type: docker
name: sync-tifinity-io

clone:
  disable: true
  
- name: clone
  image: centos
  environment:
    SSH_KEY:
      from_secret: SSH_KEY
  commands:
    - yum install -y git
    - yum install -y openssh-clients
    - mkdir $HOME/.ssh
    - echo -e "$SSH_KEY" > $HOME/.ssh/id_rsa
    - chmod 600 $HOME/.ssh/id_rsa
    - touch $HOME/.ssh/known_hosts
    - chmod 600 $HOME/.ssh/known_hosts
    - ssh-keyscan -H github.com > $HOME/.ssh/known_hosts
		# 带子模块的最好用这个参数，而不是先clone再update，否则子模块的clone默认用的还是 https
    - git clone --recursive git@github.com:Tifinity/Tifinity-Blog.git
  when:
    branch:
    - master
~~~

经过漫长的调试过程之后总算成功。

<img src="https://i.loli.net/2021/09/07/MnYaNwfED96sIc2.png" alt="截屏2021-09-07 16.21.42" style="zoom:50%;" />

