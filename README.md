# 安装
- 下载: www.mongodb.org 下载最新的stable版
- 解压文件 tar zxvf 文件名.tgz
- 不用编译，本身就是编译后的二进制可执行文件
- 移动到指定目录 mv ./mongodb ./xxx/mongodb
- bin目录中的内容
## 核心
- mongod 数据库服务端
- mongos 分片的路由器
- mongo  客户端

## 导出导入
- mongodump 导出bson数据
- mongorestore 导入bson
- bsondump bson转换为json
- mongoexport 导出json,csv,tsv格式
- monogoimport 导入json，csv,tsv

## 诊断工具
- mongostats
- mongotop
- mongoniff 检查mongo运行状态

## 启动服务
- ./bin/mongod --dbpath /path/mongodata --logpath /path/to/log --fork --port 27017
- fork 后台运行
- --smallfiles 选项来启动 暂用非常小的空间 300-400M  没这个选项会占用3-4G

# 命令
- show dbs;
- show collections;
- use dbName;  会自动创建该库
- db.help(); 查看操作库的方法
- db.createCollection('user');  
- db.user.insert();  可以自动创建
- db.dropDatabase();
- db.stu.remove({gender:'m'}, true)  true表示只删一行   默认false，匹配几行，删几行
- $set 修改某列
- $unset 删除某列
- $rename 重命名某列
- $inc 增长某个列
- $setOnInsert 
- db.stu.update({}, {$set:{}}, {multi: true}); 修改多行 
- db.stu.update({}, {$set:{}}, {upsert: true}); 无匹配的行，直接插入改行
- $setOnInsert 当upsert为true，并且发生了insert操作时i，可以补充的字段
- db.stu.update({}, {$set:{}, $setOnInsert:{}}, {upsert: true}); V2.4.0以上才能使用
- $nor 列举的条件都不成功，则为真
- $in  满足一条为真  如果是数组，数组元素有一条满足in的条件即可
- $all 满足所有为真  主要针对属性为数组的操作
- $exist 某列存在则为真
- $mod   满足求余条件则为真  $mod:[5,0], 对5取余为0的
- $type  满足为某数据类型则为真
- $where find({$where: 'this.age > 10'}) js的条件判断表达式  必须把二进制转成json对象进行，效率低，不推荐使用
- $regex 正则表达式 find({name:{$regex:/^abc.*/}})  效率低，不推荐使用

#  游标
- mongodb的地步引擎是js引擎，可以mongo在命令行终端直接使用js的语法进行操作，比如for循环
- 游标不是查询结果，而是查询的返回资源或者接口
- var cursor = db.collection.find(query);
- cursor.hasNext(), 判断游标是否已经取到尽头
- cursor.next()  取出游标的下个单元
- printjson
```javascript
while(cursor.hasNext()) {
    printjson(cursor.next())
}

for(var cursor = db.find(); cursor.hasNext(); ) {
    printjson(cursor.next())
}

cursor.forEach(function(o) {
    printjson(o)
})

var cursor = db.bar.find().skip(500);
cursor.forEach(function(o) {
    printjson(o)
})


var cursor = db.bar.find().skip(500).limit(10);
cursor.forEach(function(o) {
    printjson(o)
})


cursor.toArray();
cursor.toArray()[4];
toArray 不要随意用
原因: 会把所有的行立即以对象形式组织在内存里，会耗耗时间，占用内存，给数据库会造成压力
取少量几行可以使用此方法
```

## 索引
- 索引提高查询速度，降低写入速度，权衡常用的查询字段，不必在太多列上建索引
- 在mongodb中，索引可以按字段升序，降序来创建，便于排序
- 默认是用哪个btree来组织文件，2.4版本以后，也允许建立hash索引
- 索引作用类型: 单列索引  多列索引  子文索引
- 索引性质: 普通索引 唯一索引 稀疏索引 哈希索引
- db.stu.find({dn:99}).explain();  查看查询计划
- BasicCursor 说明没有使用索引，全表遍历查询
- nscannedObjects 理论上要扫描多少行
- db.stu.ensureIndex({sn: 1}); 1 表示升序索引  -1 表示降序索引
- BtreeCursor sn_1  使用了btree索引
- db.stu.getIndexes(); 查看索引
- db.stu.dropIndex({name:-1}); // -1 一定把索引类型指定上 正序，倒序
- db.stu.ensureIndex({sn:1, name:1}) 多列索引  跟两个列单独建索引不一样
- db.shop.ensureIndex({'spc.area': 1}) 子文档索引
- db.stu.ensureIndex({sn: 1}, {unique: true}); 唯一索引
- 稀疏索引特点：如果针对filed做碎银，针对不含field列的子文档，将不建立索引
- 与之相对的普通索引，会把该文档的filed列的值认为为NULL,并建索引
- 适用于小部分文档含有某列时
- db.stu.ensureIndex({sn: 1}, {sparse: true}); 稀疏索引
- 建立完稀疏索引以后，如果sn字段不存在，将查询不到该文档
- 如果内容最后一行没有sn列
- 如果分别加普通索引和稀疏索引
- 对于最后一行的email分别当成null 和 忽略最后一行来处理
- 根据{email: null}来查询，普通索引能查到，稀疏索引查不到最后一行
- 哈希索引: 具有散列特性，范围查询，顺序读取查询不建议使用， 字符串的值可以使用hash
- db.tea.ensureIndex({email: 'hashed'}) 创建hash索引
- 重建索引: 一个表经过多次修改后，导致表的文件产生空洞，索引文件也如此
- 可以通过索引的重建，开提高索引的效率
- db.tea.reIndex() 
- 重建所有索引: 索引文件碎片，提高索引效率

