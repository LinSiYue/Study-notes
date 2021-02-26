# Zookeeper

## 一、安装配置

```shell
tickTime=2000
dataDir=/Users/zdandljb/zookeeper/data
dataLogDir=/Users/zdandljb/zookeeper/dataLog
clientPort=2181 
initLimit=5 
syncLimit=2
server.1=server1:2888:3888 
server.2=server2:2888:3888 
server.3=server3:2888:3888
```

* tickTime：发送心跳的间隔时间，单位：毫秒 ;
* dataDir：zookeeper保存数据的目录；
* clientPort：客户端（或从机）连接zookeeper服务器的端口，zookeeper会监听这个端口，接收客户端的请求；
* initLimit：zookeeper服务端接收客户端（从机）初始化连接最长心跳间隔数，即最长连接时长为initLimit * tickTime，超过该连接时长，则表明该客户端连接失败；
* syncLimit：表示主机和从机间发送消息、请求和应答的最大心跳间隔；
* server.A=B：C：D：其 中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一 集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是 一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。



## 二、 Zxid

致使ZooKeeper节点状态改变的每一个操作都将使节点接收到一个Zxid格式的时间戳，并且这个时间戳全局有序。也就是说，也就是说，每个对节点的改变都将产生一个唯一的Zxid。如果Zxid1的值小于Zxid2的值，那么Zxid1所对应的事件发生在Zxid2所对应的事件之前。实际上，ZooKeeper的每个节点维护者三个Zxid值，为别为：cZxid、mZxid、pZxid。 

* cZxid：是节点创建时间对应的Zxid格式时间戳。
* mZxid：是节点修改时间对应的Zxid格式时间戳。
* pZxid：该节点的子节点列表最后一次被修改时的时间，子节点内容变更不会变更pZxid。
* ctime：节点创建时的时间；
* mtime：节点最后一次更新时的时间；
* cversion：子节点更改次数；
* dataVersion：节点数据更改次数；
* aclVersion：节点的ACL更改次数；
* ephemeralOwner：如果节点是临时节点，则表示创建该节点的会话SessionID；如果节点是持久节点，则该属性值为0；
* dataLength：数据内容的长度；
* numChildren：数据节点当前的子节点个数。

实现中Zxid是一个64为的数字，它高32位是epoch用来标识leader关系是否改变，每次一个leader被选出来，它都会有一个 新的epoch。低32位是个递增计数 

## 三、操作

### 1. 创建

```shell
#创建持久化节点：create <目录> <数据内容>
create /app "app"

#创建有序持久化节点：create -s <目录> <数据内容>
#创建完成提示：Created /a0000000001
#会将目录id加上序列，创建有序持久化节点可以为分布式环境创建一个唯一的id
create -s /a "a"

#创建临时节点，create -e <目录> <数据内容>
#退出会话，节点消失，重新登陆之后，查询不到
create -e /tmp "tmp"

#创建有序临时节点：create -s -e <目录> <数据内容>
#创建完成提示：Created /b0000000002
#可用来生成分布式锁
create -s -e /b "b"
```

### 2. 更新节点

```shell
#改变节点数据，set <目录> <节点数据>
#修改完成，版本号加1，即dataVersion从0变为1，没修改一次就加1。
set /a "abc"

#基于版本号修改，类似于乐观锁机制，set <目录> <节点数据> <版本号>
#当传入的版本号与当前节点版本号不一致时，Zookeeper拒绝本次修改。
set /a "abcd" 1

```

### 3. 删除节点

```shell
#删除节点
delete /a

#根据版本号删除节点
delete /a 1
```

### 4. 查看节点

```shell
#查看节点，值和属性
get /a

#查看节点属性
stat /a

#查看节点的子节点
ls /a

#查看节点的子节点和节点属性
ls2 /a
```

### 5. 授权

* c：创建
* d：删除
* r：读
* w：写
* a：授权

```shell
#为所有登陆的用户授权，setAcl <节点> world:anyone:<acl>
setAcl /a world:anyone:cdrwa

#为ip地址用户授权，setAcl <节点> ip:<ip地址>:<acl>,ip:<ip地址>:<acl>
setAcl /a ip:192.168.60.130:cdrwa

#auth授权模式
#先添加用户，addauth digest <用户名>:<密码>
addauth digest itcast:123456
setAcl /a auth:itcast:cdrwa

#digest授权，setAcl <节点> digest:<用户名>:<密码密文>:<acl>
#用户名和密码密文，echo -n <用户名>:<密码> | openssl dgst -binary -sha1 | openssl base64
echo -n itcast:123456 | openssl dgst -binary -sha1 | openssl base64
setAcl /a digest:itcast:qlzQzCLKhBQOghkooLvb+Mlwv4A=:cdrwa
addauth digest itcast:123456

#查询授权，getAcl <节点>
getAcl /a
```

