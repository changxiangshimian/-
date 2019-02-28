# hadoop-HA(高可用)部署

> 软件配置简览

| 名称 | 版本 |
| - | - |
| centos|7.6.1810|
|java|1.8.0_201-b09|
|hadoop|2.9.2|
| zookeeper|3.4.10|

| 主机IP | 主机名 |
| - | - |
| 192.168.137.200 | hadoop01 |
| 192.168.137.201 | hadoop02 |
| 192.168.137.202 | hadoop03 |

> java自行安装

> 去zookeeper的镜像网站[下载](https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/)zookeeper

* 解压zookeeper压缩包并重命名为zookeeper
  * tar -zxvf zookeeper-3.4.10.tar.gz
  * mv zookeeper-3.4.10 zookeeper

* 复制并重命名`zookeeper/conf/zoo_sample.cfg`文件为`zoo.cfg`再进行修改
  * `cp zoo_sample.cfg zoo.cfg`
  * `vi zoo.cfg`

  ```

  server.1=hadoop01:28880:38880
  server.2=hadoop02:28880:38880
  server.3=hadoop03:28880:38880
  dataDir=/home/yetao_yang/zookeeper/data
  dataLogDir=/home/yetao_yang/zookeeper/datalog

  ```
* 在zookeeper目录下创建`data`和`datalog`两个目录

* 在`data0`目录下创建`myid`文件并添加内容1
  * `echo 1 > myid`


* 配置环境变量
  * `vi /etc/profile`
  * 添加如下配置
    * `source /etc/profile`使配置生效
      ```shell
      export ZOOKEEPER_HOME=/home/yetao_yang/zookeeper
      export PATH=$PATH:${JAVA_HOME}/bin:${ZOOKEEPER_HOME}/bin
      ```

* 把zookeeper文件夹上传到另外两个机器上
  * **`注：hadoop02节点上的myid内容为2，hadoop03节点上的myid内容为3。`**
  * 如果是第三方用户请注意文件的权限

* 分别启动三台机器的zookeeper
  * `zkServer.sh start`
    ```shell
    ZooKeeper JMX enabled by default
    Using config: /home/yetao_yang/zookeeper/bin/../conf/zoo.cfg
    Starting zookeeper ... STARTED
    ```

* 分别查看zookeeper的运行状态
  * `zkServer.sh status`
  * hadoop01

    ```shell
    ZooKeeper JMX enabled by default
    Using config: /home/yetao_yang/zookeeper/bin/../conf/zoo.cfg
    Mode: follower
    ```
  * hadoop02
    ```shell
    ZooKeeper JMX enabled by default
    Using config: /home/yetao_yang/zookeeper/bin/../conf/zoo.cfg
    Mode: leader
    ```
  * hadoop03
    ```shell
    ZooKeeper JMX enabled by default
    Using config: /home/yetao_yang/zookeeper/bin/../conf/zoo.cfg
    Mode: follower
    ```
  * **可以看到一个leader和两个follower，说明zookeeper安装成功。**
  * 分别停掉zookeeper
    * `zkServer.sh stop`

> 安装hadoop

