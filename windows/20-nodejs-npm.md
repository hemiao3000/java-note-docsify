# **<font color="orange">node.js 和 NPM 的安装</font>**

## 01. node.js 和 npm 的下载安装 

在 [官网](https://nodejs.org/) 下载 node 的长期支持<small>（LTS）</small>版本。例如，当前<small>（2020-11-5）</small>最新的长期支持版是 **node-v14.15.0** 。

> node 的安装包中已包含 npm 。无需再找 npm 的安装包。

安装结束后，直接在命令行<small>（cmd）</small>中执行 **node -v** 和 **npm -v**，会看到它们俩的版本信息，即证明安装成功。

> node.js 相当于 JDK，如果你不安装 node.js 那么你无法在你的 PC 上运行 JavaScript 代码。<small>（你最多只能在你的浏览器中运行 JavaScript 代码）</small>。这种情况下，你连『**在终端上输出显示 `hello world`** 』这样的小功能都无法实现。
> 
> npm 相当于 Maven，确切地说相当于是 Maven 的部分功能<small>（包管理）</small>。通过它，你可以下载你的项目中所需要的包，并管理包和你的项目之间、包和包之间的依赖关系。


## 02. npm 的全局安装和局部安装 

虽然我们可以用 Maven 来类比 NPM，但是 NPM 中有『**全局安装**』和『**本地安装**』的区别，而 Maven 中并没有这样的概念。

因此，npm 的本地仓库就细分为：『**本地全局安装仓库**』和『**本地局部安装仓库**』。

> 全局安装仓库和本地局部安装仓库一共是 **1 + N** 个。

在 JavaScript / node.js 领域我们通过 npm 来下载、管理包。其实，这里的『**包**』分为两种：

- 一种包本质上是命令、工具、软件。我们下载这个包的目的是为了系统<small>（的命令行）</small>中能『多』出来一个命令使用。例如 **vue-cli** 和 **cnpm** 。

  对于这种本质上是『命令』的包，我们要通过全局安装将它们下载、安装到『本地的全局仓库』中。

- 另一种包，就和 Java 领域中使用 Maven 下载的包一样，是就普普通通的包，是在项目中要引用、使用到的 JavaScript 代码、CSS 代码等。例如，**vue** 和 **bootstrap** 。

  对于这种本质上是『代码』包，我们要通过局部安装将他们下载、安装到『本地的局部仓库』中。
  

## 03. 中央仓库和淘宝镜像

无论是全局安装还是局部安装，我们使用 npm 时都会从中央仓库下载包。

你可通过下述命令查看到这个默认的中央仓库的网址：

```sh
npm config get registry

# 不出意外显示的应该是："https://registry.npmjs.org/"
```

但是从默认的中央仓库下载包是速度很感人。通常，我们会利用淘宝提供的国内镜像来提升下载速度。

- 方案一：在不改配置的情况下，每当你使用 **npm install -g** 命令时，多附带一个参数，强行指定当前的下载操作从你指定的淘宝的仓库下载。

  ```sh
  npm install -g xxx --registry=https://registry.npm.taobao.org
  ```

  当然，这种方案用起来比较麻烦。

- 方案二：修改配置，替换掉默认的中央仓库网址。让 npm 命令下载时总是从淘宝的仓库下载。

  ```sh
  npm config set registry https://registry.npm.taobao.org/
  ```

  **推荐这个方案。**

- 方案三：下载、使用淘宝提供的 cnpm 。

  这个方案本质上是『**方案一**』的简化版。

  ```sh
  npm install -g cnpm --registry=https://registry.npm.taobao.org
  ```

  安装 xxx 包时，使用命令：

  ```sh
  cnpm install -g xxx
  ```

  <small>有人反应这种方式下载的包的依赖上有时可能会有问题。所以优先还是考虑方案二。</small>


## 04. npm 全局安装包及使用

当你使用 **npm install xxx -g** 命令去全局安装 xxx 库时，npm 会从网络上的『**中央仓库**』下载包到你的『**本地全局仓库**』。

> **-g** 是 **--global** 的简写，表示当前的安装操作是全局安装。

如果你没有改动过你的 npm 的设置，你的本地全局仓库应该在 **C:\Users\\<用户名>\AppData\Roaming\npm\node_modules** 。

> 这里需要注意的是 **AppData** 是一个隐藏文件夹。<small>你需要想办法看到它。</small>

这个路径你可以通过 **npm root -g** 命令查看得到它。<small>另外，在Windows 的文件资源管理器中你可以通过 **%APPDATA%\npm\node_modules** 可以直接进入到这个目录，*%appdata%* 大小写不敏感。</small>

全局安装后，**%APPDATA%\npm\node_modules** 目录下会出现你所安装的 xxx 库的源码，而上一级目录，即 **%APPDATA%\npm** 则会出现这个库的可执行程序。

由于 **npm** 是 node.js 的默认的包管理工具。因此，通过 **npm** 全局安装的包无需额外的配置，你就可在命令行中使用它。

> 在 Windows 上全局安装包<small>（命令）</small>，你可以通过 where 命令查看该命令在哪。

## 05. 设置 cache 和 prefix

有些教程会在 npm<small>（和 cnpm）</small>安装之后，去修改它的 **cache** 和 **prefix** 这两个配置项。这两个配置项指向了两个本地的目录。

对于初学者和非专业前端开发人员而言，不需要去改变这两个目录的默认位置。它们不影响 npm 的使用。

> 初学者无论安装什么软件、工具。**可改可不改的配置一律不要改**。
>
> 不过在 Ubuntu 上，由于路径的权限问题，为了图省事，可以考虑修改这两个路径：
> 
>  npm config set cache $HOME/node<br>
>  npm config set prefix $HOME/node

## 06. npm 局部安装包及使用

与全局安装对应的是局部安装。之前反复提到过，全局安装安装的包本质上是命令、是工具、是软件。而局部安装的包，才是我们开发中要用到的“那种”包。

如果在安装命令 **npm install xxx** 中没有出现 **--global**<small>（或其简写 **-g**）</small>，那么就意味着是局部安装。即，相当于安装命令自带了 **--save-dev** 。

局部安装都是安装在『**当前项目中**』的。即，通常在使用局部安装命令时，你是在项目所在的位置执行安装命令。

局部安装结束后，你的当前项目的 **node_modules** 目录下会出现一个你所安装的包的文件夹，其中有该包的内容。至此，你在你的项目中也就可以引用、使用这个包。

另外，你所安装的这个包的信息，会记录在前项目的 **package.json** 文件中。

---

> <small>『**全局安装**』的包无法在项目中使用，因为，我们反复强调过：全局安装的包不是我们项目中要用到的“那种包”。
> 
> 全局安装安装的是逻辑上的命令，局部安装安装的是逻辑上的工具包。</small>

局部安装的包在项目中可以通过 **.require** 的方式使用。例如：

```js
var path = require('path');
var webapck = require('webpack');
var xxx = require('xxx');
```

## 07. 两种安装方式的由来（了解、自学）

早期的 npm 的包的安装方式只有全局安装，并没有局部安装的概念，所有下载的包都放在了全局的仓库中。<small>这和 Java 的 Maven 的行为很像。</small>

但是 npm 在这里有个小问题：**全局安装没有办法下载/安装同一个包的多个版本。**

以 Java 的 Maven 做类比，在 Maven 中如果你下载了同一个包的多个版本，那么在本地仓库中它们的目录结构会是如下形式：

```
xxx
│── 1.0
│   └── ...
│── 1.1
│   └── ...
│── 1.2
│   └── ...
└── 等
```

但是 npm 没有这版本这层文件夹，即，npm 全局安装所下载的总是 xxx 包的『**当前最新版本**』。形如：

```
xxx
└── ...
```

这就导致了一个问题：你的 PC 上的两个不同的项目使用的如果是『**同一个包的两个不同版本**』，那么这里完全没有办法满足这个需求。

为此，npm 才提出了『**局部安装**』的概念，各个项目各下各的、各装各的、各用各的，各人玩各人的，互不干扰。

这样以前的安装方式<small>（即，全局安装）</small>的用途就越来越弱，最终慢慢退化成用来安装命令类的包。


## 08. 命令总结

<dl>

<dt>node -v</dt>
<dd>查看所安装的 node 环境的版本信息</dd>

<dt>npm -v</dt>
<dd>查看所安装的包管理器 npm 的版本信息</dd>

<dt>npm config get registry</dt>
<dd>查看 npm 所使用的网络仓库网址</dd>

<dt>npm config set registry https://registry.npm.taobao.org/</dt>
<dd>更换 npm 所使用的网络仓库网址。换为淘宝提供的镜像网址。</dd>

<dt>npm install xxx -g</dt>
<dd>从网络仓库下载并全局安装 xxx 包。</dd>
<dd><small>npm install xxx --global</small></dd>

<dt>%APPDATA%\npm\node_modules</dt>
<dd>npm 全局安装的包的安装路径</dd>

<dt>%APPDATA%\npm</dt>
<dd>npm 全局安装的包的可执行文件的所在路径</dd>

<dt>npm uninstall -g vue-cli</dt>
<dd>npm 全局卸载 vue-cli<small>（1.x、2.x 版本）</small></dd>

<dt>npm install -g @vue/cli</dt>
<dd>npm 全局安装 vue-cli<small>（3.x 版本）</small></dd>

<dt>npm install xxx</dt>
<dd>从网络仓库下载并局部安装 xxx 包<small>（到当前项目）</small>。</dd>
<dd><small>npm install xxx --save-dev</small></dd>

<dt>npm install -g yarn</dt>
<dd>通过 npm 安装它的竞争对手 yarn 。</dd>

<dt>yarn -v</dt>
<dd>查看包管理器 yarn 的版本信息</dd>

<dt>yarn config get registry</dt>
<dd>查看包管理器 yarn 的网络仓库网址。</dd>

<dt>yarn config set registry http://registry.npm.taobao.org/</dt>
<dd>将包管理器 yarn 的网络仓库网址指定为淘宝镜像。</dd>

</dl>



