---
title: ttRSS 的搭建
date: 2018-08-14 02:01:52
tags: ttrss
categories: Tech
description: ttRSS 搭建指南
---

# 安装
- Docker CE：提供 Docker 环境
- PostgreSQL 数据库：存储 RSS 服务运行所需的记录；也可以使用 MySQL，但是前者的性能更好
- Tiny Tiny RSS

## 安装 Docker CE：

1. Ubuntu

    * 设置仓库
    
    	* 更新 `apt` package index：
    
            ```shell
            sudo apt-get update
            ```

    	* 安装依赖包以允许 `apt` 通过 HTTPS 使用仓库：
            ```shell
            sudo apt-get install \
            apt-transport-https \
            ca-certificates \
            curl \
            software-properties-common
            ```
    	* 添加 Docker 官方 GPG key：
            ```shell
            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
            ```
    	* 验证 GPG key 已安装：
            ```shell
            sudo apt-key fingerprint 0EBFCD88
            ```
        显示以下内容说明安装成功：
        ```shell
        pub   4096R/0EBFCD88 2017-02-22
        Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
        uid                  Docker Release (CE deb) <docker@docker.com>
        sub   4096R/F273FCD8 2017-02-22
        ```
    * 安装 Docker

    	* 配置稳定版的仓库：

            ```shell
            sudo add-apt-repository \
            "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
            $(lsb_release -cs) \
            stable"
            ```
    	* 安装 Docker CE
    		* 更新 `apt` package index：
                ```shell
                sudo apt-get update
                ```
    		* 安装：
                ```shell
                sudo apt-get install docker-ce
                ```
    		* 测试是否安装成功：
                ```shell
                sudo docker run hello-world
                ```
2. CentOS

    * 设置仓库

        * 安装 `yum-utils`：

            ```shell
            sudo yum install -y yum-utils
            ```
        * 建立稳定的仓库：

            ```shell
            sudo yum-config-manager \
            --add-repo \
            https://docs.docker.com/v1.13/engine/installation/linux/repo_files/centos/docker.repo
            ```
    * 安装 Docker

        * 更新 `yum package index`：

            ```shell
            sudo yum makecache fast
            ```
        * 安装最新版本的 Docker：

            ```shell
            sudo yum -y install docker-engine
            ```
        * 运行 Docker：

            ```shell
            sudo systemctl start docker
            ```
        * 测试运行状态：

            ```shell
            sudo docker run hello-world
            ```

## 安装 PostgreSQL 的 Docker 镜像，此镜像已预先配置完成以直接配合 RSS 镜像使用的数据库环境：

```shell
sudo docker pull nornagon/postgres \
sudo docker run -d --name ttrssdb nornagon/postgres
```

## 安装 Tiny Tiny RSS 本体的镜像：

```shell
sudo docker pull fischerman/docker-ttrss \
sudo docker run -d --link ttrssdb:db -p 8080:80 -e SELF_URL_PATH=http://[[IP_ADDRESS]]:8080 fischerman/docker-ttrss
```

其中:

- `-p 8080:80`：该参数表明，将该容器内应用的 `80` 端口（冒号后）映射到主机的 `80` 端口（冒号前）上；如果主机还需运行其他 80 端口的服务（如博客建站），则应将冒号前的值改为一个未被占用的端口；例如，`-p 8080:80` 将启用主机的 `8080` 端口。

- `-e SELF_URL_PATH=http://[[IP_ADDRESS]]:8080`：该参数表明，该 Tiny Tiny RSS 应用将可从 `http://[[IP_ADDRESS]]` 访问；如果在上一步保持了默认的端口设置，则直接将上述网址换为你主机的 IP 地址（或解析至该主机的域名）即可，否则，应在地址之后进一步表明使用的端口，如：`http://xxx.xxx.xxx.xxx:8080`。

# 用插件添加主题、全文输出和多客户端支持

## 安装主题（以仿 Feedly 主题为例）

下载主题文件：[tt-rss-feedly-theme-1.3.0](https://www.dropbox.com/s/88olyj3dsmmr23t/tt-rss-feedly-theme-1.3.0-for-ttrss-17.4.0.zip?dl=0)

解压缩主题文件：

```shell
sudo apt-get install unzip // 如果没有安装 unzip
```

```shell
unzip master.zip
```

拷贝文件：

```shell
docker ps # 查看当前 Docker 容器列表，找到 IMAGE 一列为 `[[fischerman/docker-ttrss]]` 的记录，复制其 `[[CONTAINER ID]]`
```

![15091878055188](/images/15091878055188.png)

```shell
sudo docker cp tt-rss-feedly-theme-master/feedly.css [[CONTAINER ID]]:/var/www/themes
```

```shell
sudo docker cp tt-rss-feedly-theme-master/feedly [[CONTAINER ID]]:/var/www/themes
```

## 配置 RSS 全文输出

注册 Mercury 并复制其 API 密钥，可以在 Web Parser 页面看到。

为 Tiny Tiny RSS 安装全文输出插件：

```git
git clone https://github.com/WangQiru/mercury_fulltext.git
```

```shell
sudo docker cp mercury_fulltext [[CONTAINER ID]]:/var/www/plugins
```

在 RSS 设置配置 mercury_fulltext 插件。

## 多客户端支持（模拟 Fever）

安装插件：

```git
git clone https://github.com/rannen/tinytinyrss-fever-plugin.git
```

```shell
sudo docker cp ~/tinytinyrss-fever-plugin/fever [[CONTAINER ID]]:/var/www/plugins
```

安装后，在 Tiny Tiny RSS 的设置页面启用名为 fever 的插件。保存后，再次进入设置时，Preference 选项卡下会多出一个名为 Fever Emulation 的板块。在该板块的文本框中，设置一个专用于该插件的密码，然后点击 Save。该文本框下方同时给出了一个独立的服务器地址，其格式类似于 `http://rss.example.org/plugins/fever/`； 将其记下以备后用。

这时，在 Reeder 等 app 中，只要在添加 RSS 账号时选择 Fever，在服务器地址栏填入上述地址，用户名填写 `admin`，密码是以上设定的密码，即可配合这些你惯用的 app 使用 Tiny Tiny RSS 了；文件夹、标为已读、星标等功能均被良好支持。