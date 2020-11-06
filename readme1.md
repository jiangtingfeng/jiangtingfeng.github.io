### 搭建mongodb集群

>  ### 1.下载相应的安装包，上传到服务器  ----开源项目包找焦植
>
>  #### 2. 解压安装包，进行相应的文件夹创建，以及创建启动时需要的配置文件
>
>  ### 1.单节点创建
>
>  	>  ![image-20201029212611458](C:\Users\j30007830\AppData\Roaming\Typora\typora-user-images\image-20201029212611458.png)
>
>  ​	  
>
>  > 2.根据配置文件进行相应文件的创建
>  >
>  > ```yaml
>  > systemLog:
>  > destination: file
>  > path: "/opt/local/mongodb-one/single/log/mongod.log"
>  > logAppend: true
>  > storage:
>  > dbPath: "/opt/local/mongodb-one/single/data/db"
>  > journal:
>  >  enabled: true
>  > processManagement:
>  > fork: true
>  > net:
>  > bindIp: localhost,172.16.214.253
>  > port: 27017
>  > ```
>  >
>  > 3. 启动服务
>  >
>  >    ```shell
>  >    /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/single/mongod.conf   // kill -2 pid  杀死进程
>  >    ```
>  >
>  > 4. 使用客户端建立连接
>  >
>  >    ```shell
>  >    /opt/local/mongodb-one/bin/mongo --host=172.16.214.253 --port=27017
>  >    ```
>  >
>  > 5. ![image-20201029213437832](C:\Users\j30007830\AppData\Roaming\Typora\typora-user-images\image-20201029213437832.png)

### 伪副本集搭建（本节点,副本节点，仲裁节点(投票)）（与gs数据库(主从集群)的区别 没有固定的主集）

##### 选举规则

>  		1. 票数最多，且票数大于节点数的一半
>  		2. 票数相同，版本最新为主节点
>  		3. 默认所有副本集都可以投票且权重一样，但是仲裁者不能对自己进行投票

##### 故障问题

> 1. 主节点宕机，会开启选举，服务正常
> 2. 副节点宕机，不会开启选举，服务正常
> 3. 主节点+仲裁节点宕机   开启选举  不能选举成功  服务异常
> 4. 副节点+仲裁节点宕机   不开启选举  服务降级  变为单节点服务  服务正常

![image-20201030092329227](C:\Users\j30007830\AppData\Roaming\Typora\typora-user-images\image-20201030092329227.png)

```yaml
>  1.每个副本集的配置文件
>
>  ```yaml
>  systemLog:
>     destination: file
>     path: "/opt/local/mongodb-one/replica_sets/myrs_27018/log/mongod.log"  #日志文件位置
>     logAppend: true   #文件追加
>  storage:
>     dbPath: "/opt/local/mongodb-one/replica_sets/myrs_27018/data/db"   #数据文件位置
>     journal:
>      enabled: true
>  processManagement:
>     fork: true
>     pidFilePath: "/opt/local/mongodb-one/replica_sets/myrs_27018/log/mongod.pid"  # 进程管理信息文件位置
>  net:
>     bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
>     port: 27018   #端口号  同一台主机时变端口 
>  replication:
>     replSetName: myrs    #副本集统一的名称   不变
>  ```
>
>  2.依次启动副本集
>
>  ```shel
>  /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/replica_sets/myrs_27018/mongod.conf
>  ```
>
>  3.进行客户端的连接并进行相应的初始化
>
>  ```shell
>  /opt/local/mongodb-one/bin/mongo --host=172.16.214.253 --port=27017 / --host=172.16.214.253 --port=27018  
>  /--host=172.16.214.253 --port=27017
>  ```
>
>  4.初始化各个副本集
>
>  1. 初始化主节点
>
>  ```js
>    rs.initiate(configuration)   #主节点初始化 configuration 可以不进行传入  rs.config()/ rs.status()  可以查看相应的信息
>  ```
>
>  ​	2.主节点添加副本节点和仲裁节点
>
>  ```js
>    rs.add(ip+host,arbiterOnly)  # 添加副本集 第一个参数 ip+host  指定副本集， 参数arbiterOnly 为true时 则添加的该节点为仲裁节点
>    rs.addArb(ip+host)   # 添加仲裁节点     参数 ip+host  指定副本集
>  ```
>
>  3.初始化副本节点
>
>  ``` js
>  rs.slaveOk(Boolean)  #承认自己是主节点的副本节点 true 同意  false  拒绝  默认是 true
>  ```
>
>  4.初始化仲裁者节点(只保存配置信息)
>
>  ```js
>  rs.slaveOk();    # 只能看到local 库  不会同步数据到仲裁节点
>  ```
```