## 四、代码接口使用

### 1. 连接

```java
public static void main(String[] args) {
    try{
        // 计数器对象
        CountDownLatch countDownLatch = new CountDownLatch(1);
        
        // arg1：服务器ip和端口
        // arg2：客户端和服务端之间的会话超时时间，单位毫秒
        // arg3：监视器对象
        Zookeeper zookeeper = new Zookeeper("192.168.60.130:2181", 5000, new Watcher(){
            @Override
            public void process(WatchedEvent event){
                // 监视到zookeeper连接状态
                if(event.getState() == Event.KeeperState.SyncConnected){
                    System.out.println("连接创建成功！");
                    countDownLatch.countDown();
                }
            }
        });
        
        // 主线程阻塞等待连接对象创建成功
        countDownLatch.await();
        
        System.out.println(zookeeper.getSessionId());
    } catch(Exception e) {
        e.printStackTrace();
    } finally{
        zookeeper.close();
    }
}
```

### 2. 创建

```java
// 同步方式
create(String path, byte[] data, List<ACL> acl, CreateMode createMode);

// 异步
create(String path, byte[] data, List<ACL> acl, CreateMode createMode,
      AsyncCallback.StringCallback callBack, Object ctx);
```

```java
// 连接...

// arg1：节点路径
// arg2：节点数据
// arg3：权限列表 world:anyone:cdrwa
// arg4：节点类型 持久化节点
zookeeper.create("/create/node1", "node1".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
// read权限，持久化有序节点
zookeeper.create("/create/node1", "node1".getBytes(), ZooDefs.Ids.READ_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
// 临时节点
zookeeper.create("/create/node1", "node1".getBytes(), ZooDefs.Ids.READ_ACL_UNSAFE, CreateMode.EPHEMERAL);

List<ACL> acls = new ArrayList<ACL>();
Id id = new Id("world", "anyone");
// ZooDefs.Perms.的属性有READ、ALL、WRITE、ADMIN、CREATE、DELETE
acls.add(new ACL(ZooDefs.Perms.READ, id));
zookeeper.create("/create/node1", "node1".getBytes(), acls, CreateMode.PERSISTENT);
```

### 3. 更改

```java
// 同步更新
// arg1：节点路径
// arg2：数据内容
// arg3：版本号 -1代表版本号不参与更新
Stat stat = zookeeper.setData("/set/node1", "node12".getBytes(), 1);
// 返回版本号
System.out.println(stat.getVersion());
```

```java
// 异步更新
zookeeper.setData("/set/node1", "node14".getBytes(), -1, new AsyncCallback.StatCallback(){
    @Override
    public void processResult(int rc, String path, Object ctx, Stat stat){
        // 0代表更新成功
        System.out.println(rc);
        // 路径
        System.out.println(path);
        //上下文参数
        System.out.println(ctx);
        //属性
        System.out.println(stat.getVersion());
    }
}, "I am Context");

Thread.sleep(10000);
System.out.println("结束");
```

### 4. 授权

```java
// 连接...

// 添加授权用户
zookeeper.addAuthInfo("digest1", "itcast1:1234561".getBytes());
```

### 5. 查看

```java
// arg1：目录
// arg2：watch
// arg3：stat
byte[] bs = zookeeper.getData("/node1", false, null);
```

### 6. 监听器

```java
// 创建连接会调用回调函数
@Override
public void process(WatchedEven event) {
	try{
        if(event.getType() == Event.EventType.None) {
            if(enent.getState() == Event.KeeperState.SyncConnected) {
                System.out.println("连接成功");
                // 阻塞
                countDownLatch.countDown();
            }else if(enent.getState() == Event.KeeperState.Disconnected) {
                System.out.println("断开连接");
            }else if(enent.getState() == Event.KeeperState.Expired) {
                System.out.println("连接超时");
            }else if(enent.getState() == Event.KeeperState.AuthFailed) {
                System.out.println("认证失败");
            }
        }
    } catch(Exception ex) {
        ex.printStackTrace();
    }
}
```



