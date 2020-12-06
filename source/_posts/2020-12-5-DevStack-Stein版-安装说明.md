---
title: DevStack(Stein版)安装说明
date: 2020-12-05 14:55:19
tags: OpenStack
categories: 云计算
---

### 1. 修改apt源

```shell
# cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

```shell
# vim /etc/apt/sources.list
```
<!-- more -->

```shell
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

```shell
# apt-get update
# apt-get upgrade
```

### 2. 配置pip源与Python

（1）配置pip国内源

```shell
# mkdir ~/.pip
# vim ~/.pip/pip.conf

[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
extra-index-url = https://mirrors.163.com/pypi/simple/

timeout = 60
```

（2）配置pip默认为pip3，Python默认为Python3

```shell
# ln -s /usr/bin/pip2 /usr/bin/pip2.7
# ln -s /usr/bin/pip3 /usr/bin/pip3.6
# rm -f /usr/bin/pip
# ln -s /usr/bin/pip3 /usr/bin/pip
# rm -f /usr/bin/python
# ln -s /usr/bin/python3 /usr/bin/python
```

### 3. 准备devstack安装

#### （1）clone devstack代码（Stein版）

```shell
# git clone http://git.trystack.cn/openstack-dev/devstack -b stable/stein
```

#### （2）创建stack账户

```shell
# devstack/tools/create-stack-user.sh
```

#### （3）移动代码（同后续安装目标目录放在一起）

```shell
# mv devstack /opt/stack
# chown -R stack:stack /opt/stack/devstack
```

#### （4）切换到 stack 用户

```shell
# su - stack
# cd devstack
```

#### （5）对于stack账户的pip，再次执行步骤2

```shell
# mkdir ~/.pip
# vim ~/.pip/pip.conf

[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
extra-index-url = https://mirrors.163.com/pypi/simple/

timeout = 60
```


### 4. 控制节点安装

#### 4. 1 安装控制节点，编辑local.conf 

```shell
stack@controller:~/devstack$ vim local.conf
```

控制节点（ip与网卡名根据实际设置）

```shell
[[local|localrc]]
#---------------common section even some node role may not use this setting

# use TryStack git mirror

GIT_BASE=http://git.trystack.cn
NOVNC_REPO=http://git.trystack.cn/kanaka/noVNC.git
SPICE_REPO=http://git.trystack.cn/git/spice/spice-html5.git
#LIBVIRT_TYPE=kvm

DEST=/opt/stack
LOGFILE=$DEST/logs/stack.sh.log
VERBOSE=True
LOGDAYS=1
LOG_COLOR=True
RECLONE=false
PIP_UPGRADE=Flase
DOWNLOAD_DEFAULT_IMAGES=False
IMAGE_URLS="http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img"
IP_VERSION=4
SERVICE_IP_VERSION=4
ENABLE_IDENTITY_V2=False

DATABASE_TYPE=mysql

SERVICE_HOST=控制节点的ip地址
MYSQL_HOST=$SERVICE_HOST
RABBIT_HOST=$SERVICE_HOST
GLANCE_HOSTPORT=$SERVICE_HOST:9292
ADMIN_PASSWORD=admin
MYSQL_PASSWORD=admin
RABBIT_PASSWORD=admin
SERVICE_PASSWORD=admin

# Neutron options

NEUTRON_CREATE_INITIAL_NETWORKS=False

MULTI_HOST=1

USE_PYTHON3=True

enable_service placement-api 
enable_service placement-client
disable_service etcd3 

#---------------node special section
HOST_IP=控制节点的ip地址
FLAT_INTERFACE=enp5s0（网卡名）
#RECLONE=True
#enable_plugin octavia https://opendev.org/openstack/octavia stable/stein
#enable_plugin octavia http://git.trystack.cn/openstack/octavia stable/stein

#disable_service n-cpu q-agt n-api-meta c-vol n-net
disable_service n-cpu c-vol
enable_service q-fwaas,q-vpn
enable_service q-lbaasv2,octavia,o-cw,o-hk,o-hm,o-api
```



#### 4. 2 控制节点上，修改脚本

##### （1）修改tools/install_pip.sh

```shell
# vim /opt/stack/devstack/tools/install_pip.sh
```

注释掉135-140行

##### （2）修改lib/apache

```shell
# vim /opt/stack/devstack/lib/apache
```

第98行改为  

```shell
uwsgi=$(ls uWSGI*)
```

第99行改为   

```shell
# mkdir uwsgi & tar xvf $uwsgi -C uwsgi
```

##### （3）修改lib/tempest

```shell
# vim /opt/stack/devstack/lib/tempest
```

第110行改为python3的写法：

```shell
echo $size | python -c "import math; print(int(math.ceil(float(int(input()) / 1024.0 ** 3))))"
```

#### 4. 3 控制节点上，执行denstack安装脚本

```
# FORCE=yes ./stack.sh
```

#### 4. 4、控制节点上，报错后的一些操作

   每次报错后都要先执行 ./unstack.sh
   再执行步骤4-3

##### （1）如果os-testr===1.0.0安装卡住，则可以人工安装os-testr===2.0.0

```
#pip install os-testr==2.0.0
```

若无法解决，就直接改upper-constraints.txt文件

```
# vim /opt/stack/requirements/upper-constraints.txt
```

第438行改为   os-testr===2.0.0

##### （2）nova.py文件报错，novaclient.v2库中没有list_extensions函数，

修改horizon/openstack_dashboard/api/nova.py

```shell
# rm /opt/stack/horizon/openstack_dashboard/api/nova.py
```

