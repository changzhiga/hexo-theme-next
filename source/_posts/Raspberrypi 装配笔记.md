---
title: Raspberrypi 装配笔记
date: 2018-08-13 02:07:40
tags: raspberrypi
categories: Tech
description: Raspberrypi Configuration
---

* 1 镜像烧制
* 2 基础配置
	* 2.1 SSH 连接
	* 2.2 修改管理员密码
	* 2.3 Samba
* 3 功能配置
	* 3.1 Homebridge

## 1 镜像烧制

1. 从树莓派官网下载最新的 Raspbian 系统镜像，通过 Etcher 烧录进 TF 卡中；如果不使用最新的 Debian 系统，可以烧录硬盘备份的 Debian Jessie 镜像。

2. 烧制完成后，进入系统根目录 `boot`，新建无后缀的空脚本文件 `ssh`，再新建名为 `wpa_supplicant.conf` 的无线网络配置文件，其内容如下：

    ```
    country=CN
    ctrl_interface=DIR=/var/run/wpa_supplicant
    GROUP=netdev
    update_config=1
    network={
        ssid="[[Wi-Fi 名称]]"
        psk="[[Wi-Fi 密码]]"
        key_mgmt=WPA-PSK
        priority=1
    }
    ```

3. 完成以上步骤之后，即可拔出 TF 卡，将其插入树莓派中，接通电源，等待其自动连接上局域网。

## 2 基础配置

### 2.1 SSH 连接

在终端输入 `ssh pi@192.168.X.X`，初始密码为 `raspberry`。亦可参照 VPS 配置 SSH 的做法生成公钥和私钥来创建快捷短语。
    
### 2.2 修改管理员密码

```shell
passwd pi
```
    
### 2.3 Samba

修改系统软件源为阿里云：

```
sudo nano /etc/apt/sources.list
```

将 deb 后的 URL 修改为 http://mirrors.aliyun.com/raspbian/raspbian/。
       
* 刷新软件列表：

```
sudo apt-get update
```
        
* 安装 Samba 及其依赖：

```shell
sudo apt-get install samba samba-common-bin
```
        
* 修改 Samba 的配置文件：

```shell
sudo nano /etc/samba/smb.conf
```

在配置文件的最后加上：

```
[pi]

    path = /home/pi/

    valid users = pi

    browseable = Yes

    writeable = Yes

    writelist = pi

    create mask = 0777

    directory mask = 0777
```

保存，重新运行 Samba 服务：
        
```shell
sudo /etc/init.d/samba restart
```

* 添加 pi 用户为 Samba 用户：

```shell
sudo smbpasswd -a pi
```

## 3 功能配置

### 3.1 Homebridge

项目地址：
* [Homebridge](https://github.com/nfarina/homebridge)

* [Homebridge-Mi-Aqara](https://github.com/YinHangCode/homebridge-mi-aqara)

* [Homebridge-Yeelight](https://github.com/vvpossible/homebridge_yeelight)

* [Homebridge-Mi-Philips-Light](https://github.com/YinHangCode/homebridge-mi-philips-light)

1. 安装 NodeJS：

    ```shell
    curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash - 
    sudo apt-get install -y nodejs
    ```

2. 安装 Avahi：

    ```shell
    sudo apt-get install libavahi-compat-libdnssd-dev
    ```
        
3. 安装 Homebridge 本体和插件：

    ```shell
    sudo npm install -g --unsafe-perm homebridge # Stretch 需要加入 --unsafe-perm 参数   
    sudo npm install -g --unsafe-perm homebridge-mi-aqara # Aqara 平台插件，用于网关及其连接件      
    sudo npm install -g --unsafe-perm homebridge-yeelight # Yeelight 灯光系统插件
    sudo npm install -g --unsafe-perm homebridge-mi-philips-light # Philips 灯光系统插件
    ```

    安装完成后运行 `homebridge -D` 检查是否能正常运行。

4. 编辑配置文件：

    ```shell
    mkdir ~/.homebridge # 如果已经创建则跳过
    cd ~/.homebridge
    sudo nano config.json
    ```

    编辑配置文件如下：
    
    ```json
    {
        "bridge": {
            "name": "HomeBridge",
            "username": "B8:27:EB:EE:AF:1B",
            "port": 51888,
            "pin": "199-50-228"
        },
        "platforms": [{
            "platform": "MiAqaraPlatform",
            "gateways": {
                "34ce009525c9": "B51730D119334789"
            },
            "defaultValue": {
                "158d000227648c": {
                    "MotionSensor_MotionSensor": {
                        "name": "Motion Sensor"
                    }
                },
                "158d00019d3a9b": {
                    "Global": {
                        "disable": true 
                    },
                    "Button_StatelessProgrammableSwitch": {
                        "name": "Desktop Button",
                        "disable": false
                    },
                    "Button_Switch_VirtualSinglePress": {
                        "name": "Single Press"
                    },
                    "Button_Switch_VirtualDoublePress": {
                        "name": "Double Press"
                    }
                },
                "158d00019cb82c": {
                    "TemperatureAndHumiditySensor_TemperatureSensor": {
                        "name": "Temperature"
                    },
                    "TemperatureAndHumiditySensor_HumiditySensor": {
                        "name": "Humidity"
                    }
                }
            }
        },
            {
                "platform": "yeelight",
                "name": "Lightstrip"   
            },
            {
                "platform": "MiPhilipsLightPlatform",
                "deviceCfgs": [{
                "type": "MiPhilipsSmartBulb",
                "ip": "192.168.31.114",
                "token": "7b2c6803b67b99f9b7a250cd919858dc",
                "lightName": "Bulb",
                "lightDisable": false
            }]
            }
        ]
    }
    ```

    保存退出，试运行：`homebridge -D`

5. 使用 Screen 保持 Homebridge 在后台运行：

    ```shell
    sudo apt-get install screen
    ```

安装完后，运行 `screen -S homebdg`，在窗口里运行 Homebridge。