# <font color="orange">安装 CentOS-8 之后的常见配置</font>


## 1. 显示 IP

1.  打开、编辑相关配置文件

    ```sh
    vi /etc/sysconfig/network-scripts/ifcfg-ens192 
    ```

2.  修改最后一行

    ```
    ONBOOT=no
    改为
    ONBOOT=yes
    ```

3.  重启网络

    ```sh
    nmcli c reload ens192
    ```

## 2. 配置固定 IP 

1.  打开、编辑相关配置文件

    ```sh
    vi /etc/sysconfig/network-scripts/ifcfg-ens192 
    ```

2.  修改第 4 行

    ```
    BOOTPROTO=dhcp
    改为
    BOOTPROTO=static
    ```

3.  在最后添加

    ```
    IPADDR=192.172.0.140
    NETMASK=255.255.255.0
    GATEWAY=192.172.0.1
    DNS1=8.8.8.8
    PREFIX=24
    ```

4.  重启系统 

    <small>可以先做完下面的『修改 hostname』之后再一次性重启。</small>

    ```sh
    reboot
    ```

## 3. 修改 hostname 

1.  编辑、修改相关配置文件

    ```sh
    vi /etc/rc.d/rc.local
    ```

2.  添加相关修改 hostname 的命令

    ```sh
    nmcli g hostname 192.172.0.140
    ```
    
3.  赋予该文件可执行权限

    ```sh
    chmod a+x /etc/rc.d/rc.local
    ```

4.  重启系统

    ```sh
    reboot
    ```

## 4. 修改命令行提示 

1.  打开、编辑相关配置文件

    ```sh
    vi ~/.bashrc
    ```

2.  添加内容

    ```sh
    export PS1='[\u@\H \W]\$ '
    ```

3.  激活新配置

    ```sh
    source ~/.bashrc
    ```

> 考虑到你已经打开了 .bashrc ，你可以顺手就在这里加上两个 alias ，以后用得上。<small>（当然，前提是未来你要安装对应的软件，否则这里加上的内容是有问题的）。</small>
> ```sh
> alias vi='vim'
> alias dockerps='docker ps --format "table {{.Image}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"'
> ```

## 5. 修改阿里源 


1.  保存原有 .repo 配置<small>（可选）</small>

    略
    
2.  新建配置源文件

    ```sh
    vi /etc/yum.repos.d/CentOS-Base.repo
    ```

3.  填入如下内容

    ```sh
    # CentOS-Base.repo
    #
    # The mirror system uses the connecting IP address of the client and the
    # update status of each mirror to pick mirrors that are updated to and
    # geographically close to the client.  You should use this for CentOS updates
    # unless you are manually picking other mirrors.
    #
    # If the mirrorlist= does not work for you, as a fall back you can try the 
    # remarked out baseurl= line instead.
    #
    #
     
    [base]
    name=CentOS-$releasever - Base - mirrors.aliyun.com
    failovermethod=priority
    baseurl=https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
            http://mirrors.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/
            http://mirrors.cloud.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/
    gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
     
    #additional packages that may be useful
    [extras]
    name=CentOS-$releasever - Extras - mirrors.aliyun.com
    failovermethod=priority
    baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os/
            http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/os/
            http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/os/
    gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
     
    #additional packages that extend functionality of existing packages
    [centosplus]
    name=CentOS-$releasever - Plus - mirrors.aliyun.com
    failovermethod=priority
    baseurl=https://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/os/
            http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/os/
            http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/os/
    gpgcheck=1
    enabled=0
    gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
     
    [PowerTools]
    name=CentOS-$releasever - PowerTools - mirrors.aliyun.com
    failovermethod=priority
    baseurl=https://mirrors.aliyun.com/centos/$releasever/PowerTools/$basearch/os/
            http://mirrors.aliyuncs.com/centos/$releasever/PowerTools/$basearch/os/
            http://mirrors.cloud.aliyuncs.com/centos/$releasever/PowerTools/$basearch/os/
    gpgcheck=1
    enabled=0
    gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
    
    
    [AppStream]
    name=CentOS-$releasever - AppStream - mirrors.aliyun.com
    failovermethod=priority
    baseurl=https://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/
            http://mirrors.aliyuncs.com/centos/$releasever/AppStream/$basearch/os/
            http://mirrors.cloud.aliyuncs.com/centos/$releasever/AppStream/$basearch/os/
    gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
    ```

4.  验证、更新 yum 缓存

    ```
    yum makecache
    ```


## 6. 关闭防火墙 

```sh
systemctl stop firewalld
systemctl disable firewalld --now
```
## 7. 启用 NTP 服务器 

