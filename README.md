# 1 官方安装文档

https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-red-hat/

# 2 安装过程

## 2.1 创建  /etc/yum.repos.d/mongodb-org-6.0.repo

内容如下：

```
[mongodb-org-6.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/6.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-6.0.asc
```

## 2.2 执行安装

### 2.2.1 安装最新版本

```
$sudo yum install -y mongodb-org
```



### 2.2.2 安装指定版本

```
$sudo yum install -y mongodb-org-6.0.4 mongodb-org-database-6.0.4 mongodb-org-server-6.0.4 mongodb-org-mongos-6.0.4 mongodb-org-tools-6.0.4
```

### 2.2.3 避免版本升级

```
vi /etc/yum.conf
```

加上：

```
exclude=mongodb-org,mongodb-org-database,mongodb-org-server,mongodb-mongosh,mongodb-org-mongos,mongodb-org-tools
```

### 2.2.4 指定文件夹

```
vi /etc/mongod.conf
```

storage;

dbPath = /app/mongodb/db
systemLog.path = /app/mongodb/log




# 3 部署mongoDB 副本集

## 3.1 官方文档

https://www.mongodb.com/docs/manual/tutorial/deploy-replica-set/

## 3.2 设置环境变量

vi /etc/profile

MONGO_HOME=/XXX
PATH=$MONGO_HOME/bin:$PATH
export MONGO_HOME PATH

## 3.3 默认配置文件

mognDB 默认启动脚本：
/usr/bin/mongod 
mognDB 配置文件：
/etc/mongod.conf


security:
  authorization: 'enabled'
  clusterAuthMode: "keyFile"
  keyFile: /etc/mongod/rs.key

replication:
  oplogSizeMB: 82400 #复制操作日志的最大大小，以M为单位，默认情况下，mongod进程基于最大可用空间创建oplog，对于64位系统，oplog通常占可用磁盘空间的5%；
  replSetName: tx  #副本集的名称
  enableMajorityReadConcern: false

## 3.4  生成认证 rs.key 

mkdir -p /etc/mongod

openssl rand -base64 741 > rs.key

# 4  mongoDB 命令 

## 4.1  启动

mongod  -f /etc/mongod.conf --replSet=tx

## 4.2 关闭服务 

mongod --shutdown --dbpath /app/mongodb/db

## 4.2 客户端创建副本集



$mongosh 

```
>use admin
>config = {_id: 'tx', members: [
{_id: 0, host: '127.0.0.1:27017',priority:2},
{_id: 1, host: '127.0.0.2:27017',priority:1}, 
{ "_id" : 2,"host" : "127.0.0.3:27017","arbiterOnly" : true}
>rs.initiate(config)
```





# 5 配置用户 



进入主节点：

use admin

```
db.createUser(
  {
    user: "root",
    pwd: "@123_79_200",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
  }
)
db.auth("root", "@123_79_200")

```