## 用户管理
- 有一个admin数据库，需要先切换到admin库
- use admin
- mongo以数据库为单位，每个库都有自己管理员
- db.addUser('sa', 'passwd', false); // false-是否只读，
- 启动mongo的时候，需要加入--auth选项， mongodb才会进行认证
- mongod --dbpath --logpath --fork --auth --port
- db.auth('sa', 'passwd');
- 这里加的用户属于超级用户

- user test
- db.addUser('web', 'web123', false), false 是否只读
- db.auth('web','web123');  认证
- db.changeUserPassword('web','web321'); 修改密码
- db.removeUser('web') 删除用户


## 备份与恢复
- -d 库名
- -c 集合名
- -f 字段名
- -o 导出的文件名
- -q 查询条件
- --csv  导出csv格式
- mongoexport -d test -c stu -f sn,name -q '{sn:{$lte: 1000}}' -o ./test.stu.json
- mongoexport -d test -c stu -f sn,name -q '{sn:{$lte: 1000}}' --csv -o ./test.stu.csv

- -d 库名
- -c 集合名
- --type 导入文件类型
- --file 导入内容的文件
- mongoimport -d test -c animal --type json --file ./test.stu.json
- mongoimport -d test -c bird --type csv --headline -f sn,name --file ./test.stu.csv
- 导出二进制内容，索引什么的都不会丢，用户数据库的备份
- 导出json，csv格式，用户数据库之间的数据交换

- mongodump
- -d 库名
- -c 表名
- -f filed1,field2 
- 默认是导出到mongo下dump目录
- 导出的文件放在以database命名的目录下
- 每个表导出2个文件，分别是bson结构的数据库文件，json的索引信息
- 如果不生命表名，导出所有的表
- mongodump -d test -c tea
- mongodump -d test
- 导入
- mongorestore -d test  --directoryperdb /dum/test
- 二进制备份，不仅可以备份数据，还可以备份索引

## replication set 复制集
- replication set   多台服务器维护相同数据副本，其中一台出问题，其他仍然可以工作
- 创建文件
- mkdir /home/m17 /home/m18 /home/m19 /home/mlog

- 启动3个实例，且声明实例属于某复制集
- mongod  --dbpath /home/m17  --logpath /home/mlog/m17.log --fork --port 27017 --replSet rs2
- --smallfiles 启动的快
- mongod  --dbpath /home/m17  --logpath /home/mlog/m17.log --fork --port 27017 --replSet rs2 --smallfiles
- mongod  --dbpath /home/m18  --logpath /home/mlog/m18.log --fork --port 27018 --replSet rs2 --smallfiles
- mongod  --dbpath /home/m19  --logpath /home/mlog/m19.log --fork --port 27019 --replSet rs2 --smallfiles
- ps aux|grep mongo

- 配置
- 随意找一台连接
- use admin
- 配置文件
```javascript
var rsconf = {
    _id: 'rs2',
    members:[
        {
            _id: 0,
            host: '192.168.1.202:27017'
        },
        {
            _id: 1,
            host: '192.168.1.202:27018'
        },
        {
            _id: 2,
            host: '192.168.1.202:27019'
        }
    ]
}

printjson(rsconf)

```
-  根据配置做初始化
- rs.initiate(reconf);
- 查看状态 rs.status()
- id 为0 默认为primary

- mongo --port 27018


- 删除节点
- rs.remove('192.168.1.202:27019');
- 删除节点，客户端会重连

