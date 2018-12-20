#　zk学习笔记

## zk数据模型介绍
**zookeeper数据模型一**

- 树形结构，类似tree.js这样的插件

>                            /
                            |
                            /慕课网
                |                                   |
            /慕课网/前端                          /慕课网/后端
       |                |                    |                   |
    /慕课网/前端/css /慕课网/前端/js      /慕课网/后端/JAVA /慕课网/后端/大数据

**zookeeper数据模型二**

- 目录结构
>       /
>       /慕课网
>               /慕课网/后端
>                           /慕课网/后端/大数据
>                           /慕课网/后端/JAVA
>               /慕课网/前端
>                           /慕课网/前端/CSS
>                           /慕课网/前端/JS

**zookeeper数据模型三**

- 类似Linux/unix的文件目录：/usr/local/...
- 每个节点都叫znode,且可以有子节点，也可以有数据
- 每个节点分为临时节点与永久节点，临时节点与客户段断开连接后自动消失（可能在一分钟后，根据keepalive设置心跳时间决定）

**zookeeper数据模型四**

- 节点自带版本号，对其更改会版本号自增（乐观锁）
- 节点信息可以使用命令行获取
- 删除/修改节点信息时，如果版本号不对会报错（如果不加版本号，会直接删除/修改）

**zookeeper数据模型五**

- 节点内部可以存储数据，但最好控制在几K
- 节点可以设置权限acl，通过权限控制限制客户端的访问
 
## zk客端连接

zookeeper安装目录下，bin/目录下，zkServer.sh启动脚本`./zkServer.sh restart`是重启zk服务器。

bin/下zkCli.sh客户端连接脚本，`./zkCli.sh` 直接执行脚本文件就可以启动连接zk服务器，打出`help`可以列出所有命令。

## zookeeper的作用体现

- 分布式锁（version的使用）
- 高可用选举机制（master选举模式），首脑宕机后会自动从节点中选举出一个节点代替首脑节点工作，且保证这个节点是唯一的。
- 统一配置文件管理，只需要修改一台机器的文件，其他所有在zk上的机器都会同步
- 发布与订阅（注册中心），（dubbo发布者将数据）一台机器在一个znode上存储数据，（订阅者）另一台服务器在zonde上读取数据  
- 保证数据强一致性，根据（zk同步数据机制），（集群下）一个节点的信息会同步到所有子节点上

## zk基本命令

进入客户端之后我们先使用`help`命名查看有那些命名：
>       ZooKeeper -server host:port cmd args
        stat path [watch] //查看节点信息，不包含数据信息，设置‘监察’
        set path data [version]//修改节点信息，不带版本号直接删除，带版本号的话需要version正确才能修改成功
        ls path [watch]//查看目录下节点（只有名称）
        delquota [-n|-b] path
        ls2 path [watch]//查看该节点下所有子节点，且打印出此节点的详情（不带数据），相当于ls+stat命名
        setAcl path acl//设置权限
        setquota -n|-b val path
        history
        redo cmdno
        printwatches on|off
        delete path [version]
        sync path
        listquota path
        rmr path
        get path [watch]//获取节点详情+节点数据
        create [-s] [-e] path data acl//创建节点 -s自增节点 -e创建临时节点（临时节点客户端断开连接后会自动删除）
        addauth scheme auth
        quit
        getAcl path
        close
        connect host:port

上面是常用的几个命名的简要分析。

### 下面详细解释重要命令：

stat 查看节点信息，不包含数据信息，设置‘监察’
```
stat /cdz
```
```
cZxid = 0x8 //创建节点之后zk为此节点分配Zxid 
            //Zxid 每一个节点创建时，zk都会为其分配一个Zxid（zookeeper transaction id/zookeeper事务id），
            //且是保持自增的，也就是说如果A节点的Zxid小于B节点，那么必然是A在B之前创建
ctime = Fri Dec 21 00:38:58 CST 2018//创建时间
mZxid = 0x8//每一次修改，mZxid都会在Zxid的基础上自增1，此会最后一次修改自增数（故也就可以使用mZxid-cZxid=修改次数）
mtime = Fri Dec 21 00:38:58 CST 2018//最后一次修改时间
pZxid = 0x8//最后一个子节点的Zxid
cversion = 0//子节点版本号，每次子节点变化（增加或者删除）其就会自增
dataVersion = 0//版本号（每次修改会自增）
aclVersion = 0//权限版本好
ephemeralOwner = 0x0//判断是否为临时节点 0x0不为临时节点，0x1003fa618230003为临时节点
dataLength = 3//数据的字符串长度
numChildren = 0//子节点数量
```


get 获取节点详情+节点数据
```
get /cdz1
```
```
666
cZxid = 0x9
ctime = Fri Dec 21 01:11:33 CST 2018
mZxid = 0x9
mtime = Fri Dec 21 01:11:33 CST 2018
pZxid = 0x9
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x1003fa618230003
dataLength = 3
numChildren = 0
```

create 创建节点
```
create /cdz 1111
```
```
Created /cdz
```

创建一个临时节点
```
create -e /cdz/tem 2222
```
```
Created /cdz/tem
```

创建一个自增节点
```
create -s /cdz/insert1 1111
```
```
Created /cdz/insert10000000004
```

修改节点
```
set /cdz/insert 2222
```
返回更改后的节点信息
```
cZxid = 0x19
ctime = Fri Dec 21 01:42:58 CST 2018
mZxid = 0x1c
mtime = Fri Dec 21 01:45:48 CST 2018
pZxid = 0x19
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x1003fa618230006
dataLength = 3
numChildren = 0
```

删除节点
```
delete /cdz/insert 无返回
```

### watch机制

