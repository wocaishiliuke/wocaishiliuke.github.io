---
title: AWS搭建ShadowSocks
date: 2018-05-05 19:01:09
categories:
    - Tools
tags:
    - ShadowSocks
    - AWS
    - Greatwall
---

本文将介绍AWS上ShadowSocks服务的搭建，以及本地客户端梯子的使用。

<!-- more -->

##### 目录
+ I.申请免费的AWS
+ II.AWS上搭建SS服务
+ III.本地浏览器使用

---

# I.申请免费的AWS

#### 1 [申请AWS账户](https://www.amazonaws.cn/)

> AWS提供免费1年的基础服务，流量每月15G，对于查资料来说，一般不会超。有东京节点，延迟低

- 1.注册

> 填写信息（一些敏感信息可以随意填，也可以填真实信息，使用英文）

- 2.付款信息

> 只支持信用卡，银联Visa等都可以。推荐某宝的虚拟aws EC2 信用卡，几块RMB（**最近虚拟卡好像都很难过验证，所以。。。**），当然也可以使用真实的信用卡

- 3.身份验证

> 国家选中国，填入手机号，立刻呼叫。页面弹出PIN码，接听验证电话，输入PIN码即可。等待电话挂掉和网页刷新，完成验证

- 4.选择支持方案

> 选择"基本（免费）"，继续完成AWS账号创建，邮箱会收到几封邮件

# II.AWS上搭建SS服务

#### 1.创建EC2实例

- 1.进入[AWS主页](https://www.amazonaws.cn/)，登录控制台
- 2.选择地区(如东京)。之前发送的邮件中，限定了可以创建主机的区域，如果在以外的区域创建会失败
- 3.服务菜单中选择EC2，启动实例，选择系统Ubuntu Server 16.04 LTS(HVM)（带有的免费标签的）

> 若之前填写的信用卡有误，可能会提示：确保之前的信息提供正确。进入我的账户，在付款方式中更改信用卡信息

- 4.选择符合条件的免费套餐，点击审核和启动
- 5.创建新密匙对，,密匙对名字随意（英文），这里取名"r3"，并下载密匙对r3.pem文件

> 妥善保存和备份该文件，是用于ssh登录该实例的秘钥信息

- 6.启动实例

> 服务器名=实例ID（i-xxx），可查看该服务器实例信息。其中公有DNS和公有IP，后面会用到

#### 2.连接服务器

> 本地如果是Windows，可用xshell或putty等ssh客户端，自行解决。这里本地使用Ubuntu连接AWS的Ubuntu，具体可参见[使用 SSH连接到Linux 实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)。简化步骤如下：

- 1.检查是否安装了SSH客户端（Ubuntu默认安装openssh client）。如果没有可参见[官网](http://www.openssh.com/)

```
ssh
```

- 2.修改秘钥r3.pem的权限

```
chmod 400 /path/my-key-pair.pem
```

- 3.链接实例，会用到本地的秘钥+服务器实例的公有DNS

```
ssh -i /path/r3.pem ubuntu@ec2-198-51-100-1.compute-1.amazonaws.com
```

> 出现ubuntu@ip-x-x-x-x:!$ 表示连接成功

#### 3.安装SS服务

- 1.将当前登录用户提升至root权限

```
sudo -s
```

- 2.更新软件列表

```
apt-get update
```

- 3.安装python-pip，后面要用pip命令安装Python版的SS服务端

```
apt-get install python-pip
```

- 4.安装shadowsocks

```
pip install shadowsocks
```

> 可能会报locale.Error: unsupported locale setting，设置LC_ALL后，再进行安装

```
$ locale

# 去除所有本地化的设置，让命令能正确执行
$ export LC_ALL=C

$ locale
```

#### 4.配置SS服务

```
# 创建配置文件
vi /etc/shadowsocks.json
```

> 单用户（只允许一个账户使用）

```
{
  "server": "0.0.0.0",
  "server_port": 8989,
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "password": "yourpassword",
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": false
}
```

> 多用户（开设多个账户）

```
{
  "server": "0.0.0.0",
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "port_password": {
    "8989": "password0",
    "9001": "password1",
    "9002": "password2",
    "9003": "password3",
    "9004": "password4"
  },
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": false
}
```

> 修改server_port和password，选择服务器的空闲端口即可。local_port、local_address和加密方式等不必修改

#### 5.启动SS服务

```
ssserver -c /etc/shadowsocks.json -d start
```

> 开机启动设置（优化）：修改rc.local文件，将上述start命令添加到第二行

```
vi /etc/rc.local
```

#### 6.暴露服务器端口

> 在AWS控制台，找到运行的实例

- 1.点击实例的安全组"launch-wizard-1"，进入实例的安全组设置。点击"入站"，进行编辑
- 2."添加规则"，其中类型和协议不需要改动，端口范围填写配置中设置的SS服务端口，来源选择任何位置
- 3.如果之前配置的多用户，这里需要把所有分配的端口都分别添加上，保存退出

> 至此，完成SS服务搭建

# III.本地浏览器使用

> 参考[Ubuntu环境搭建](https://wocaishiliuke.github.io/linux/2018/06/30/Ubuntu01/)

# 参考：

- [使用SSH连接到Linux实例](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)