- 添加节点
- rs.add('192.168.1.202:27019') 删除之后，直接add，不行
- 需要更新rsconf 
- rs.reconfig(rsconf);
- rs.help()
- not master and slaveOk=flase code 13435
- rs.slaveOk(); 调用一下这个方法即可
- slave 默认不允许读，不允许写，slaveOk之后，可以允许读，写只能再primary上

- 关闭primary
- use admin
- db.shutdownServer()

- seconndry  
- rs.status()
- 18 ,19 机器正常
- 过一会  18 会变成primary状态

### 自动化脚本实现上面步骤
- start.sh
```bash
#!/bin/bash
IP=192.168.1.202
NA=rs2

if [$1 =='reset']; then
    # 清理现场
    pkill -9 mongo
    rm -rf /home/m*
    exit;
fi 

if [$1 == 'install']; then
    # 创建目录
    mkdir -p /hmoe/m17  /hmoe/m18 /hmoe/m19 /hmoe/mlog
    
    # 创建3个实例
    /usr/local/mongodb/bin/mongod --dbpath /home/m17 --logpath /home/mlog/m17.log --fork --port 27017 --smallfiles --replSet ${NA}
    /usr/local/mongodb/bin/mongod --dbpath /home/m18 --logpath /home/mlog/m18.log --fork --port 27018 --smallfiles --replSet ${NA}
    /usr/local/mongodb/bin/mongod --dbpath /home/m19 --logpath /home/mlog/m19.log --fork --port 27019 --smallfiles --replSet ${NA}
fi

if [$1 == 'repl']; then  
    # 启动客户端
    /usr/local/mongodb/bin/mongo <<EOF
    use admin
    var rsconf = {
        _id: '${NA}',
        members:[
            {
                _id: 0,
                host: '${IP}:27017'
            },
            {
                _id: 1,
                host: '${IP}:27018'
            },
            {
                _id: 2,
                host: '${IP}:27019'
            }
        ]
    }
    rs.initiate(rsconf);
EOF
exit;
fi
``` 
- sh start.sh

## shard分片
- mongos - 路由器
- configserver 配置服务器
- shard 分片
- configserver 不存贮真正的数据，存贮的mata信息，即某条数据在哪个片上的信息
- mongos 查询某条数据时，要先找configserver，询问得到该数据在哪个shard上
- 分片需要如下要素:
- 1. 要有N（N>=2）个mongod服务片节点
- 2. 要有configserver维护meta信息
- 3. 要启动mongos做路由
- 4. 要设定好数据的分片规则（configserve才能维护）


#### 创建目录
- mkdir -p /home/m17 /home/m18 /home/m20 /home/mlog
- mongod --dbpath /home/m17 --logpath /home/mlog/m17.log --fork --port 27017 --smallfiles
- mongod --dbpath /home/m18 --logpath /home/mlog/m18.log --fork --port 27018 --smallfiles
- mongod --dbpath /home/m20 --logpath /home/mlog/m20.log --fork --port 27020 --configsvr
- mongos --logpath /home/mlog/m30.log --port 30000 --configdb 192.168.1.202:27020
- ps aux|grep mongo
- mongo --port 30000
- sh.addShard('192.168.1.202:27017')
- sh.addShard('192.168.1.202:27018')
- sh.status();
- mongo --port 27017
- mongo --port 27018
- sh.enableSharding('shop');  该库启用分片
- partitioned: true
- sh.shardCollection('shop.goods', {goods_id:1});
- shop.goods 要分片的表名
- {goods_id:1} 要分片的键名 片键
- mongodb不是从单个文档的级别绝对平均的散落在各个片上
- 而是N个文档形成一个chunk，优先放在某个片上
- 当这片上的chunk比另一个chunk区别比较大事，chunk的数量相差>=3时，会把本片上的chunk移动到另一个片上
- 以chunk为单位，维护片之间的数据均衡
- 为什么插入10万条数据，才2个chunk？
- 答: 说明chunk比较大，默认64M
- use config
- db.settings.find()
- {_id: "chunkSize", "value": 64 }   64M
- db.settings.update({_id: "chunkSize" }, {$set:{ "value": 1}})
- db.settings.save({_id: "chunkSize", "value": 1 })

- 既然优先往某个片上插入，当chunk失衡时，再移动chunk，自然随着数据的增多
- shard的实例之间，有chunk来回移动的现象，这将带来什么问题？
- 答: 服务器之间io的增加
- 能否我定义一个规则，某N条数据形成一个chunk，预先分配M个chunk，M个chunk预先分配在不同的片上
- 以后的数据直接入各自预分配好的chunk，不再来回移动？
- 答: 能，手动预先分片

