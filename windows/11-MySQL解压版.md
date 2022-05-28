# <font color="orange">MySQL 压缩版</font>

## 0. 写在前面的话

有些软件对于安装路径有一定的要求，例如：路径中不能有空格，不能有中英文，不能有特殊符号，等等。

为了避免不必要的麻烦，也懒得一一辨别踩坑，我们人为作出『**统一的约定**』：

- 安装版的软件，一律安装在软件的默认安装路径，不要去改变它。<small>默认在哪就是哪，别动。</small>

- 解压版的软件，一律安装在：**D:\ProgramFiles** 。这是一个没中文、没空格的路径！

MySQL 的安装<small>（无论是安装版，还是解压版）</small>，都依赖于所安装的电脑上要安装 `Visual C++ Redistributable`。[下载链接](https://www.microsoft.com/en-us/download/details.aspx?id=40784)

## 1. 下载、解压 mysql

下载，略。

解压到 **D:\ProgramFiles** 目录下。


## 2. 添加缺省的配置文件

mysql 解压的文件中默认没有两个关键性的东西：**my.ini** 文件和 **data** 目录。

在 mysql 的解压目录/安装目录下的根目录下创建 **my.ini** 文件，并添置以下内容： 

```properties
[client]
port=3306

[mysql]
default-character-set=utf8

[mysqld]
port=3306
basedir=D:\ProgramFiles\mysql-5.7.24-winx64
datadir=D:\ProgramFiles\mysql-5.7.24-winx64\data
character_set_server=utf8mb4
default-storage-engine=INNODB
```

各配置项解释如下：

```properties
## CLIENT SECTION
## ---------------------------------------------------------
[client]
port=3306                       设置 mysql 客户端连接服务端时默认使用的端口

[mysql]
default-character-set=utf8      设置 mysql 客户端默认使用的字符集

## SERVER SECTION
## ---------------------------------------------------------
[mysqld]
port=3306                       mysql 服务端监听/占用的端口

basedir=D:\ProgramFiles\mysql-5.7.24-winx64         mysql 的安装路径

datadir=D:\ProgramFiles\mysql-5.7.24-winx64\data    mysql 数据库数据文件所在路径

character_set_server=utf8mb4    新建 schema 或 table 时默认使用的字符集

default-storage-engine=INNODB   新建 table 时默认使用的引擎
```

> <small>注意，此处的 basedir 和 datadir 要符合实际情况中 MySQL 的解压路径。</small>


## 3. 初始化

> 由于后续要使用到 MySQL 解压目录下的 *`bin`* 目录下的各种命令，方便起见，可以新建 *`MYSQL_HOME`* 环境变量，并将其下的 *`bin`* 目录添加到环境变量 *`path`* 中。

在终端中输入，*`mysqld --initialize`*，回车后可见在 mysql 解压目录中生成了 data 文件夹。

<font color="red">**注意**</font>：在 data 文件夹中，有一个 `.err` 文件，其中的内容包含了（最后一行）初始的 root 用户的密码，类似如下：

```sh
2019-01-09T08:11:05.817097Z 1 [Note] A temporary password is generated for root@localhost: ?ds:i+)4C/wo
```

上例中，其中 `?ds:i+)4C/wo` 就是 root 用户的初始密码。



## 4. 启动 MySQL 服务端

执行命令启动 MySQL 服务端


```shell
d:\ProgramFiles\mysql-5.7.24-winx64\bin\mysqld
```

注意，执行的是 *`myslqd`*，而不是 *`mysql`* 。它们两个是不同的东西，不要执行错了。



## 5. 修改密码

启动 mysql，通过 *`mysql -uroot -p`* 进行连接/登录<small>（使用默认的初始密码）</small>，然后可通过以下命令进行修改 root 用户密码：



```sql
set password for root@localhost=password('新密码');
```



最后输入：**show variables like "character%";** 确保字符集编码是 UTF8 编码。



## 6. 确认 mysql 编码

mysql 的默认编码是 latin1，可通过下面命令查看确认：

```sql
show variables like 'character%';
```

如果显示结果中出现 latin1，则通过修改配置文件 *`my.ini`* 进行配置：

```
[mysqld]
## 新建 schema 或 table 时默认使用的字符集
character_set_server=utf8
```


## 7. 可选操作: 注册成 Windows 服务<small>（非必须）</small>

在终端中输入：*`mysqld --install`*，安装 mysql 服务。

另外，反向的卸载/删除操作中，使用命令 *`mysqld --remove`*，卸载前记得要停止 mysqld 。

可以通过以下命令启动/停止 mysql 服务端。

```
net start mysql
net stop mysql
```
