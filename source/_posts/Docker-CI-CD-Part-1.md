---
title: Docker CI/CD Part 1
tags:
  - docker
  - jenkins
date: 2017-06-13 15:01:37
---

## Docker 基础知识

Docker 使用 Google 公司推出的 Go 语言 进行开发实现，基于 Linux 内核的 cgroup，namespace，以及 AUFS 类的 Union FS 等技术，对进程进行封装隔离，属于**操作系统层面**的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也被称为容器。

### 容器和虚拟机

刚刚接触容器的初学者容易将其和虚拟机弄混，网络上也有很多在两者之间做的比较。然而，追溯到本质和源头，两者的不同之处在于：传统虚拟机技术是**先虚拟出一套硬件**，然后在其上运行一个完整操作系统，在该系统上再运行所需应用进程，对外表现就像一台真实的计算机；而容器内的应用进程**直接运行于宿主的内核**，内部没有自己的内核，而且也没有进行硬件的虚拟化，对外表现就像一个普通的进程。
如下图所示：

![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/vm.png)
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/docker.png)

在这里我们可以做一个简单的实验：运行一个基于 ubuntu bash 的容器 *caffe*，然后查看该容器下运行的进程：
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/caffe.png)
发现只有 bash 和 ps 这两个进程在执行，这意味着当我们使用容器时，只是虚拟化了执行相应程序需要用到的环境。而不需要的部分并没有被虚拟化，因此大大减少了开销。

也正是因为不需要进行硬件虚拟以及运行完整操作系统等额外开销，容器技术效率更高、启动更快，同时具有虚拟机在不同硬件下虚拟化出相同环境的能力；虽然对于初学者来说有一定学习成本，但整体上降低了部署难度，在当下大受欢迎。

### 镜像，容器和仓库

要入门 Docker，首先需要理解 docker 中镜像 (Image)，容器 (Container) 和 仓库(Repository) 这三个核心概念。

- 镜像
  - 一种特殊的文件系统，提供容器运行时所需的程序、库、资 源、配置等文件以及一些为运行时准备的一些配置参数。
  - 通过分层构建的方式在不同镜像间复用共同的部分
  - 可以理解成没有进行实例化的容器
- 容器
  - 本质为进程，具有创建、启动、停止、删除、暂停的生命周期
  - 类似于沙箱，具有与宿主机独立的命名空间，其中的进程与宿主机隔离
  - 每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层
- 仓库
  - 仓库是一个集中的存储、分发镜像的服务
  - 通过仓库可以在多台计算机间方便地共享定制的镜像，或者给多人进行分享

### Docker 安装配置

Docker 的安装步骤还是比较简单的，根据[官网上的教程](https://store.docker.com/search?type=edition&offering=community)一步步安装即可，这里不再赘述。

由于国内网访问 Docker 原生镜像比较慢且容易丢包，这里顺便提一下为 Docker 进行加速器的配置，以阿里云和 mac 下的 Docker 为例：

1. 首先到阿里云登录[容器Hub服务](https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fcr.console.aliyun.com%2F%3Fspm%3D5176.100239.blogcont29941.12.QdYNk6&lang=zh)的控制台
2. 在左侧的加速器帮助页面，找到你独立分配的加速地址
3. 在 docker 的 preference 中，找到 Daemon 下的 registry mirror 添加地址即可
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/docker2.png)

### dockerfile 脚本

Dockerfile 是一个文本文件，其内包含了一条条的指令，每一条指令会构建镜像的一层，因此每一条指令的内容，就是描述该层应当如何构建。通过这种方式我们可以根据自己的需要定制 docker 镜像，方便使用。

dockerfile 支持的指令包括：
- FROM 指定基础镜像
- RUN 执行命令行命令
- COPY 复制文件
- BUILD 构建镜像

通过 dockerfile 我们可以将已有项目自动化构建成一个 docker 镜像，这样部署的时候就只需要从仓库中把之前构建好的镜像 pull 下来，就可以直接运行，免去了环境不兼容、安装依赖的繁琐过程。

## Jenkins 配置

