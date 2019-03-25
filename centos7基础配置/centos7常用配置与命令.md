* 关闭防火墙
  * 停止防火墙
    * `systemctl stop firewalld.service`
  * 禁止防火墙开机启动
    * `systemctl disable firewalld.service`

<hr>

* 先让centos可以访问外网
  * `cd /etc/sysconfig/network-scripts`
  * `vi ifcfg-eth0`
* 修改ONBOOT为yes
  * `ONBOOT=yes`

<hr>

* 安装net-tools工具
  * `yum install net-tools`

<hr>

* 安装wget工具
  * `yum install wget`

<hr>

* 配置ssh`有些centos的版本默认是没有安装sshd服务的`
  * `yum list installed | grep openssh-server`检查一下是否安装了sshd
  * 如果没有则进行安装
    * `yum install openssh-server`
  * 进入`/etc/ssh`目录,编辑sshd_config文件

    ```shell
    Port 22
    ListenAddress 0.0.0.0
    ListenAddress ::
    PermitRootLogin yes
    PasswordAuthentication yes
    ```

  * 开启ssh服务
    * `service sshd start`
  * 确认是否开启sshd服务
    * `ps -e | grep sshd`
  * 开机启动sshd服务
    * `systemctl enable sshd`

<hr>

* hostname重命名
  * `hostnamectl set-hostname newhostname`

<hr>

> centos7之间设置免密登录

| ip地址 | 主机名 |
| - | - |
| 192.168.137.200|hadoop01|
| 192.168.137.201|hadoop02|
| 192.168.137.202|hadoop03|

* 依次修改三台机器`/etc/hosts`的文件
  * `vi /etc/hosts`

    ```shell
    192.168.137.200   hadoop01
    192.168.137.201   hadoop02
    192.168.137.202   hadoop03
    ```
* 分别修改三台机器的`/etc/ssh/sshd_config`文件
  * `vi /etc/ssh/sshd_config`

    ```shell
    RSAAuthentication yes #添加该配置
    PubkeyAuthentication yes  #消除注释
    AuthorizedKeysFile      .ssh/authorized_keys #指明key的存储路径 默认值不用改
    ```

* 三台机器之间进行ping测试
  * `ping hadoop02`
  ```shell
  $ ping hadoop02
  PING hadoop02 (192.168.137.201) 56(84) bytes of data.
  64 bytes from hadoop02 (192.168.137.201): icmp_seq=1 ttl=64 time=0.665 ms
  64 bytes from hadoop02 (192.168.137.201): icmp_seq=2 ttl=64 time=0.311 ms
  64 bytes from hadoop02 (192.168.137.201): icmp_seq=3 ttl=64 time=0.368 ms
  64 bytes from hadoop02 (192.168.137.201): icmp_seq=4 ttl=64 time=0.405 ms
  ```
* 给三台机器生成密匙文件
  * `ssh-keygen -t rsa`
  * 会在当前用户下的.ssh目录下生成两个文件`id_rsa`和`id_rsa.pub`
  * 在三台设备上创建`authorized_keys`文件<span style="color:red">在.ssh目录下创建</span>
    * 并在在每个authorized_keys文件把所有三台机器`id_rsa.pub`文件里的内容复制进去

* 分别给三台机器.ssh文件夹设置权限
  ```shell
  [root@hadoop01 ~]# chown test: ./.ssh
  [root@hadoop01 ~]# chown test: ./.ssh/*
  [yetao_yang@hadoop01 ~]# chmod 700 ./.ssh
  [yetao_yang@hadoop01 ~]# chmod 600 ./.ssh/*
  ```

* 测试配置是否成功

  ```shell
  [yetao_yang@hadoop01 .ssh]$ ssh hadoop02
  Last login: Fri Feb 15 13:32:21 2019 from hadoop03
  [yetao_yang@hadoop02 ~]$ exit
  登出
  Connection to hadoop02 closed.
  [yetao_yang@hadoop01 .ssh]$ ssh hadoop03
  Last login: Fri Feb 15 13:33:02 2019 from hadoop02
  [yetao_yang@hadoop03 ~]$ exit
  登出
  Connection to hadoop03 closed.
  [yetao_yang@hadoop01 .ssh]$
  ```
<hr>

* 使用tar工具进行打包
  * `tar -zcvf hadoop.tar.gz hadoop`
    * `把hadoop这个文件夹里面的东西全部打包成hadoop.tar.gz的压缩包`

<hr>

* centos之间传送文件
  * `scp /home/yetao_yang/hadoop.tar.gz yetao_yang@hadoop02:/home/yetao_yang`
    * 把`hadoop.tar.gz`文件传输到`yetao_yang@hadoop02`的`/home/yetao_yang`目录下

<hr>

* 新建文件
  * `touch filename`