redhat/centos 7.x 默认使用的时间同步服务为 ntp 服务，但是从 redhat/centos 8 开始在官方的仓库中移除了 ntp 软件，换成默认的 chrony 进行时间同步的服务，虽然也可以通过添加第三方的源安装 ntp ，但是毕竟还是使用官方推荐的更好一些。

```sh
date 
yum install -y chrony
# vi /etc/chrony.conf
systemctl enable chronyd --now
timedatectl
ll /etc/localtime
```

如果安装系统时，你设置的时区不是东八区<small>（Asia/Shanghai）</small>，那么你还要做下面的操作：

```sh
rm /etc/localtime

ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

最后，再次通过 date 命令来验证。


## 8. 安装 lrzsz 命令 

在 linux 中 `rz` 和 `sz` 命令允许开发者与主机通过串口进行传递文件了。

- sz：将选定的文件发送（send）到本地机器
- rz：运行该命令会弹出一个文件选择窗口，从本地选择文件上传到 Linux 服务器

安装命令：

```sh
yum install -y lrzsz
```

从服务端发送文件到客户端：

```sh
sz filename
```

从客户端上传文件到服务端：

```sh
rz
```

## 9. 安装 vsftpd 搭建 ftp 服务器 

1.  安装 vsftpd

    ```sh
    yum install -y vsftpd
    ```

2.  创建存放资料的文件夹

    ```sh
    mkdir /srv/ftp
    ```

3.  编辑相关配置文件

    可以先将 vsftpd.conf 保存一份。

    ```sh
    vi /etc/vsftpd/vsftpd.conf
    ```

4.  配置匿名用户可访问

    > vi 技巧：
    > - 删除以 # 开头的注释：`g/^#/d`
    > - 删除空行： `g/^\s*$/d`
    > - 删除#后面的行： `g/#.*/d`



    ```sh
    # 匿名用户配置
    anonymous_enable=YES
    # anon_root=/srv/ftp 默认路径是 /var/ftp/pub
    # no_anon_password=YES
    anon_upload_enable=NO
    anon_mkdir_write_enable=NO

    # 本地用户配置
    local_enable=YES
    write_enable=YES
    #local_umask=022
    ```

5.  重启 vsftpd 服务

    ```sh
    systemctl restart vsftpd
    ```

6.  无法显示远程文件夹怎么办？

    出现『无法显示远程文件夹』的原因是 xftp 开启了被动模式，进去连接属性，点击选择，可以看到勾选了【使用被动模式】，将它取消掉即可。



## 10. 创建新用户

下面的示例以创建名为 `git` 的用户为例：

```bash
userdel -rf git         # 删除名为 git 的用户

useradd -m \
    -d /home/git \
    -s /bin/bash \
    git                 # 新建名为 git 的用户
                        
sudo passwd git         # 为新用户 git 设置密码
sudo vi /etc/sudoers    # 将新用户 git 添加成 sudoer

# compgen -u            # 查看系统所有用户
```

## 11. 添加用户至用户组

下面以添加 `当前用户` 到 docker 组为例子：

```bash
# 创建名为 docker 的用户组。
# 正常情况下，这条命令的结果会告诉你 docker 用户组已存在。
sudo groupadd docker 

# 将当前用户（即你所登录系统的账号）添加至 docker 用户组
sudo gpasswd -a $USER docker 

# 更新 docker 用户组
newgrp docker 
```

## 12. 安装 Oracle JDK8

-   从华为镜像网站下载历史版本，并解压

    ```bash
    # 安装 wget 工具
    yum install -y wget 

    # 下载
    wget https://repo.huaweicloud.com/java/jdk/8u201-b09/jdk-8u201-linux-x64.tar.gz

    # 解压
    mkdir -p /usr/java
    tar xvf jdk-8u201-linux-x64.tar.gz -C /usr/java

    # 创建链接
    ln -s /usr/java/jdk1.8.0_201/ /usr/java/jdk
    ```

-   删除已安装的 openJDK

    ```bash
    # 查看已安装的 openJDK
    yum list installed | grep java

    # 删除已安装的 openJDK
    yum remove -y java-1.8.0-openjdk*
    ```

-   配置、启用 Oracke JDK

    ```sh
    # 编辑全局配置文件 
    vi /etc/profile

    # 在文件末尾加入如下三行
    export JAVA_HOME=/usr/java/jdk
    export PATH=$PATH:$JAVA_HOME/bin
    export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

    # 启用新配置文件
    source /etc/profile

    # 验证
    java -version

    # 重新安装依赖于 java 的软件
    yum install -y maven ...
    ```

