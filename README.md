
# mongod 入门篇
### MongoDB环境: 64位Linux
### MongoDB版本: 3.4.0
### MongoDB安装: homebrew install mongodb

+ mongod 主角 执行程序 数据库部署使用
+ mongo 数据连接的客户端 连接数据库才可以进行
+ mongoimport 和 mongoexport 数据的导入导出
+ mongodump 和 mongorestore 二进制数据的导入导出 不能读取
+ mongooplog 操作日志的回放

## 搭建简单的mongodb服务器

1. 首先，创建一个mongodb_simple的目录，进入目录中
2. 创建文件夹: data 用于存储数据库的数据文件
3. 创建文件夹: log 用于存储数据库日志文件
4. 创建文件夹: bin 用于存储数据库的可执行文件
5. 创建文件夹: conf 用于存储数据库的配置文件
6. 将/3.4.0/mongod 文件复制到 bin/文件中
7. 进入 conf 文件夹 创建 mongod.conf 文本文件 写入配置参数
   
   * port = 12345 //mongod 启动时监听的端口
   * dbpath = data //指明数据存储的目录 相对路径 绝对路径均可
   * logpath = log/mongod.log // 日志文件目录 
   * fork = true //linux表示启动一个后台进程 在window下无效
   

####  进入上层目录 ./bin/mongod  -f /mongod.conf  启动时指定的配置文件 

- about to fork child process, waiting until server is ready for connections.
- forked process: 6207
- child process stsrted successfully, parent exiting

## 连接mongodb服务器

  * ./bin/mongo --help //查看参数
  * ./bin/mongo 127.0.0.1:12345/test
  * 出现 > 这个符号表示数据库连接成功
  * 关闭数据库 db.shutdownServer()
  * 显示 shutdown command only works with the andmin database; try'use admin'
  * use admin
  * switched to db admin
  * db.shutdownServer() 或者 quit exit
  * tail -f log/mongod.log
    
