# 配置 FRP 实现内网穿透

## 1. FRP 的作用

利用处于内网或防火墙后的机器，对外网环境提供 http 或 https 服务。

对于 http, https 服务支持基于域名的虚拟主机，支持自定义域名绑定，使多个域名可以共用一个 80 端口。

利用处于内网或防火墙后的机器，对外网环境提供 tcp 和 udp 服务，例如在家里通过 ssh 访问处于公司内网环境内的主机。

## 2. 配置说明

### 2.1 实现功能

1. 外网通过 ssh 访问内网机器

2. 自定义绑定域名访问内网 web 服务

### 2.2 配置前准备

1. 公网服务器 1 台。公网服务器是内网服务器的代理，所有的请求都先走到它这里，然后被它转到内网服务器，从而实现外网间接访问内网服务器的共嗯那个。它扮演的是 FRP Server 的角色。

2. 内网服务器 1 台。内网服务器是逻辑上的真正的被访问服务， 它扮演的是 FRP client 的角色。

3. 内网服务器部署一个 web 服务，可以用 tomcat 模拟，这里就不演示了。

## 3. 安装 FRP

公网服务器与内网服务器都需要下载 FRP 进行安装，公网服务器通过配置，扮演 FRP Server 角色；内网服务器通过配置，扮演 FRP Client 角色。

从 github 上下载：[frp_0.15.1_linux_amd64.tar.gz](
https://github.com/fatedier/frp/releases/download/v0.15.1/frp_0.15.1_linux_amd64.tar.gz) 。

将下载的压缩包解压，例如:

```sh
tar xzvf frp_0.15.1_linux_amd64.tar.gz -C /usr/share/
```

这里主要关注 4 个文件：

| 角色                 | 关注           |
| :------------------- | :------------- |
| 服务端（公网服务器） | frps、frps.ini |
| 客户端（内网服务器） | frpc、frpc.ini |

## 4. 配置

### 4.1 服务端配置

FRP 服务端要运行在公网服务器上（因为它有公网 IP），它是内网服务器的代理。

配置服务端（公网服务器）时，可以先删掉 frpc 和 frpc.ini 这两个客户端配置文件（以免后续有混淆），然后再进行配置。

```sh
vi ./frps.ini
```

配置内容：

```ini
[common]
bind_port = 7000             # 与客户端绑定的进行通信的端口
vhost_http_port = 8081     # 访问客户端 web 服务自定义的端口号
```

保存然后启动服务

```
./frps -c ./frps.ini
```

这是前台启动，后台启动命令为

```sh
nohup ./frps -c ./frps.ini &
```


### 4.2 客户端配置

运行 FRP 的客户端的服务器是逻辑上的受访服务器，它才是用户真正所要访问、操作的对象。

配置客户端（内网服务器）时，可以先删掉 frps 和 frps.ini 两个服务端配置文件，以免混淆，然后再进行配置。

```sh
vi ./frpc.ini
```

配置内容：

```sh
[common]
server_addr = 120.56.37.48   # 公网服务器ip
server_port = 7000                # 与服务端 bind_port一致
 
# 公网通过ssh访问内部服务器（未验证）
[ssh]
type = tcp              #连接协议
local_ip = 192.168.3.48 #内网服务器ip
local_port = 22         #ssh默认端口号
remote_port = 6000      #自定义的访问内部ssh端口号
 
# 公网访问内部web服务器以http方式
[web]
type = http         #访问协议
local_port = 8081   #内网web服务的端口号
custom_domains = repo.iwi.com   # 所绑定的公网服务器域名或IP
```