### 分片集群搭建

>  ##### 搭建分片集群图

![image-20201029220038730](C:\Users\j30007830\AppData\Roaming\Typora\typora-user-images\image-20201029220038730.png)

> 1.第一套副本集
>
> ```yaml
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27018/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27018/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27018/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27018   #端口号  同一台主机时变端口 
> replication:
> replSetName: myshardrs01    #副本集统一的名称   不变
> sharding:
> clusterRole: shardsvr   # 分片角色和配置角色   shardsvr   configsvr
> 
> #副本
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27118/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27118/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27118/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27118   #端口号  同一台主机时变端口 
> replication:
> replSetName: myshardrs01    #副本集统一的名称   不变
> sharding:
> clusterRole: shardsvr   # 分片角色和配置角色   shardsvr   configsvr
> 
> #仲裁者
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27218/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27218/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27218/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27218   #端口号  同一台主机时变端口 
> replication:
> replSetName: myshardrs01    #副本集统一的名称   不变
> sharding:
> clusterRole: shardsvr   # 分片角色和配置角色   shardsvr   configsvr
> ```
>
> 2.配置好相应文件后启动第一套的三个副本集
>
> ```shell
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myshardrs_27018/mongod.conf
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myshardrs_27118/mongod.conf
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myshardrs_27218/mongod.conf
> ```
>
> 3.第二套与第一套一样创建
>
> ```yaml
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27318/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27318/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27318/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27318   #端口号  同一台主机时变端口 
> replication:
> replSetName: myshardrs02    #副本集统一的名称   不变
> sharding:
> clusterRole: shardsvr   # 分片角色和配置角色   shardsvr   configsvr
> 
> #副本
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27418/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27418/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27418/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27418   #端口号  同一台主机时变端口 
> replication:
> replSetName: myshardrs02    #副本集统一的名称   不变
> sharding:
> clusterRole: shardsvr   # 分片角色和配置角色   shardsvr   configsvr
> 
> #仲裁者
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27518/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27518/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myshardrs_27518/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27518   #端口号  同一台主机时变端口 
> replication:
> replSetName: myshardrs02    #副本集统一的名称   不变
> sharding:
> clusterRole: shardsvr   # 分片角色和配置角色   shardsvr   configsvr
> ```
>
> 4.启动服务
>
> ```shel
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myshardrs_27318/mongod.conf
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myshardrs_27418/mongod.conf
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myshardrs_27518/mongod.conf
> ```
>
> 5.搭建配置服务
>
> ```yaml
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27019/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27019/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27019/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27019   #端口号  同一台主机时变端口 
> replication:
> replSetName: myconfigrs    #副本集统一的名称   不变
> sharding:
> clusterRole: configsvr   # 分片角色和配置角色   shardsvr   configsvr
> 
> #副本
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27119/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27119/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27119/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27119   #端口号  同一台主机时变端口 
> replication:
> replSetName: myconfigrs    #副本集统一的名称   不变
> sharding:
> clusterRole: configsvr   # 分片角色和配置角色   shardsvr   configsvr
> 
> #仲裁者
> systemLog:
> destination: file
> path: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27219/log/mongod.log"  #日志文件位置
> logAppend: true   #文件追加
> storage:
> dbPath: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27219/data/db"   #数据文件位置
> journal:
>  enabled: true
> processManagement:
> fork: true
> pidFilePath: "/opt/local/mongodb-one/sharded_cluster/myconfigrs_27219/log/mongod.pid"  # 进程管理信息文件位置
> net:
> bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> port: 27219   #端口号  同一台主机时变端口 
> replication:
> replSetName: myconfigrs    #副本集统一的名称   不变
> sharding:
> clusterRole: configsvr   # 分片角色和配置角色   shardsvr   configsvr
> ```
>
> 6.启动服务
>
> ```shel
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myconfigrs_27019/mongod.conf
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myconfigrs_27119/mongod.conf
> /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myconfigrs_27219/mongod.conf
> ```
>
> 7.初始化节点
>
> ```
> 主节点
> rs.initiate()
> rs.add(host);
> rs.addAbr(host);  #  配置服务的时候直接加两个副本
> 副本节点
> rs.slaveOk();
> ```
>
> 8.创建路由节点
>
> > 第一个路由节点的创建和连接
> >
> > ```yaml
> > ## 创建相应的文件  创建相应的配置文件 mongos.conf
> > systemLog:
> > destination: file
> > path: "/opt/local/mongodb-one/sharded_cluster/mymongos_27017/log/mongod.log"  #日志文件位置
> > logAppend: true   #文件追加
> > storage:
> > dbPath: "/opt/local/mongodb-one/sharded_cluster/mymongos_27017/data/db"   #数据文件位置
> > journal:
> >  enabled: true
> > processManagement:
> > fork: true
> > pidFilePath: "/opt/local/mongodb-one/sharded_cluster/mymongos_27017/log/mongod.pid"  # 进程管理信息文件位置
> > net:
> > bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> > port: 27017   #端口号  同一台主机时变端口 
> > sharding:
> > # 指定配置节点副本集
> > configDB: myconfigrs/172.16.214.253:27019,172.16.214.253:27119,172.16.214.253:27219
> > ```
> >
> > 启动mongos的服务 可以查看数据库 但是不能插入数据
> >
> > ```shell
> > /opt/local/mongodb-one/bin/mongos -f /opt/local/mongodb-one/sharded_cluster/mymongos_27017/mongos.conf
> > ```
> >
> > 添加分片副本集
> >
> > ```shel
> > mongos>sh.add("myshardrs01/172.16.214.253:27018,172.16.214.253:27118,172.16.214.253:27218");  //分片副本集1
> > mongos>sh.add("myshardrs02/172.16.214.253:27038,172.16.214.253:27418,172.16.214.253:27518");  //分片副本集2
> > mongos>sh.status();
> > mongos>sh.enableSharding("article");   //给库开启分片功能
> > mongos>sh.shardCollection("article.student",{nickname:"hashed"});   //集合分片  sh.shardCollection(namespace, key, unique);
> > 段  提升性能（唯一 索引）   //hash分片
> > mongos>sh.shardCollection("article.author",{age:1});   //集合分片  sh.shardCollection(namespace, key, unique);命名空间  分片字
> > 段  提升性能（唯一 索引）  //范围分片  分片窗口大小默认64M  可以在 config库中对该值进行修改
> > //修改范围分片的窗口大小
> > use config
> > db.settings.save({_id:"chunksize", value:1})    //   默认是64M     db.settings.save({_id:"chunksize", value:64}) 
> > ```
> >
> > 路由二节点搭建
> >
> > ```yaml
> > ## 创建相应的文件  创建相应的配置文件 mongos.conf
> > systemLog:
> > destination: file
> > path: "/opt/local/mongodb-one/sharded_cluster/mymongos_27117/log/mongod.log"  #日志文件位置
> > logAppend: true   #文件追加
> > storage:
> > dbPath: "/opt/local/mongodb-one/sharded_cluster/mymongos_27117/data/db"   #数据文件位置
> > journal:
> >  enabled: true
> > processManagement:
> > fork: true
> > pidFilePath: "/opt/local/mongodb-one/sharded_cluster/mymongos_27117/log/mongod.pid"  # 进程管理信息文件位置
> > net:
> > bindIp: localhost,172.16.214.253  #绑定的ip   同步主机变ip
> > port: 27117   #端口号  同一台主机时变端口 
> > sharding:
> > # 指定配置节点副本集
> > configDB: myconfigrs/172.16.214.253:27019,172.16.214.253:27119,172.16.214.253:27219
> > #security:
> > 	#开启权限认证
> > 	#authorization: enabled
> > ```
> >
> > 启动路由服务：
> >
> > ```shell
> >  /opt/local/mongodb-one/bin/mongos -f /opt/local/mongodb-one/sharded_cluster/mymongos_27117/mongos.conf   
> > ```
> >
> > springData
> >
> > ```yaml
> > mongodb://172.16.214.253:27017,172.16.214.253:27117/article #连接路由节点
> > ```