### mongoDB基本使用
  >show dbs //查看当前系统中有多少数据库
  * 使用 use + 数据库名 切换数据库 
  * db.dropDatabase() 删除数据库
  > mongoDB一张表称作一个集合
  * mongo如何写入
  * db.+ 数据库表名称 +.insert() 只接受一个参数，参数即写入的文档,格式为json 
  * show collections 显示新建的数据表
  * db.+ 数据库表名称 +.find() 查找数据 不传参数返回所有数据,只接受一个参数
    参数为json格式 返回指定参数的数据
  * 写入多条数据 可以用javascript for(i = 0; i<100; i++) db.+数据库表名+.insert({x:i})
  * 计数 db.+ 数据库表名 +.find().count()
  * 多条数据查询 过滤 skip() 限制 limit() 排序 sort();
  db.+数据库表名+.find().skip(3).limit(2).sort({x:1});
  > mongoDB数据更新
  * db.+ 数据库表名 +update({x:1},{x:999}) 至少接受两个参数 一个是要查找的更新数据，一个是更新后的数据
  * 在更新时，有时候只更新部分字段 需要更新的字段使用{$set:{需要更新的数据}}
    防止其他数据被覆盖丢失
  * 更新一条不存在的数据时,会自动创建 使用 db.+ 数据库表名 .+update({x:1},{x:999},true)
  * 更新多条数据时 db.+ 数据库表名 +update({x:1},{$set:{x:999}},false, true);
  > mongoDB数据删除
  * remove() 只接受一个参数
  * drop 清空表数据
  > mongoDB创建索引
  * db.+ 数据库表名 +.getIndexes() 查找集合的索引情况
  * db.+ 数据库表名 +.ensureIndex({x:1}) 创建索引 参数正数是正向排序 反向则逆向排序
  文档数量较多创建索引消耗的时间过多，负载较重不能直接使用这个命令，在使用数据库之前就把
  索引创建完毕。否则，严重影响数据库的性能
  ###mongoDB常见的查询索引
  1._id索引
  - _id索引是绝大数集合默认建立的索引
  - 对于每一个插入的数据，MongoDB都会自动生成唯一的_id字段
      
  2.单键索引
  - 单键索引是最普通的索引
  - 与_id索引不同，单键索引不会自动创建
   如：一条记录形式为：{x:1, y:2, z:3}
       
  3.多键索引
  - 多键索引与单键索引创建形式相同，区别在与字段的值
  - 单键索引：值为一个单一的值，例如字符串，数字或者日期
  - 多键索引：值具有多条记录，例如数组
  
  4.复合索引
  - 当我们的查询条件不过只有一个时，就需要建立复合索引
  - 插入{x:1, y:2, z:3}记录
  - 按照x与y的值查询
  - 创建索引：db.collection.ensureIndex({x:1, y:1})
  - 使用{x:1, y:1}作为条件查询
  
  5.过期索引
  - 是在一段时间后会过期的索引
  - 在索引过期后，相应的数据会被删除
  - 适合存储一些在一段时间之后会失效的数据比如用户的登录信息
   存储日志
  - 建立方法：
   db.collection.ensureIndex({time:1},{expireAfterSeconds:10})
  
  *过期索引的限制*
  
  - 存储在过期索引字段的值必须是指定的时间类型
  说明：必须是ISODate或者ISODate数组，不能使用时间戳，否则不能被删除
  - 如果指定ISOData数组，则按照最小的时间进行删除
  - 过期索引不能是复合索引
  - 删除时间不是精确
    说明：删除过程是由后台程序60s跑一次，而且删除也需要一些时间，所以存在误差
    
  6.全文索引 
   - 对字符串与字符数组创建全文可搜素的索引
   - 使用情况：{author:'', title:'', article:''}
   - 建立方法：
       * db.articles.ensureIndex({key:'text'})
       * db.articles.ensureIndex({key_1:'text',key_2:'text'})
       * db.articles.ensureIndex({'$**':'text'})
   - 查询全文索引
       * db.articles.find({$text:{$serch:"aa"}}) 
   - 全文索引相似度：
       * $meta 操作符：{score:{$meta:'textScore'}}
       * 写在查询条件后面可以返回结果的相似度
       * 与 sort 一起使用，可以达到很好的使用效果
   - 全文索引的使用限制
       * 每次查询，只能指定一个$text查询
       * $text查询不能出现在 $nor 查询中
       * 查询中包含了 $text， hint不能起作用
  ### 地理位置索引
   - 索引属性：
       * 名字： name 指定
         * db.collection.ensureIndex({},{name:''})
       * 唯一性： unique 指定
         * db.collection.ensureIndex({},{unique:true/false})
       * 稀疏性 sparse 指定
         * db.collection.ensureIndex({},{sparse:true/false})
       * 是否定时删除 expireAfterSeconds 指定
         * TTL，过期索引
   - 地理位置索引
       * 将一个点的位置存储在MongeDB中，，创建索引，可以按照位置来查找其他点
         - 2d索引，用于存储和查找平面上的点
            ---
            > 创建方式
            - db.collection.ensureIndex({w:"2d"})
            - 位置表示方式:经纬度［经度，纬度］
            - 取值范围：经度［－180，180］纬度［－90，90］
            - 查询方式：
            (1) $near查询：查询距离某个点最近的点 返回最近的100点
                可以使用$maxDistance
            (2) $geoWithin查询：查询某个形状内的点
                形状表示
                a.$box:矩形，使用 {$box:[[<x1>,<y1>],[<x2>,<y2>]]}
                b.$center: 圆形，使用  {$center:[[<x1>,<y1>],r]} 表示
            (3) $polygon: 多边形，使用 {$polygon:[[<x1>,<y1>],[<x2>,<y2>],[x3,y3]]}表示    
            - geoNear 查询：
                geoNear 使用runCommand命令进行使用，使用如下
                 db.runCommand({geoNear:<collection>,near:[x,y],minDistance:(对2d索引无效)，maxdistance:...})
                 
            ---
         - 2dsphere索引，用于存储和查找球面上的点
           * 创建方式：db.collection.ensureIndex({w:"2dsphere"})
           * 位置表示方式：
             - GeoJSON 描述一个点，一条直线，多边形等形状
             - 格式：{type:"", coordinates:[<coordinates>]}
   - 索引构建情况分析
   
       > 如何评判当前索引构建情况
           
          1.mongostat工具介绍
            
            mongostat:查看 mongodb运行状态的程序，
            使用： mongostat -h 127.0.0.1:12345
            字段说明：
            索引情况：idx miss 
         
          2.profile集合的介绍
          
            db.getProfilingStatus() 查看当前的profile设置
            db.setProfilingLevel(2) 0 关闭 1 2 mongod记录所有的操作
            
          3.日志介绍
          
            ./conf/mongod.conf 添加配置文件 verbose = vvvvv  v是级别 最大5个  
         
          4.explain分析
   ### MongoDB安全
   > 安全概述
   
       * 1. 最安全的是物理隔离：不现实
       * 2. 网络隔离其次   
       * 3. 防火墙隔离
       * 4. 用户名密码
       
   > 开启权限认证
       
       1. auth 开启
       
       2. keyfile 开启
   > MongoDB 创建用户
       
       1.创建语法：createUser
       
       2.{user: "<name>",
          pwd: "<cleartext password",
          customData:{<any information>},
          roles:[{role:"<role>", db:"<database>"}]
       }
       
       3.角色类型：内建类型 (read, readWrite, dbAdmin, dbOwner, userAdmin)
       
       