下面我们通过 Docker 搭建 Jenkins 服务器，使其为我们的期末项目提供持续集成和持续部署的功能。
具体持续集成和持续部署的概念可以参考[这个网站](http://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/)，我们的思路是通过 Jenkins 在 GitHub 上的钩子，监测到新的 commit 提交到 master 分支后，自动化执行测试脚本，若能够通过即与项目原有代码整合。

### 使用 Docker 安装

有了 docker 之后安装起来非常容易，注意需要暴露8080端口让宿主机可以访问到：

```bash
docker pull jenkins:latest
docker run -d --name myjenkins -p 8080:8080 -v ${pwd}/data:/var/jenkins_home jenkins
```

### NGINX 反向代理

HTTP 默认的80端口只有一个，因此当存在多个网站都需要部署到同一个主机上时，如果不想指定端口进行访问，我们可以通过 NGINX 反向代理服务器作为中转，实现同一台主机上通过不同域名访问到不同服务器到效果。例如：两个域名 `a.example.cn`, `b.example.cn` 都指向同一IP地址，只需要通过配置 NGINX 就可以让客户端访问这两个域名时进入不同到两个网站。我们这里只是 NGINX 的一种用法，NGINX 服务器和各种 web 应用服务器还可以进行分布式地部署。具体 NGINX 和反向代理的细节在此就不赘述了，可以参考 [NGINX 官网](http://nginx.org/)获取更多信息。
放在反向代理之后的 Jenkins 的配置如下，值得注意的时其中涉及的路径需要根据实际情况进行更改：

{% codeblock jenkins.conf %}
server {
  listen          80;       # Listen on port 80 for IPv4 requests

  server_name     jenkins.example.com;

  #this is the jenkins web root directory (mentioned in the /etc/default/jenkins file)
  root            /var/run/jenkins/war/;

  access_log      /var/log/nginx/jenkins/access.log;
  error_log       /var/log/nginx/jenkins/error.log;
  ignore_invalid_headers off; #pass through headers from Jenkins which are considered invalid by Nginx server.
  location ~ "^/static/[0-9a-fA-F]{8}\/(.*)$" {

    #rewrite all static files into requests to the root
    #E.g /static/12345678/css/something.css will become /css/something.css
    rewrite "^/static/[0-9a-fA-F]{8}\/(.*)" /$1 last;
  }

  location /userContent {
        #have nginx handle all the static requests to the userContent folder files
        #note : This is the $JENKINS_HOME dir
	root /var/lib/jenkins/;
        if (!-f $request_filename){
           #this file does not exist, might be a directory or a /**view** url
           rewrite (.*) /$1 last;
	   break;
        }
	sendfile on;
  }

  location @jenkins {
      sendfile off;
      proxy_pass         http://127.0.0.1:8080;
      proxy_redirect     default;
      proxy_http_version 1.1;

      proxy_set_header   Host             $host;
      proxy_set_header   X-Real-IP        $remote_addr;
      proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      proxy_max_temp_file_size 0;

      #this is the maximum upload size
      client_max_body_size       10m;
      client_body_buffer_size    128k;

      proxy_connect_timeout      90;
      proxy_send_timeout         90;
      proxy_read_timeout         90;

      proxy_buffer_size          4k;
      proxy_buffers              4 32k;
      proxy_busy_buffers_size    64k;
      proxy_temp_file_write_size 64k;
}

  location / {

     # Optional configuration to detect and redirect iPhones
      if ($http_user_agent ~* '(iPhone|iPod)') {
          rewrite ^/$ /view/iphone/ redirect;
      }

      try_files $uri @jenkins;
   }
}
{% endcodeblock %}

### 配置

完成反向代理的配置后，访问 jenkins.example.com 即可进入 jenkins 的界面，然后逐一完成安装插件、设置密码等操作。
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/jenkins.png)

## Future Work

本篇博客主要针对性地解决了 docker 入门知识，jerkins 服务器搭建以及反向代理的设置问题，在后续的博客中我们将逐渐对一个电影购票网站项目进行 CI/CD 的实现。