* 下载[hadoop](https://hadoop.apache.org/releases.html)的安装包
  * 新建hadoop_HA的目录,并进入该目录
    * 新建`binary`文件夹,并把hadoop的安装包解压到此处
    * 在hadoop_HA文件夹分别建立 `journal` `var` `hdfs/data` `hdfs/name`的文件夹路径
      ```shell
      [yetao_yang@hadoop01 hadoop_HA]$ mkdir journal
      [yetao_yang@hadoop01 hadoop_HA]$ mkdir var
      [yetao_yang@hadoop01 hadoop_HA]$ mkdir hdfs
      [yetao_yang@hadoop01 hadoop_HA]$ mkdir hdfs/data
      [yetao_yang@hadoop01 hadoop_HA]$ mkdir hdfs/name
      ```

* 配置hadoop的环境变量
  * `vi /etc/profile`
    ```shell
    export HADOOP_HOME=/home/yetao_yang/hadoop_HA/binary/hadoop-2.9.2
    export PATH=$PATH:${JAVA_HOME}/bin:${ZOOKEEPER_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
    ```
  * `source /etc/profile`使配置生效


* **切换到hadoop的配置目录**
  * `/home/yetao_yang/hadoop_HA/binary/hadoop-2.9.2/etc/hadoop`

* 向`core-site.xml`文件的`configuration`标签添加内容
  ```xml
  <!-- 指定hdfs的nameservice为ns -->
  <property>
  <name>fs.defaultFS</name>
  <value>hdfs://ns</value>
  </property>
  <!--指定hadoop数据临时存放目录-->
  <property>
  <name>hadoop.tmp.dir</name>
  <value>/home/yetao_yang/hadoop_HA/tmp</value>
  </property>

  <property>
  <name>io.file.buffer.size</name>
  <value>4096</value>
  </property>
  <!--指定zookeeper地址-->
  <property>
  <name>ha.zookeeper.quorum</name>
  <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
  </property>
  ```

* 向`hdfs-site.xml`文件的`configuration`标签添加内容
  ```xml
  <!--指定hdfs的nameservice为ns，需要和core-site.xml中的保持一致 -->      
  <property>      
      <name>dfs.nameservices</name>      
      <value>ns</value>      
  </property>    
  <!-- ns下面有两个NameNode，分别是nn1，nn2 -->  
  <property>  
     <name>dfs.ha.namenodes.ns</name>  
     <value>nn1,nn2</value>  
  </property>  
  <!-- nn1的RPC通信地址 -->  
  <property>  
     <name>dfs.namenode.rpc-address.ns.nn1</name>  
     <value>hadoop01:9000</value>  
  </property>  
  <!-- nn1的http通信地址 -->  
  <property>  
      <name>dfs.namenode.http-address.ns.nn1</name>  
      <value>hadoop01:50070</value>  
  </property>  
  <!-- nn2的RPC通信地址 -->  
  <property>  
      <name>dfs.namenode.rpc-address.ns.nn2</name>  
      <value>hadoop02:9000</value>  
  </property>  
  <!-- nn2的http通信地址 -->  
  <property>  
      <name>dfs.namenode.http-address.ns.nn2</name>  
      <value>hadoop02:50070</value>  
  </property>  
  <!-- 指定NameNode的元数据在JournalNode上的存放位置 -->  
  <property>  
       <name>dfs.namenode.shared.edits.dir</name>  
       <value>qjournal://hadoop01:8485;hadoop02:8485;hadoop03:8485/ns</value>  
  </property>  
  <!-- 指定JournalNode在本地磁盘存放数据的位置 -->  
  <property>  
        <name>dfs.journalnode.edits.dir</name>  
        <value>/home/yetao_yang/hadoop_HA/journal</value>  
  </property>  
  <!-- 开启NameNode故障时自动切换 -->  
  <property>  
        <name>dfs.ha.automatic-failover.enabled</name>  
        <value>true</value>  
  </property>  
  <!-- 配置失败自动切换实现方式 -->  
  <property>  
          <name>dfs.client.failover.proxy.provider.ns</name>  
          <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>  
  </property>  
  <!-- 配置隔离机制 -->  
  <property>  
           <name>dfs.ha.fencing.methods</name>  
           <value>sshfence</value>  
  </property>  
  <!-- 使用隔离机制时需要ssh免登陆 -->  
  <property>  
          <name>dfs.ha.fencing.ssh.private-key-files</name>  
          <value>/home/yetao_yang/.ssh/id_rsa</value>  
  </property>  

  <property>      
      <name>dfs.namenode.name.dir</name>      
      <value>file:///home/yetao_yang/hadoop_HA/hdfs/name</value>      
  </property>      

  <property>      
      <name>dfs.datanode.data.dir</name>      
      <value>file:///home/yetao_yang/hadoop_HA/hdfs/data</value>      
  </property>      

  <property>      
     <name>dfs.replication</name>      
     <value>2</value>      
  </property>     
  <!-- 在NN和DN上开启WebHDFS (REST API)功能,不是必须 -->                                                                      
  <property>      
     <name>dfs.webhdfs.enabled</name>      
     <value>true</value>      
  </property>      
  ```
* 向`mapred-site.xml`文件的`configuration`标签添加内容

  ```xml
  <property>
　   <name>mapreduce.framework.name</name>
　   <value>yarn</value>
　</property>
  ```
* 向`yarn-site.xml`文件的`configuration`标签添加内容

  ```xml
  <!-- 指定nodemanager启动时加载server的方式为shuffle server -->
　<property>
　 <name>yarn.nodemanager.aux-services</name>
　 <value>mapreduce_shuffle</value>
　</property>
　<!-- 指定resourcemanager地址 -->
　<property>
　 <name>yarn.resourcemanager.hostname</name>
　 <value>hadoop03</value>
　</property>
  ```

  * 修改`hadoop-env.sh`文件
    * 修改JAVA_HOME的路径
      * `export JAVA_HOME=/usr/local/jdk1.8.0_201`
    * 修改hadoop配置文件路径
      * `export HADOOP_CONF_DIR=/home/yetao_yang/hadoop_HA/binary/hadoop-2.9.2/etc/hadoop`
    * 退出修改 并使环境变量生效
      * `source hadoop-env.sh`

* 把hadoop_HA拷贝到另外两台机器上并配置环境变量

* 分别在三台机器上启动zookeeper
  * `zkServer.sh start`

* 在节点一(hadoop01)上启动journalnode集群
  * `hadoop-daemons.sh start journalnode`
    ```shell
    [yetao_yang@hadoop01 ~]$ hadoop-daemons.sh start journalnode
    hadoop02: starting journalnode, logging to /home/yetao_yang/hadoop_HA/binary/hadoop-2.9.2/logs/hadoop-yetao_yang-journalnode-hadoop02.out
    hadoop01: starting journalnode, logging to /home/yetao_yang/hadoop_HA/binary/hadoop-2.9.2/logs/hadoop-yetao_yang-journalnode-hadoop01.out
    hadoop03: starting journalnode, logging to /home/yetao_yang/hadoop_HA/binary/hadoop-2.9.2/logs/hadoop-yetao_yang-journalnode-hadoop03.out
    ```

* 在节点一(hadoop01)上格式化zkfc
  ```shell
  hdfs zkfc -formatZK
  ```