###  安全认证(admin库)

>  1.角色表
>
>  ![image-20201030084524234](C:\Users\j30007830\AppData\Roaming\Typora\typora-user-images\image-20201030084524234.png)

> 2.进入admin库创建用户并授予相应的权限
>
> ```shell
> use admin;
> #创建超级管理员  用户名
> db.createUsre({user:"myroot",pwd:"123456",roles:[{"role":"root","db":"admin"}]})
> //或
> db.createUser({user:"myroot",pwd:"123456",roles:["root"]})
> //创建专门用来管理admin库的账号myadmin,只用来作为用户权限的管理
> db.createUser({user:"myadmin",pwd:"123456",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})
> //查看已经创建了用户的情况
> db.system.users.find();
> //删除用户
> db.dropUser("myadmin");
> //修改密码
> db.changeUserPassword("myroot","123456");
> use admin 
> //验证用户账号密码
> db.auth("myroot","123456");
> //创建普通用户
> use article;
> db.createUser({user:"jtf",pwd:"123456",roles:[{role:"readWrite",db:"articledb"}]});
> db.auth("jtf","123456");
> ```
>
> #### 1.服务端开启认证(单实例)
>
>  1. 重启服务是带上参数
>
>     ```shell
>     /opt/local/mongodb-one/bin/mongod -f /opt/local/mongodb-one/sharded_cluster/myshardrs_27018/mongod.conf  --auth
>     ```
>
>     2. 在配置文件中加上
>
>     ```yaml
>     security:
>     	#开启权限认证
>     	authorization: enabled
>     ```
>
>     3. 客户端连接后进行登入
>
>     ```shell
>     db.auth(user,password);
>     ```
>
>     ```yaml
>     #url配置： 单节点 
>     uri: mongodb://jtf:123456@172.16.214.253:27017/articledb
>     #副本集
>     uri: 
>     spring.data.mongodb.uri= mongodb://jtf:123456@172.16.214.253:27018,172.16.214.253:27019,172.16.214.253:27020/article?connect=replicaSet&slaveOk=true&replicaSet=myrs
>     #分片配置
>     mongodb://jtf:123456@172.16.214.253:27017,172.16.214.253:27117/article #连接路由节点
>     ```
>
> #### 2.服务端开启认证(副本集)
>
> ![image-20201030092337704](C:\Users\j30007830\AppData\Roaming\Typora\typora-user-images\image-20201030092337704.png)

