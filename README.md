[![GitHub stars](https://img.shields.io/github/stars/khs1994-docker/ci.svg?style=social&label=Stars)](https://github.com/khs1994-docker/ci)

# 如何工作

本地制作 Docker 镜像推送到私有仓库（Docker Registry）

本地开发，推送到 Gogs

Drone 拉取代码完成编译，部署

# 示例

* [Hexo](https://github.com/khs1994-docker/drone-demo-hexo)

* [Gitbook](https://github.com/khs1994-docker/drone-demo-gitbook)

# 初始化

首次使用本项目时，务必执行一次 `./ci.sh` 完成初始化。

## 编辑 `.env` 文件

务必编辑 `.env` 文件，具体变量含义文件中的注释有详细的说明。

# `80` `443` 端口是否占用

本项目使用 `nginx` 来反向代理各种服务，各服务通过不同的网址来暴露。

## 已占用

>这里又分两种情况，一种是外部的 `nginx` 容器占用 80 端口，一种是外部非容器方式运行的 `nginx` 占用 80 端口。

### 非容器方式运行的 nginx

使用时将 `conf.d` 中的 `*.conf` `ssl` `auth` 文件复制到本机 nginx 配置文件夹（一般情况下为 `/etc/nginx/conf.d`）。

务必修改 `/etc/hosts/`。

```bash
127.0.0.1 registry-server drone-server gogs-server
```

### 容器方式运行的 nginx

使用时将 `conf.d` 中的 `*.conf` `ssl` `auth` 文件复制到其他的文件夹中。

将三个 `*.conf` 中的 `registry-server` `drone-server` `gogs-server` 字段替换为本机路由器分配的地址。

之后将配置文件挂载到已存在的 nginx 容器中。

## 未占用

那么就请使用本项目的 nginx，请在 `docker-compose.override.yml` 文件中将 nginx 部分的注释取消。

# 开发环境

Gogs + Drone + Docker Registry

> 0.8 版本的 Drone 在使用 coding.net 时出现问题，已经反馈给两方。不知道是我配置问题还是什么，暂时放弃测试。

## 准备 SSL 证书

* git.t.khs1994.com

* drone.t.khs1994.com

* docker.t.khs1994.com

## Gogs

修改 `./gogs/app.prod.ini` 数据库密码、邮箱服务器密码等

## Docker Registry

请查看 [官方文档](https://docs.docker.com/registry/)。

## 启动

```bash
$ ./ci.sh

# 根据提示执行后续命令
```

# 生产环境

GitHub + Drone + 腾讯云容器服务镜像仓库(Tencent Hub)

由于本人腾讯云服务器配置较低，能用外部服务尽量用外部服务，可能在未来配置较高时自己搭建私有服务。

```bash
$ ./ci.sh production

# 根据提示执行后续命令
```

# Drone

参考 [官方文档](http://docs.drone.io/)，并仔细查看 `docker-compose.yml` 文件。

## 封闭注册

请在 `docker-compose.github.yml` 中将 `- DRONE_ADMIN=khs1994,khs1994-merge-robot` 替换为你的 GitHub 用户名。

关于用户管理请查看 [官方文档](http://docs.drone.io/zh/user-registration/)

# 本地环境注意事项

Drone 使用新的容器来拉取 git 代码（Drone 启动的 git 容器不能配置 host），这就要求 `git 服务器` 必须是「公网地址」（即公共 DNS 能够解析该 `git 网址`），当然也可以在 docker 的 daemon.json 文件中定义私有 `DNS 服务器`，来将 `git 服务器` 指向私有地址。

遇到这个问题的人看一下就明白了，这里不再赘述。

我的解决办法是在 DNS 解析商将 `Gogs` 的网址（ 例如 `git.t.khs1994.com` ） 解析到路由器分配给电脑的 IP。

# Swarm mode

```bash
$ docker-machine create \
      -d virtualbox \
      --engine-opt dns=114.114.114.114 \
      --engine-registry-mirror https://registry.docker-cn.com \
      --virtualbox-memory 2048 \
      --virtualbox-cpu-count 2 \
      ci

$ docker-machine ssh ci

$ git clone -b master --depth=1 https://github.com/khs1994-docker/ci.git

$ cd ci

$ ./ci.sh swarm
```

# More Information

* [Gogs](https://github.com/gogits/gogs)

* [Gogs Docker](https://github.com/gogits/gogs/tree/master/docker)

* [Drone](https://github.com/drone)

* [Drone Documents](http://docs.drone.io/)

* [Drone Docker](https://store.docker.com/profiles/drone)
