---
title: 部署Docker-GitLab
date: 2019-05-31 15:02:55
tags: 
  - GitLab 
  - Docker 
  - CI
---

踩坑docker下部署gitlab,主要集中在gitlab的端口暴露和runner

环境: Ubuntu 18.10

---

# Docker安装

1. 更新apt

``` bash
sudo apt update
```

2. 安装相应工具

``` bash
sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg-agent \
     software-properties-common
```

3. 添加key

``` bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4. 设置repository

``` bash
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```

5. 更新apt

``` bash
sudo apt-get update
```

6. 安装Docker

``` bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Docker现已安装完成

以下步骤是将用户加入Docker用户组，避免每次sudo才能执行Docker

7. 添加docker用户组

``` bash
sudo groupadd docker
```

8. 将用户加入docker用户组

``` bash
sudo gpasswd -a ${USER} docker
```

9. 重启 docker 服务

``` bash
sudo service docker restart
```

10. 切换当前会话到docker group或重启系统

``` bash
newgrp - docker
```

# GitLab安装

1. 启动镜像并暴露端口

``` bash
sudo docker run --detach \
  --publish 4433:443 --publish 8929:8929 --publish 2222:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

将内部的https的443端口对外暴露4433、将内部的ssh的22端口对外暴露2222

**注意**: 在这里的8929:8929，后面一个8929代表Docker内部http端口，本应是80但是下一步会修改*external_url*为8929，这样才能正确显示gitlab的clone地址和让runner正确连接gitlab-ci。

2. 修改 */srv/gitlab/config/gitlab.rb* 配置文件

```
external_url 'http://192.168.8.4:8929' # 改为ip+修改后的http端口
gitlab_rails['gitlab_shell_ssh_port'] = 2222 # 改为修改后的ssh端口
```

3. 重启 gitlab container

``` bash
docker restart gitlab
```

# Runner

1. 启动runner

``` bash
 docker run -d --name gitlab-runner --restart always \
   -v /srv/gitlab-runner/config:/etc/gitlab-runner \
   -v /var/run/docker.sock:/var/run/docker.sock \
   gitlab/gitlab-runner:latest
```

2. 注册runner

``` bash 
 docker run --rm -t -i -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner register
```

按照提示步骤输入即可。

**注意**: 在输入tag的时候若为空则表示每个stage都触发

# 参考连接

- https://docs.docker.com/install/linux/docker-ce/ubuntu/
- https://docs.gitlab.com/omnibus/docker/
- https://docs.gitlab.com/runner/install/docker.html
- https://docs.gitlab.com/runner/register/index.html#docker
- https://www.jianshu.com/p/d707f70c60d2
- https://www.cnblogs.com/mafeng/p/8683914.html