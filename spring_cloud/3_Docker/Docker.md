# Docker

[![](./img/docker1.png)](https://www.docker.com/)

>解决了软件开发,测试和运维的问题
>可以`简单`的理解为一个轻量级的虚拟机
>把一个程序放在独立的环境里面运行

### 安装 Centos

* 删除非官方的Docker软件包
  * `sudo yum -y remove docker docker-common container-selinux`
* 删除docker-selinux与官方docker-engine软件包冲突的软件包
  * `sudo yum -y remove docker-selinux`
* 安装yum-utils，它提供了yum-config-manager实用程序
  * `sudo yum install -y yum-utils`
* 使用以下命令设置稳定的存储库

```shell
sudo yum-config-manager \
    --add-repo \
    https://docs.docker.com/v1.13/engine/installation/linux/repo_files/centos/docker.repo
```

* 更新yum软件包索引
  * `sudo yum makecache fast`
* 安装Docker
  * 安装最新版
    * `sudo yum -y install docker-engine`
  * 安装特定版本的Docker
    * `yum list docker-engine.x86_64  --showduplicates |sort -r`
    ```shell
    docker-engine.x86_64  1.13.0-1.el7                               docker-main
    docker-engine.x86_64  1.12.5-1.el7                               docker-main   
    docker-engine.x86_64  1.12.4-1.el7                               docker-main   
    docker-engine.x86_64  1.12.3-1.el7                               docker-main  
    ```
    * 选择一个特定的版本进行安装
      * `sudo yum -y install docker-engine-<VERSION_STRING>`
* 启动Docker
  * `sudo systemctl start docker`
* 运行hello-world 映像验证安装是否正确
  * `sudo docker run hello-world`
* 升级Docker
  * `sudo yum makecache fast`
  * 然后选择要安装的新版本



### 安装 Windows

* 如果是window10 专业版的话

  [Docker](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)

* 如果是非专业版或window10以下的话
  [Docker Toolbox](https://download.docker.com/win/stable/DockerToolbox.exe)

* 然后下一步就可以了

* 验证
  * 在cmd输入`docker -v`
  * 安装正常的话会打印 docker的版本

* 在桌面上找到`Docker Quickstart Terminal`快捷方式并启动
  * 它会提示找不到`boot2docker.iso`文件正在从github上下载到你的某个目录`从提示信息可以看出来下载哪里去了`
    * 在安装Docker Toolbox目录下找到`boot2docker.iso`文件复制到那个到上面的目录
  * 重新点击`Docker Quickstart Terminal`快捷方式
  ![](./img/docker2.png)



**<font color='red'>>>未完待续<<</font>**
