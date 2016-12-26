# mongod
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
  * db.shutdownServer()
  * tail -f log/mongod.log
    

