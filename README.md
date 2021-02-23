[toc]
# 硬件资源需求
## 硬件配置
业务名称 | 资源配置 | 存储(/opt)大小 | 说明 
:-:|:--:|:-: | :-: |:--:|:---:
nginx  | 1核1G |20G | 业务网关,统一入口| 10.10.1.10|80
dnsmasq | 1核1G |20G | 域名解析服务|10.10.1.10|10080
openladp | 1核1G |20G | 提供统一账户管理|10.10.1.10|11080
wiki | 4核8G |40G | 团队成员的协作|10.10.1.11|8090
jira | 4核8G |40G | 团队事务跟踪|10.10.1.12|8080
jenkins | 4核8G |40G | CI/CD|10.10.1.13|8080
gitlab | 4核8G |40G |代码仓库 |10.10.1.14|10080
harbor | 2核4G |200G |镜像仓库 |10.10.1.15|10080
nexus | 2核4G |40G |产品仓库|10.10.1.16|10080
proxmox| -|-|虚拟化平台 | 10.10.1.22|8006
KVM| -|-|服务器控制台 | 10.10.1.21
## 网络配置
业务名称 |  域名 | 说明 | IP |端口
:-:|:--:|:--: |:--:|:---:
nginx  | - | 业务网关,统一入口| 10.10.1.10|80
dnsmasq | dns.lizhi.co | 域名解析服务|10.10.1.10|10080
openladp | ldap.lizhi.co | 提供统一账户管理|10.10.1.10|11080
wiki | wiki.lizhi.co | 团队成员的协作|10.10.1.11|8090
jira |jira.lizhi.co | 团队事务跟踪|10.10.1.12|8080
jenkins | jenkins.lizhi.co  | CI/CD|10.10.1.13|8080
gitlab |gitlab.lizhi.co  |代码仓库 |10.10.1.14|10080
harbor |regitstry.lizhi.co  |镜像仓库 |10.10.1.15|10080
nexus | nexus.lizhi.co  |产品仓库|10.10.1.16|10080
proxmox|-|虚拟化平台 | 10.10.1.22|8006
KVM|-|服务器控制台 | 10.10.1.21

    openladp/dnsmasq/nginx 资源消耗不大,部署在一台机器上
    jira、jenkins、gitlab为核心业务,生产中需要确保足够的硬件资源
    测试环境系统版本为CentOS 7.9
    数据盘均挂载于 /OPT 目录下

# 整体架构

```
graph LR
A(Client)
B(nginx/gateway)
C(jira)
D(confluence)
E(gitlab)
F(jenkins)
G(harbor)
H(nexus)
I(DNS)
J(Openldap)
K(PHPldapadmin)
CC(Postgres)
A-->B
B-->A
B---C
B---D
B---E
B---F
B---G
B---H
B---K
subgraph registry
G
end
subgraph LDAP
K---J
end
subgraph JiRa
C-->CC
end
subgraph WiKi
D-->DD(Postgres)
end
subgraph GitLab
E-->EE(Redis)
EE-->EEE(Postgres)
end
```
# 安装过程
## 基础平台搭建
基础平台由虚拟化平台提供,本次采用proxmox实现,也可以选择自己熟悉的虚拟化平台.
![image](78494A3000DD453F97F8DA8C98FE3C1E)
相对来讲proxmox实现提供和esxi相差不多的功能.但是资源消耗更少.公司测试平台IO性能较弱,实际使用发现exsi完全无法使用.proxmox却能流畅运行
### 准备工作
#### 技能需求

类型 | 能力要求
:-:|---
服务器 | 服务器安装调试、阵列卡的使用
虚拟化 | 任意一种虚拟化平台的安装调试、镜像制作、虚拟机备份与恢复
网络 | 了解ip、网关、dns工作原理,能自己规划网络
存储 | 了解文件类型,磁盘的分配和挂载
操作系统(linux) | linux的基本操作、磁盘的挂载、iptables、yum的使用、shell脚本的简单使用
容器 | docker安装、使用、基本原理、dockerfile的简单编写
容器编排 | docker-compose的基本使用方式、docker-compose文件的简单编写
代码仓库 | git工具的安装与简单使用、gitlab的安装与使用
jenkins | 了解CI/CD的流程、了解从编写到发布的整个开发过程、了解java编译环境
nginx | 了解反向代理机制
待补充...|...