将上述nova.py文件替换为下面网址里的文件
https://opendev.org/openstack/horizon/src/commit/b148c9207580731863e4dc49771e132a8c31edbc/openstack_dashboard/api/nova.py
也可以用wget直接下载，下载链接如下：
https://opendev.org/openstack/horizon/raw/commit/b148c9207580731863e4dc49771e132a8c31edbc/openstack_dashboard/api/nova.py

##### （3）oslo_privsep.daemon.FailedToDropPrivileges: privsep helper command exited non-zero 

修改/usr/local/bin/privsep-helper

```
# vim /usr/local/bin/privsep-helper
```

sys.exit(helper_main())   改为 sys.exit(0)

##### （4）如果cirros-0.4.0-x86_64-disk.img下载卡住，可以浏览器下载，再放入对应目录

下载链接复制到浏览器   http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
把下载好的镜像文件传到服务器，再放入对应目录

```
# mv cirros-0.4.0-x86_64-disk.img /opt/stack/devstack/files/
```

##### （5）若git某个模块卡住，则可以手工git clone

```
# git clone http://git.trystack.cn/openstack/neutron /opt/stack/neutron -b stable/stein
```



#### 4. 5 控制节点安装结束后的输出


执行 # ./run_tests.sh 进行各项服务的测试

执行 # openstack user list ，若出现 Missing value auth-url required for auth plugin password
则执行 # source openrc admin admin

#### 4. 6 控制节点安装结束后登录OpenStack web界面，账号密码均为 admin

   http://172.16.62.229/dashboard/auth/login/

若侧边栏：管理员/计算/实例 报错，则检查nova日志

```
# journalctl -f --unit devstack@n-api.service
```

报错：oslo.messaging._drivers.impl_rabbit [-] AMQP server on 172.16.62.229 is unreachable: [Errno 104] Connection reset by peer.
rabbitmq添加用户

```shell
# rabbitmqctl add_user admin admin
# rabbitmqctl set_permissions -p / admin "." "." ".*" ; 
# rabbitmqctl set_user_tags admin administrator;
# service rabbitmq-server restart
# rabbitmqctl list_users
```

### 5. 计算节点安装

#### 5.1 安装计算节点，编辑local.conf 

```shell
# stack@computer1:~/devstack$ vim local.conf
```

计算节点（ip与网卡名根据实际设置）

```shell
[[local|localrc]]
#---------------common section even some node role may not use this setting

# use TryStack git mirror

GIT_BASE=http://git.trystack.cn
NOVNC_REPO=http://git.trystack.cn/kanaka/noVNC.git
SPICE_REPO=http://git.trystack.cn/git/spice/spice-html5.git
#LIBVIRT_TYPE=kvm	

DEST=/opt/stack
LOGFILE=$DEST/logs/stack.sh.log
VERBOSE=True
LOGDAYS=1
LOG_COLOR=True
RECLONE=false
PIP_UPGRADE=Flase
DOWNLOAD_DEFAULT_IMAGES=False
IMAGE_URLS="http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img"
IP_VERSION=4
SERVICE_IP_VERSION=4
ENABLE_IDENTITY_V2=False

DATABASE_TYPE=mysql

SERVICE_HOST=控制结点的ip地址
MYSQL_HOST=$SERVICE_HOST
RABBIT_HOST=$SERVICE_HOST
GLANCE_HOSTPORT=$SERVICE_HOST:9292
ADMIN_PASSWORD=admin
MYSQL_PASSWORD=admin
RABBIT_PASSWORD=admin
SERVICE_PASSWORD=admin

# Neutron options

NEUTRON_CREATE_INITIAL_NETWORKS=False

MULTI_HOST=1

USE_PYTHON3=True

#---------------compute node common section
ENABLED_SERVICES=n-cpu,q-agt,n-api-meta,placement-client,n-novnc

NOVA_VNC_ENABLED=True
NOVNCPROXY_URL="http://$SERVICE_HOST:6080/vnc_auto.html"

#---------------compute node special section
HOST_IP=计算结点ip地址
FLAT_INTERFACE=enp5s0（本机网卡名）
VNCSERVER_PROXYCLIENT_ADDRESS=$HOST_IP
VNCSERVER_LISTEN=$HOST_IP
#ENABLED_SERVICES+=,c-vol
```



#### 5. 2、计算节点上，修改脚本，同4-2

#### 5. 3、计算节点上，执行denstack安装脚本

```shell
# FORCE=yes ./stack.sh
```

#### 5. 4、计算节点上，报错后的一些操作

   每次报错后都要先执行 ./unstack.sh
   再执行步骤5-3
   具体操作同4-4，一般只会遇到4-4-(1)、(3)的报错，git clone和安装时的网络环境有关

#### 5. 5、计算节点安装结束后的输出

执行 # ./run_tests.sh 进行各项服务的测试

执行 # openstack user list ，若出现 Missing value auth-url required for auth plugin password
则执行 # source openrc admin admin

#### 5. 6、让计算节点注册，在控制节点上运行

```shell
stack@controller:~/devstack$ /opt/stack/devstack/tools/discover_hosts.sh
```

#### 5. 7、计算节点安装结束后登录OpenStack web界面，账号密码均为 admin格式为

http://控制结点ip地址/dashboard/auth/login/

例如：http://172.16.62.229/dashboard/auth/login/

侧边栏：管理员/计算/虚拟机管理器，可以看到新增的计算节点
http://控制结点ip地址/dashboard/admin/hypervisors/

 