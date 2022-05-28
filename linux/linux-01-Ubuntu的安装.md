# <font color="orange">Ubuntu 的安装</font>

## 1. 安装过程

### 1.1 欢迎界面界面

选择右侧按钮，进入 Ubuntu 系统的安装环节。

![ubuntu-01](./img/ubuntu-01.png)


### 1.2 选择键盘布局界面

使用默认值。我们的键盘的布局都是『美式布局』。

![ubuntu-02](./img/ubuntu-02.png)

### 1.3 默认安装的软件和升级 

![ubuntu-03](./img/ubuntu-03.png)

这个界面中有 1 对 radio 和 2 个 checkbox 进行选择。

-   **1** 和 **2** 是互斥的。表示安装过程中，是否要安装某些办公软件等非系统级的应用。

    这里，我们选择 **2**，最小化安装。最小化安装只会安装必要的工具软件和浏览器。
    
    至于其它的软件，如有需要，后续可以再安装。因此，就不在系统安装过程中进行安装。

- **3** 表示安装过程中是否要进行升级。

  系统盘中的某些软件的版本可能已经落后于当前中央仓库的最新版本。Ubuntu 可以在安装过程中进行联网升级，安装最新版本的软件。

  这里，我们取消这项功能。<small>（其实，最好是安装过程中断网）。</small>

- **4** 是否安装某些第三方的闭源软件。

  这里，我们选择需要。因为这里我们要用到无线网卡驱动。


### 1.4 分区 

![ubuntu-04](./img/ubuntu-04.png)

如果你的电脑上只有一块硬盘，那么你将在这个界面看到一个 `sda`，如果你是双硬盘那么你还会看到 `sdb`。如果有第三块，那自然还有 `sdc` 以此类推。

如果你的硬盘<small>（`sda`）</small>上装过系统，无论是 Windows 还是 Linux，那么你在 `sda` 下会看到 sda1、sda2、... 等。具体有多少个，取决于你之前的分区的数量。

你可以点击下方的 **`-`** 按钮，将这些不要的分区都删除<small>（在最终确认前，这里并没有真删除，还有返回余地）</small>。如果你将你原来的分区都删除干净了，那么就是类似上图形式，出现 `free space`，空余空间。

> 如果你是准备安装双系统，那么肯定就没有必要将 Windows 的各个分区都删光了。
> 
> 只要为安装 Ubuntu 准备出足够大的 freespace 即可。

![ubuntu-05](./img/ubuntu-05.png)


---

最简单的情况，你为你的 Ubuntu 系统仅需创建 2 个分区：`交换（swap）分区` + `一个主分区` 。


![ubuntu-07](./img/ubuntu-07.png)

1. 所创建的分区大小。默认值就是整个 free space 。
2. 分区类型，我们这里选择主分区。<small>因为，我们计划中也没有其它分区了，用不着『凑四』。</small>

3. 所创建分区从 free space 的哪头开始算起。默认是从头部开始。因为，我们这就要给分区，从头、从尾算起都是一回事。

4. 分区文件系统类型。默认值，不变。

5. 文件系统挂载点。<small>即，指定是哪个目录下的内容放到这个分区下。</small>

---


分区创建完毕后，就是如下样子。

![ubuntu-08](./img/ubuntu-08.png)


这里注意的是，这里我们在第一个硬盘（`sda`）<small>（的第一个扇区）</small>安装了 boot loader<small>（grub2）</small>。

> 毫无疑问，当电脑启动时，BIOS 将加载这里的 bootloader，而后再由这个 bootloader 加载操作系统镜像。

---



### 1.5 创建登录用户

输入用户名和密码，并且给你的电脑起个名字。

注意这的电脑名不宜过长，否则在终端显示上会占据太长空间影响感观。

![ubuntu-09](./img/ubuntu-09.png)

### 1.6 安装中，等待

![ubuntu-10](./img/ubuntu-10.png)

在安装过程中可能会出现 `retrieving ...` 和 `downloading ...` 这都是在联网下载某些软件。

由于默认连接的中央仓库的下载速度较慢，因此这里花费时间较常。所以，此时可以点击 `skip` 跳过这个环节。等待系统安装成功后，我们进入系统中，更改中央仓库网址之后，再下载。


## 2. 安装后的必要操作

### 2.1 修改『更新提示』的频次

Ubuntu 会自动连接软件源，检查本地安装的软件当前是否有更新的版本。如果有，它会给你提示信息。

不过，它的提示信息出现的过于频繁，如果你不需要这么频繁，或者甚至不需要它监测是否需要更新，你可以进行修改。

![after-install-01](./img/ubuntu-after-install-01.png)

![after-install-02](./img/ubuntu-after-install-02.png)

![after-install-03](./img/ubuntu-after-install-03.png)


### 2.2 更改软件源

Ubuntu 默认连接的软件源的下载速度比较慢。国内的一些大厂提供了国外软件园的国内镜像。我们可以将 Ubuntu 的软件源的网址改成这些国内的镜像。

Ubuntu 的软件源的地址的相关配置在 `/etc/apt/sources.list` 。

如果你对 vi 命令不熟悉，你可以使用 gedit 编辑它。

```bash
sudo gedit /etc/apt/sources.list
```

用下面的内容覆盖掉这个文件的原始内容：

```
deb http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
```

修改完毕后，你可执行 `sudo apt update` 命令，你会看到，Ubuntu 会去你所执行的软件源下载『**软件清单**』。<small>这样后续你就可以『离线查询』，查看软件园中有哪些软件及其版本。</small>

你可以执行下述命令安装 `net-tools` 软件包测试以下：

```bash
sudo apt install net-tools
```

net-tools 安装后的『效果』是你可使用 `ifconfig` 命令。

> 另外，`openssh-server` 可以装上，后续要使用。

## 3. 中文输入法

`settings` -> `Region & Language` -> `Manage Installed Languages`


```bash
sudo apt install ibus ibus-pinyin
```

『The End』