## 手动预先分片
- sh.shardCollection('shop.user', {userid: 1}) user表用userid做shard key
- sh.status()
```javascript
for(var i = i; i <= 40; i ++) {
    sh.splitAt('shop.user', {userid: i * 1000});
    // 预先在 1k 2k ... 40k这样的界限切好chunk
    // 虽然chunk是空的，这些chunk将会均匀的移动到各片上
}
```
- sh.status()
- 通过mongos添加user数据，数据会添加到预先分配好的chunk上，chunk就不会来回移动了

## replcation与shard分片结合使用

### replaction set 1

```bash
#!/bin/bash
IP=192.168.1.203
NA=rs3

if [$1 =='reset']; then
    # 清理现场
    pkill -9 mongo
    rm -rf /home/m*
    exit;
fi 

if [$1 == 'install']; then
    # 创建目录
    mkdir -p /hmoe/m17  /hmoe/m18 /hmoe/m19 /hmoe/mlog
    
    # 创建3个实例
    /usr/local/mongodb/bin/mongod --dbpath /home/m17 --logpath /home/mlog/m17.log --fork --port 27017 --smallfiles --replSet ${NA}
    /usr/local/mongodb/bin/mongod --dbpath /home/m18 --logpath /home/mlog/m18.log --fork --port 27018 --smallfiles --replSet ${NA}
    /usr/local/mongodb/bin/mongod --dbpath /home/m19 --logpath /home/mlog/m19.log --fork --port 27019 --smallfiles --replSet ${NA}
    exit;
fi   

if [$1 == 'repl']; then
    
    # 启动客户端
    /usr/local/mongodb/bin/mongo <<EOF
    use admin
    var rsconf = {
        _id: '${NA}',
        members:[
            {
                _id: 0,
                host: '${IP}:27017'
            },
            {
                _id: 1,
                host: '${IP}:27018'
            },
            {
                _id: 2,
                host: '${IP}:27019'
            }
        ]
    }
    rs.initiate(rsconf);
EOF
exit;
fi
```

### replaction set 2

```bash
#!/bin/bash
IP=192.168.1.204
NA=rs4

if [$1 =='reset']; then
    # 清理现场
    pkill -9 mongo
    rm -rf /home/m*
    exit;
fi 

if [$1 == 'install']; then
    # 创建目录
    mkdir -p /hmoe/m17  /hmoe/m18 /hmoe/m19 /hmoe/mlog
    
    # 创建3个实例
    /usr/local/mongodb/bin/mongod --dbpath /home/m17 --logpath /home/mlog/m17.log --fork --port 27017 --smallfiles --replSet ${NA}
    /usr/local/mongodb/bin/mongod --dbpath /home/m18 --logpath /home/mlog/m18.log --fork --port 27018 --smallfiles --replSet ${NA}
    /usr/local/mongodb/bin/mongod --dbpath /home/m19 --logpath /home/mlog/m19.log --fork --port 27019 --smallfiles --replSet ${NA}
    exit;
fi    

if [$1 == 'repl']; then
    # 启动客户端
    /usr/local/mongodb/bin/mongo <<EOF
    use admin
    var rsconf = {
        _id: '${NA}',
        members:[
            {
                _id: 0,
                host: '${IP}:27017'
            },
            {
                _id: 1,
                host: '${IP}:27018'
            },
            {
                _id: 2,
                host: '${IP}:27019'
            }
        ]
    }
    rs.initiate(rsconf);
EOF
exit;
fi
```
- 通过上面的两个脚本，完成2个复制集 rs3，rs4
- 目前还差mongos configserver


- configserver
- mongod --help|grep config
- mkdir -p home/m20 home/mlog
- mongod --dbpath /home/m20  --logpath /home/m20/m20.log --port 27020 --fork  --configsvr


- mongos 
- mongos --logpath /home/mlog/m30.log --port 30000 --configdb 192.168.1.202:27020 --fork
- configdb 应该是1个或者3个，1个是不够稳妥的


- 添加分片
- mongo 192.168.1.202:30000
- sh.addShard('rs3/192.168.1.203:27017');
- sh.addShard('rs4/192.168.1.204:27017');

- sh.status()

- sh.enableSharding('shop');
- sh.shardCollection('shop.user', {userid: 1})
- sh.splitAt('shop.user',{userid: 1000})
- sh.splitAt('shop.user',{userid: 2000})
- sh.splitAt('shop.user',{userid: 3000})
- sh.splitAt('shop.user',{userid: 4000})
- use shop
```javascript
for(var i = 1; i <= 4000; i++) {
    db.user.insert({userid: i, intro: 'i am lilei'});
}
```

- mongo    -primary
- use shop

- mongo --port 27018  secondy
- rs.slaveOK(); 允许读
























 





