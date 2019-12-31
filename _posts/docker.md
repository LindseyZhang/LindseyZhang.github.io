Docker

容器化有利于 CI/CD 的顺利进行。因为进行了容器化，应用不在具有系统依赖，更新可以推送到分布式应用到任意地方。

### 三个概念

image: 一个包含可运行应用所需所有资源，如代码，运行时环境，依赖包，环境变量，配置文件等的可运行的包。

```
An image is an executable package that includes everything needed to run an application--the code, a runtime, libraries, environment variables, and configuration files.
```

container: 一个image 的运行实例。

一个 image 可以起多个实例，打个比方，image 就类似于应用的安装包？

repository

A registry is a collection of repositories, and a repository is a collection of images。

Docker’s public registry 是免费且默认配置好的。

### 基本操作

为了与系统的 命令区分，所有的 docker 命令都以 docker 开头。

安装完成后：

查看安装版本：docker --version

查看更详细的 docker 安装信息： docker version

查看 docker 统计信息：docker info

运行一个最简单的容器：

docker run hello-world

docker 会首先在本地仓库寻找名叫 hello-world，版本为 latest 的镜像，如果本地找不到，会从 library下载 hello-world 镜像。

ls 展示所有信息；可以使用在 image, container 后.

eg:

​	docker image ls 显示所有 image

 	docker container ls 显示所有 container

prune -- 清除所有未使用的镜像或容器。

eg:

  docker image prune

  docker container prune

docker build --tag=gradle-test .   根据 Dockerfile 打镜像包，并添加tag。

docker run-p8080:80 image 将机器的8080 端口映射到 docker 容器的 80端口。

https://stackoverflow.com/questions/18497688/run-a-docker-image-as-a-container

每个 container 就相当于是一个 Linux 系统。

docker image inspect --查看 image 情况

docker inspect 容器名  -- 查看容器情况

进入命令行： docker exec -it 容器名  bash

容器内外文件拷贝

docker cp SRC DEST 

eg: docker cp container_name:path local_path



---

Dockerfile ： 前面我们说了，一个镜像包含了一个可运行应用到所有资源，这些资源如何定义呢？那就需要用到 Dockerfile 了。

```
FROM python:2.7-slim // 必须指定运行时环境作为后续步骤到基础镜像，：前为镜像名，后为版本号，docker 默认的规则是，如果版本号未指定，则默认取 latest 版本。
RUN echo "hello world!"
```



USER root

RUN mkdir tmp

COPY

WORKDIR