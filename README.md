## 前言

本文档主要介绍当前三通两平台所涉及的部署环境，并且以后其他系统也会基于当前部署环境，并且再此基础上进行优化，或者如果有更好的方案，也会代替它。本文档主要涉及以下几方面：

- 部署环境基础软件及安装
- Docker镜像文件描述
- 部署环境基本脚本

系统所有的运行环境都是放在docker容器中运行的，所以docker是部署环境核心，其他软件和脚本工具都是辅助docker容器。

## 部署环境基础软件及安装

### Docker

#### 介绍

点击 [传送门](http://yeasy.gitbooks.io/docker_practice/content/) 查看docker详情

#### 安装

- ubuntu 系统<br>
	sudo apt-get install apt-transport-https<br>
	sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9<br>
	sudo bash -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"<br>
	sudo apt-get update<br>
	sudo apt-get install lxc-docker<br>
	
- centos6 系统<br>
	sudo yum install http://mirrors.yun-idc.com/epel/6/i386/epel-release-6-8.noarch.rpm<br>
	sudo yum install docker-io
	
- centos7 系统<br>
	sudo yum install docker	
	
#### 配置

如果需要从私有仓库pull镜像，需要如下设置

- ubuntu 系统<br>
	vi /etc/default/docker<br>
	DOCKER_OPTS=" --insecure-registry reg.docker:5000 "
	
- centos 系统<br>
	/etc/sysconfig/docker<br>
	other_args="--insecure-registry=reg.docker:5000"
	
### Weave

#### 介绍

因为docker默认是不支持两台物理机之前容器间访问，所以需要用到第三方组件来完成容器之间的沟通，Weave就是提供这种功能，[传送门](https://github.com/weaveworks/weave)

### 安装

sudo wget -O /usr/local/bin/weave https://raw.githubusercontent.com/zettio/weave/master/weave
sudo chmod a+x /usr/local/bin/weave
sudo apt-get install conntrack（centos yum install conntrack-tools） 
sudo weave launch


## docker镜像


## eduos安装

### 安装dns服务（value-dns）

./dns.py create



### 安装etcd服务（value-etcd）

./etcd.py create 1

### 安装负载均衡服务（value-slb）

./slb.py create 1

如有需要修改 /etc/confd下面的配置文件

### 安装redis服务 (value-redis)

./redis.py create 1 

### 安装消息中间件（value-metaq）

./zookeeper.py create  #安装服务器发现 <br>

./metaq.py create 1  #安装metaq <br>
修改 /home/software/metaq-server/conf/server.ini,加入[topic=cw]

### 安装数据库(value-db)

- 创建容器，并且进入

	./db.py create 1 && ./db.py ssh 1

- 如果/home/mysql下面还没有数据库文件，则执行下面命令，解压数据库文件

	tar -zxvf /home/software/mysql.tar.gz -C /home
	
- 默认mysql没有密码，进入mysql，修改密码操作

	mysql -u root -p
	use mysql;
	UPDATE user SET Password=PASSWORD('root') where USER='root';
	FLUSH PRIVILEGES;
	
- 设置主服务器

	- 修改/etc/mysql/my.cnf，如下，并且重启数据库 <br>
	log-bin=Mysql-bin #启用二进制日志，并指定生成log文件名 <br>
	server-id=1  #指明server id，master的server id必须在1-253之间，且唯一 <br>
    binlog-do-db=edu_el  #需要同步的数据库，默认同步所有库 <br>
    binlog-ignore-db=mysql  #忽略同步的数据库，若有多个，如下写多行 <br>
    binlog-ignore-db=information-schema
    
    - 设置从库权限
    
    	GRANT REPLICATION SLAVE ON *.* TO edu@'%' IDENTIFIED BY 'edu';
    	    
    - 获取log位置<br>
    
    	show master status\G
    
    mysqldump -uroot -proot -hlocalhost edu_el  > edu_el.sql

- 设置从服务器

	- 修改/etc/mysql/my.cnf，如下，并且重启数据库 <br>
	innodb_flush_log_at_trx_commit=0 <br>
	server-id      = 11 #唯一<br>
	
	- 设置主库log位置
	
		CHANGE MASTER TO MASTER_HOST='192.168.3.26', MASTER_USER='edu', MASTER_PASSWORD='edu', MASTER_PORT=3306, MASTER_LOG_FILE='Mysql-bin.000008',MASTER_LOG_POS=243, MASTER_CONNECT_RETRY=10;
		
	- 启动从库 slave start
	
### mysql中间件(value-atlas)

./atlas.py create 1<br>
修改/usr/local/mysql-proxy/conf/test.cnf文件
proxy-backend-addresses = {写数据库}
proxy-read-only-backend-addresses = {读数据库，逗号隔开}

	
### 安装缓存服务器（value-cs）

- 创建容器，并且进入

	./cs.py create 1 && ./cs.py ssh 1
	
- 修改/root/cs.sh，去掉注释，修改相互备份的ip

	/usr/local/bin/memcached -d -p 11211 -u root -x {ip} -X 11111 -v
	
- 重启容器 ./cs.py restart 1

### 安装文件系统 (value-fdfs)

- 创建容器，并且进入

	./fdfs.py create 1 && ./fdfs.py ssh 1
	
- 修改/etc/fdfs/storage.conf

	tracker_server=fs1.os:22122 #确保tracker server正确


### 安装课件转换服务 (value-cw)

./cw.py create 1 

### 安装画板服务 (value-nodejs)

./palette.py create 1


### 安装web服务 (eduos)

./eduos_as.py create 1