> 1. 登入主节点然后创建普通用户(会同步到副本集节点)
>
> 2. 创建副本集认证的key文件
>
>    ```shell
>    openssl rand -base64 90 -out ./mongo.keyfile
>    chmod 400 ./mongo.keyfile  #当前用户只读
>    // 将keyfile文件
>    cp mongo.keyfile /mongodb/replica_sets/myrs_27017
>    cp mongo.keyfile /mongodb/replica_sets/myrs_27018
>    cp mongo.keyfile /mongodb/replica_sets/myrs_27019
>    在配置文件中指定副本节点之间相互认证的keyfile文件
>    security:
>    	#指定keyfile鉴权文件
>    	keyfile: /mongodb/replica_sets/myrs_27017/mongo.keyfile    #  /mongodb/replica_sets/myrs_27018/mongo.keyfile 
>    															# /mongodb/replica_sets/myrs_27017/mongo.keyfile 
>    	#开启权限认证
>    	authorization: enabled
>    ```
>
> #### 3.服务端开启认证(分片集群)
>
> ​	![image-20201029220038730](C:\Users\j30007830\AppData\Roaming\Typora\typora-user-images\image-20201029220038730.png)

> 1. 登入路由节点
>
>    ```shel
>    /opt/local/mongodb-one/bin/mongo --host=172.16.214.253 --port=27017
>    mongos>use admin
>    mongos>db.createUser({user:"myroot",pwd:"123456",roles:["root"]})
>    mongos>db.auth("myroot","123456")
>      	   db.createUser({user:"bobo",pwd:"123456",roles:[{role:"readWrite",db:"articledb"}]})
>    为每个节点服务添加keyfile
>    	openssl rand -base64 90 -out ./mongo.keyfile
>    	chmod 400 ./mongo.keyfile  #当前用户只读
>    	// 将keyfile文件 复制到各个节点服务上
>    ```
