## 版本信息

Ambari：2.7.5

HDP：3.1.5

HUE：4.11.0

## 环境准备

1.hue的master节点上执行，为编译环境做准备

```shell
yum install sqlite-devel  libxslt-devel.x86_64 python-devel openldap-devel asciidoc cyrus-sasl-gssapi  libxml2-devel.x86_64 mysql-devel gcc gcc-c++ kernel-devel openssl-devel gmp-devel libffi-devel install npm
```

安装epel源：
wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo


2.所有机器上创建用户和组

```shell
groupadd hue
useradd -g hue hue
```

3.提前在mysql创建好hue的库并授权

```sql
CREATE DATABASE hue;
GRANT ALL PRIVILEGES ON hue.* TO hue@'%' IDENTIFIED BY '123456';
FLUSH PRIVILEGES;
```

4.提前建好hue在hdfs的HOME目录

```shell
hadoop fs -mkdir /user/hue
hadoop fs -chown hue:hue /user/hue
```

5.下载插件源码

在ambari server节点执行

```shell
VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
rm -rf /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE  
sudo git clone https://github.com/Gqyanxin/ambari-hue-service.git /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE
```

6.hue的安装包并放到你的Apache服务器上




在ambari server节点执行

## 代码修改

1. package/files/configs.sh文件

   ```
   USERID='ambari的管理员账号'
   PASSWD='ambari的管理员密码'
   ```

2. package/scripts/params.py文件

   第32行 download_url 改成你自己的地址，可以跟hdp的本地仓库放一起

   第40行 ambari_server_hostname 改成你自己的地址

3. 修改ignore_groupsusers_create的值
   使用命令行配置的模式进行查询和修改,注意地址需直接使用ip，否则可能访问不到
   
   ```
   # cd /var/lib/ambari-server/resources/scripts
   # python configs.py -u admin -p admin -n $cluster_name -l $ambari_server -t 8080 -a get -c cluster-env |grep -i ignore_groupsusers_create
   "ignore_groupsusers_create": "false",
   # python configs.py -u admin -p admin -n $cluster_name -l $ambari_server -t 8080 -a set -c cluster-env -k ignore_groupsusers_create -v true
   ```
   $cluster_name $ambari_server 替换成真实值
   
4. 安装simplyjson
   需要先离线安装setuptools和pip，然后用pip install simplyjson安装
   修改metainfo.xml文件，将simplejson这一类安装失败的文件单独手动安装之后，从xml里面去掉，主要都是python-xxx之类的包，部分包手动安装的时候要选择合适的版本，否则有可能安装失败。
   <img width="565" alt="image" src="https://github.com/bluesky4485/ambari-hue-service/assets/442591/b1e1af8d-5d25-48cd-9bb3-962e84367c3f">

5. 修改sudo.py文件，解决编码问题
 vim /usr/lib/ambari-agent/lib/resource_management/core/sudo.py
<img width="367" alt="image" src="https://github.com/bluesky4485/ambari-hue-service/assets/442591/ecfb17f1-aef9-42cc-9f19-c0d18bfc1d21">



## 部署安装

重启ambari

```shell
ambari-server restart
```

ambari界面操作

界面左侧 >> services >> Add service >> Hue >> NEXT >> 选择Hue Server >> NEXT >> 配置 

数据库配置，这里选了mysql：

![1583732152511](https://github.com/lijufeng2016/ambari-hue-service/blob/master/screenshots/1583732152511.png)

![1583732214223](https://github.com/lijufeng2016/ambari-hue-service/blob/master/screenshots/1583732214223.png)



## 编译

```shell
cd /usr/hdp/3.1.5.0-152/hue/
make apps
```

**注意：在准备工作的第一步的包必须安装才能编译成功**



打开主页面，输入hue hue

![1583737992211](https://github.com/lijufeng2016/ambari-hue-service/blob/master/screenshots/1583737992211.png)
