---
layout: post
date: 2015-08-05 00:00:00
title: "使用docker搭建gitlab环境"
description: ""
category: 技术
tags: 
  - gitlab
  - docker
---

###安装环境

`lsb_release -a`

    No LSB modules are available.
    Distributor ID: Ubuntu
    Description:    Ubuntu 14.04.1 LTS
    Release:        14.04
    Codename:       trusty

###安装Docker

    wget -qO- https://get.docker.com/ | sh
    sudo pip install -U docker-compose

###安装gitlab

    mkdir -p /srv/docker/gitlab/postgresql
    mkdir -p /srv/docker/gitlab/gitlab
    mkdir -p /srv/docker/gitlab/redis
    cd /srv/docker/gitlab
    wget https://raw.githubusercontent.com/sameersbn/docker-gitlab/master/docker-compose.yml
    docker-compose up


###后继维护

*   修改配置: 如果想修改一些配置，直接修改`docker-compose.yml`这个配置即可
    - docker链接: https://registry.hub.docker.com/u/sameersbn/gitlab/
    - github链接: https://github.com/sameersbn/docker-gitlab

*   后台启用: `sudo docker-compose up -d`

*   查看日志: 查看docker运行日志`sudo docker-compose logs`

*   结束运行: `sudo docker-compose stop`

*   删除容器: `sudo docker-compose rm -f` 删除容器不会影响下次启动

