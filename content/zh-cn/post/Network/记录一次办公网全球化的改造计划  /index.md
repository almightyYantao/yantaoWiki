---
title: 记录一次办公网全球化的改造计划  
date: 2023-11-20  
tags: ['办公网','全球化','全球办公','国际化','Panabit','iwan','sd-wan']  
categories: ['Network']
---
# 背景
办公网每次去海外找资料都需要重新连接VPN，或者自己连接自己买的小飞机之类的才可以。但是这种在互联网公司内的话，非常的不友好；为公司工作还要自己花钱买小飞机～

之前尝试过下面这种方式：
新增一台海外的机器（新加坡、香港）搭建SS/v2ray/trojan之类的协议，然后办公网新增一个软路由去连接，通过ACL把部分IP的用户跳转过去；
但是这种方式自己家里用用还行，如果想要在企业的办公网来使用的话，人一多就不行，因为所有人都从这个IP出去了，而且每次都需要命令行去操作ACL增加用户，非常的麻烦；

# Panabit 的 iWAN
为了解决前面的这种方式，决定测试使用 Panabit 的 Iwan 的方式，也就是所谓的 Panabit 的 `SD-WAN`；
- 重连速度很快：  
        比L2TP的要快一个数量级，L2TP要重连，需要有几十次交互，而我们只需要一次即可  
- 客户端不受底层承载线路IP变化影响：  
        当底层承载线路（比如PPPOE拨号线路）的IP地址发生变化时，不会影响iWAN隧道，iWAN隧道不会中断，保证通信正常进行；  
        因为很多用户是通过PPPOE拨号线路出去的，PPPOE拨号线路重拨时一般会改变IP地址，如果用L2TP的话，那么这个L2TP会话就要重建；  
        而用iWAN的话，现有的会话可以照常使用，不需要做任何改变；  
- 传输效率高：  
        iWAN的包头很小，只有8个字节，而且在后续版本里，我们会压缩IP报文头，这样可以继续减少额外报文头的大小，所以能大幅度提升传输效率；  
        如果用国际线路的话，节省下来的流量费用都是很可观的；  
- 抗干扰：  
        不像L2TP，中间人可以直接发包TERMINATE，iWAN控制命令有完整性检查，可以避免中间人攻击。

# 部署
## 1、搭建panabit
记得服务器申请2H的，1H需要需改核心，非常麻烦
下载Linux系统文件：文末
上传文件到root根目录下
```bash
tar -xzf PanabitFREE_SUIr2p3_20220413_Linux3.tar.gz
cd PanabitFREE_SUIr2p3_20220413_Linux3
# 输入以下命令进行安装
./ipeinstall
```
修改`/etc/PG.conf`文件
因为是单网口，所以数据口和管理口都需要配置成eth0，后面不要加任何东西
```bash
DATA_PORTS 修改成：DATA_PORTS="eth0"
```
### 修改端口
#### 上传文件，修改配置
需要上传一个joskmc文件（文件在文末）
在`/etc/PG.conf`中新增一下一行
```bash
HTTPS_PORT=2194
```
#### 执行joskmc
```bash
/root/joskmc tcp 2194
```
#### 修改：`/etc/rc.local`，增加一下三行
```bash
sleep 10
/root/joskmc tcp 2194
```

## 2、进行隧道配置（海外）
### (1)、登录WEB页面，修改网卡方向
默认账号密码：admin/panabit
系统概况 → 网络接口：eth0，修改成对外，只有对外才可以创建WAN线路
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103308.png"/>
### (2)、创建WAN线路
应用路由 → 接口线路：
需要注意，Mac地址必须克隆
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/202302171033490.png"/>
### (3)、创建IWAN连接账号
对象管理 → 账号管理 → 组织架构：
地址范围：这一块可以自己定一个内网的IP段就可以，不要冲突就好
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103417.png"/>
地址范围需要把网关地址留出来！！！！！
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103432.png"/>
对象管理 → 账号管理 → 本地账号：
处理用户组需要选择前面创建的用户组，其他的根据实际情况填写
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/202304211018672.png"/>
### (4)、创建iWAN服务
应用路由 → iWAN服务 → 服务列表：
注意：服务器网关地址要和你前面设置的地址范围要在一块，并且需要排除这个地址的下发
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103517.png"/>
应用路由 → iWAN服务 → 服务映射：
根据配置情况选择即可
iWAN使用的是UDP连接，因此端口需要开放UDP
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103531.png"/>
### (5)、创建策略路由
应用路由 → 策略路由：需要添加一条回程的全程路由，要不然DNS牵引、FQ都会失败
这里选择iWAN的线路
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103546.png"/>
## 3、客户端配置（办公网）
### (1)、新建WAN线路
应用路由 → 接口线路 → WAN线路：按照信息提示填写即可完成iWAN线路配置
注意：必须有一个外网的网卡，并且最好把加密开起来
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103622.png"/>
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103634.png"/>
### (2)、设置DNS牵引
应用路由 → DNS管控：海外域名是一个域名群组，可以自己修改
主要解决DNS污染问题，要不然可能部分网站会无法访问
这里有一个很注意的点，就是你的DNS，访问DNS的链路必须经过PA，否则牵引不会生效
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103653.png"/>
### (3)、设置策略路由
应用路由 → 策略路由：我这边直接拿了飞连的609海外分流IP段进行分流，你也可以自己修改（文末下载）
主要为了只有需要海外的才出去，不能把所有的流量全部导出去
<img src="https://raw.githubusercontent.com/almightyYantao/blog-img/master/20230217103713.png"/>

# 附件
169IP段： https://www.123pan.com/s/cRk7Vv-frSsH 提取码:NzAF   
Linux操作系统： https://www.123pan.com/s/cRk7Vv-arSsH 提取码:A5MC   
joskmc： https://www.123pan.com/s/cRk7Vv-BrSsH 提取码:kTu9   
