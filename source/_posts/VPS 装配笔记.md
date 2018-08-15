---
title: VPS 装配笔记
date: 2018-08-13 23:31:58
tags: vps
categories: Tech
description: VPS Configuration
---

## 1. 前期

按照既定格式为 VPS 主机命名，约定格式为：

**`主机商/平台名三位缩略字母 - 主机所在国家/城市名两位缩略字母 + 数字编号`**

如 Google Cloud Platform 为 `gcp`，GigsgigsCloud 为 `ggs`，Vultr 为 `vtr`，以此类推；地名如 Taiwan 为 `tw`，Tokyo 为 `tk`。故有 `gcp-tw`、`ggs-hk`、`vtr-nj1` 等，主机名全部使用小写。

考虑到部分主机商的 CentOS 和 Debian 有大大小小的问题，故操作系统首选 Ubuntu 最新版本，保守选择 Ubuntu 16.04 LTS（Xenial Xerus）。

## 2. 关键信息预存

首先存储 IP 可以方便后续的使用。主机正常运行后，使用 LaunchBar 的 Snippets 存储 各个主机的外部 IP，以主机名的 **`首字母（重复则使用前两位） + 地名（数字编码）`** 共三或四位字母作为 Snippets 的快捷短语；如 `gtw`，`gghk`。

## 3. SSH 连接

1. 生成公钥和私钥，必要时将公钥复制到主机的 SSH 添加处（`~/.ssh/authorized_keys`）。

    ```shell
    ssh-keygen -t rsa -f ~/.ssh/[[主机名]] -C [[用户名]]
    ```

    `[[主机名]]` 按照前期命名填写，`[[用户名]]` 为 `changzhiga`。

2. 某些主机可能需要通过 `ssh-copy-id` 命令添加公钥：

    ```shell
    ssh-copy-id -i ~/.ssh/[[主机对应的公钥]] [[用户名]]@IP
    ```

    公钥文件后缀为 `.pub`，`[[用户名]]` 为 `changzhiga`。

3. 通过终端 SSH 登录 root 账户，使用 Auto Config 添加主机到 HyperApp。观察 HyperApp 的 Docker 状态是否异常，如异常则需考虑重新安装系统；一般情况下会显示无法找到 `docker` 命令，说明系统需要安装 Docker。同时在终端测试 `sudo` 和 `docker` 命令，进一步确认系统是否正常。

    ```shell
    ssh root@IP
    ```
        
4. 确认系统正常后，在 root 账户下添加自定义用户并为其赋予 root 权限：

    ```shell
    adduser [[用户名]] \
    usermod -aG sudo [[用户名]] \
    su - [[用户名]]
    ```

5. 设置免密码登录

    往`~/.ssh/config`配置文件添加 ssh 服务器信息即可实现别名登录而无需记住用户名和 IP 地址：

    ```shell
    vim ~/.ssh/config
    ```

    编辑如下：

    ```shell
    Host    alias   # 自定义别名
    HostName    hostname    # ssh 服务器的 IP 或域名
    Port    22  # ssh 服务器的端口，默认为22
    User    root    # ssh 服务器用户名
    IdentityFile    ~/.ssh/id_rsa   # 第一步生成的公钥文件对应的私钥文件
    ```

    最终，只需要在 Terminal 输入`ssh alias`即可。

6. 延长 SSH 会话超时时间

- 服务器端配置
	
	```shell
	vim /etc/ssh/sshd_config
	```

	找到以下配置（默认被注释掉，需要取消注释）：

    ```shell
	ClientAliveInterval 30
	ClientAliveCountMax 3
    ```

    **ClientAliveInterval**：SSH Server 与 Client 的心跳超时时间。当客户端没有指令过来，Server 间隔 ClientAliveInterval 的时间（单位秒）会发一个空包到 Client 来维持心跳，保证 Session 有效。   
   
    **ClientAliveCountMax**：当心跳包发送失败时重试的次数，比如设置成了3，如果 Server 向 Client 连续发三次心跳包都失败了，就会断开这个 Session 连接。  

	修改完后重启 ssh 以使之生效：
	
	```shell
	service ssh restart
	```

- 客户端配置

    比起修改服务器端的配置，修改客户端更加容易一些。
    
    ```shell
    ~/.ssh/config
    ```
    修改以下内容：  
    ```
    Host myhostshortcut
    HostName myhost.com
    User root
    ServerAliveInterval 30
    ServerAliveCountMax 3
    ```

## 4. 一系列初始化脚本

### 4.1 安装 Docker

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

### 4.2 使用 HyperApp 自动安装 BBR。

### 4.3 使用一键脚本安装

```shell
wget --no-check-certificate -O ssss-U.sh https://raw.githubusercontent.com/uselibrary/Update_VPS/master/ssss-U.sh && chmod +x ssss-U.sh && bash ssss-U.sh
```

## 5. 后期

1. 使用 HyperApp 安装各种所需的 App。

2. 在 `~/.ssh/config` 中添加 SSH 登录的快捷短语，同 Snippets 的规则一致。

3. Surge 中的节点命名规则为：

    **`国旗 + 大写主机名`**
    
    如 `🇼🇸 GCP-TW`、`🇭🇰 GGS-HK`。