#### 下载
##### proxmox
下载地址:   [Proxmox](https://www.proxmox.com/en/downloads)  
国外网站下载较慢,也可以选择右侧种子下载
![image](AB55E36CA9E3480BAF1186285B51156B)
##### 操作系统
下载地址:  [Centos7.9](http://mirrors.aliyun.com/centos/7.8.2003/isos/x86_64/CentOS-7-x86_64-DVD-2003.iso)

来源为阿里源Centos镜像
#### 登录工具
xshell 与 chrome ,自行搜索下载
#### 文档准备
由于系统内账号密码较多,建议单独开辟一个文档保存账号密码,以免忘记
#### 网络权限的索取
获取管理员权限或者管理员联系方式,后续需要修改客户端dns为本系统内的dns服务器ip

### 基础平台安装
#### 服务器
服务器阵列卡配置方法自行搜索
需要注意的是用于安装虚拟化平台系统的磁盘最好保留 300G 空间
,且需要为备份文件专门划分一块较大且可靠的磁盘
#### 虚拟化平台
虚拟化安装过程自行搜索
 
需要主要以下几点:

1. 系统盘安装在单独的一块磁盘上.
2. 系统的DNS与网关需要正确配置,便于系统的apt更新
2. 安装完成后需要单独挂载存储,与exsi不同.pve的存储需要指定存放文件类型,否则保存iso或者备份文件时候会发现仅有系统盘可以使用![image](8F42135CA0B043698C713234AABA10B5)
3. iso镜像文件与备份默认保存在"/"根路径下,如果系统分区较小请第一时间挂载iso镜像文件与备份的专用路径.避免系统分区被撑爆

#### 虚拟机系统安装与模板制作
Centos 7.9 安装过程自行搜索

需要注意以下几点:
1. 系统盘20G即可,便于拷贝与备份,业务数据保存至单独数据盘
2. 数据盘挂载至 /OPT 目录
3. 关闭selinux,防火墙(测试环境下)


##### 模板制作策略
建议保留3份虚拟机模板

1. 第一份虚拟机模板是干净的,仅做好数据盘挂载、换好国内yum源；
1. 第二份虚拟机模板在第一份的基础上需要安装常用的包； 
```
yum install -y bash-completion cryptsetup device-mapper-persistent-data epel-release git htop ipvsadm iscsi-initiator-utils libicu lsof lvm2 net-tools nfs-utils nmap selinux-policy systemd sysstat tar tcpdump telnet tree unzip vim yum-utils zip zsh
```
1. 第三份虚拟机模板在第二份的基础上安装docker与docker-compose,本次所有机器均基于模板三来实现

挂载、安装docker、docker-compose的方法自行搜索.

docker版本至少为17.03,docker-compose版本最新即可,两者版本建议都为当前第二新

实验环境下docker为19.03 docker-compse为1.21.2

##### 模板制作方式
以第一份虚拟机模板为例,其余模板的思路与此类似,即安装需求软件后清除手动与自动(sys-unconfig)清除个性化信息

安装完后进系统里自行配制好网卡(onboot=yes,清掉uuid)和dns
关闭selinux和Firewalld以及NetworkManager

```
systemctl disable firewalld NetworkManager
systemctl stop firewalld NetworkManager
setenforce 0
sed -ri '/^[^#]*SELINUX=/s#=.+$#=disabled#' /etc/selinux/config
```

禁用默认zeroconf路线(当系统无法连接DHCP server的时候，就会尝试通过ZEROCONF来获取IP,并添加一条169.254.0.0/16的路由条目)

```
echo "NOZEROCONF=yes" >> /etc/sysconfig/network
```

关闭sshd的dns服务,防止ssh等待解析地址的超时错误,相应缓慢

```
sed -ri '/UseDNS/{s@#@@;s@\s+.+@ no@}' /etc/ssh/sshd_config
systemctl restart sshd
```
清除个性化信息并关机,此命令会关闭操作系统,操作之前确定配置修改完毕

```
sys-unconfig
```

将关闭的centos制作为模板

![image](EBCDF730F48D419BBD0D744BF12458C3)

#### 虚拟机部署
基于已经规划好的节点信息,在模板三的拷贝上部署所有节点,配置好网络,挂载好数据盘

确保所有节点虚拟机部署完成
||||
:--:|:-:|:-:|
ldap |jira|jenkins
nginx |wiki|harbor
dns |gitlab|nexus



## 研发平台搭建
研发平台主要分为两部分:服务平台与业务平台

nginx、ldap、dns为服务平台,为业务平台提供入口、认证信息、域名解析服务

jira、wiki、gitlab、jenkins、harbor、nexus为生产业务,满足开发人员业务需求
