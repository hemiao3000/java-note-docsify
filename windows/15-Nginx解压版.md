# <font color="orange">Nginx 解压版</font>

Nginx（"engine x"）是一个高性能的 HTTP 和反向代理服务器，也是一个 IMAP/POP3/SMTP 代理服务器。 Nginx 是由 Igor Sysoev 为俄罗斯访问量第二的 Rambler.ru 站点开发的，第一个公开版本 0.1.0 发布于 2004 年 10 月 4 日。其将源代码以类 BSD 许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。2011 年 6 月 1 日，nginx 1.0.4 发布。

## 1. 下载/安装

从官网 [*http://nginx.org/en/download.html*](http://nginx.org/en/download.html) 下载最新的文档版。例如：`nginx-1.18.0` 。

解压 *`nginx-1.18.0.zip`* 到本地目录。<small>按惯例，路径中不要有中文，最好不要有空格。</small>例如：`D:\ProgramFiles\nginx-1.18.0` 。

解压后，可到看到如下内容：

```
nginx-1.18.0
│── conf        配置文件目录
├── contrib
├── docs
├── html        类似 tomcat 的 webapps
├── logs        日志目录
├── temp
└── nginx.exe   启动程序
```


## 2. 启动

启动 Nginx 的方式有 2 种：

1.  直接双击 `nginx.exe`。双击后一个黑色的弹窗一闪而过。

2.  打开 cmd 命令窗口，切换到 nginx 解压目录下，输入命令 `nginx.exe` 或者 `start nginx` 。

检查 Nginx 是否启动成功的方式也有 2 种：

1.  直接在浏览器地址栏输入网址 [*http://localhost:80*](http://localhost:80) 。你会看到欢迎页面。

2.  在 cmd 命令窗口输入命令 `tasklist /fi "imagename eq nginx.exe"` 。你会看到类似如下页面：

    ```txt
    映像名称    PID     会话名      会话#   内存使用
    =========== ======= =========== ======= ============
    nginx.exe   17220   Console     8       7,148 K
    nginx.exe   17660   Console     8       7,508 K
    ```

## 3. 关闭

如果使用 cmd 命令窗口启动 nginx，关闭 cmd 窗口是**不能**结束 nginx 进程的。

可使用两种方法关闭 nginx：

1.  输入 `nginx` 命令：`nginx -s stop`<small>（快速停止 nginx）</small>或 `nginx -s quit`<small>（完整有序的停止 nginx）</small>。

2.  使用 `taskkill` 命令：   `taskkill /f /t /im nginx.exe` 。
