# **<font color="orange">node.js 和 NPM 的安装</font>**

## 1. yarn 的下载和安装 

> 我们安装 yarn 的目的是为了后续安装 vue 的调试工具 vue-devtools 作准备。因为，vue-devtools 要求使用 yarn 编译安装。

作为包管理器，npm 不是没有竞争对手。yarn 就是后起之秀。

早些时候 yarn 要通过 npm 来安装，不过在 yarn `1.x` 的时代 yarn 也提供了独立的安装程序来进行安装，但是到了 yarn `2.x` 时，yarn 官方又没有提供了<small>（过分）</small>。所以，我们还是通过 npm 来安装 yarn 。

> 为了减少初学时混用两个包管理工具所带来的混乱，我们『**仅在编译 vue-devtools 时使用 yarn**』，其他场景<small>（虽然可以，但是）</small>我们仍使用 npm 。

通过 npm 安装 yarn：

```sh
npm install -g yarn
```

安装完毕后，可直接执行 **yarn -v** 命令查看 yarn 的版本，以验证是否安装成功。


## 2. yarn 的配置（了解、自学）

- 查看 yarn 的中央仓库网址：

  ```sh
  yarn config get registry
  ```

- 修改 yarn 的中央仓库网址：

  ```sh
  yarn config set registry http://registry.npm.taobao.org/
  ```

- 查看 yarn 全局安装的命令的源码路径和二进制执行文件路径：

  ```sh
  yarn global dir
  yarn global bin
  ```

  注意，和 npm 一样，这两个路径和局部安装无关。


## 3. 命令总结

<dl>

<dt>npm install -g yarn</dt>
<dd>通过 npm 安装它的竞争对手 yarn 。</dd>

<dt>yarn -v</dt>
<dd>查看包管理器 yarn 的版本信息</dd>

<dt>yarn config get registry</dt>
<dd>查看包管理器 yarn 的网络仓库网址。</dd>

<dt>yarn config set registry http://registry.npm.taobao.org/</dt>
<dd>将包管理器 yarn 的网络仓库网址指定为淘宝镜像。</dd>

</dl>


## 4. yarn 命令和 NPM 命令对比（了解、自学）

| npm (v5)                              | yarn                          |
| :-                                    | :-                            |
| npm install	                          | yarn add                      |
| (N/A)	                                | yarn add --flat               |
| (N/A)	                                | yarn add --har                |
| npm install --no-package-lock         | yarn add --no-lockfile        |
| (N/A)	                                | yarn add --pure-lockfile      | 
| npm install [package] --save	        | yarn add [package]            |
| npm install [package] --save-dev	    | yarn add [package] --dev      | 
| (N/A)	                                | yarn add [package] --peer     |
| npm install [package] --save-optional	| yarn add [package] --optional |
| npm install [package] --save-exact	  | yarn add [package] --exact    |
| (N/A)	                                | yarn add [package] --tilde    |
| npm install [package] --global	      | yarn global add [package]     |
| npm update --global                   | yarn global upgrade | 
| npm rebuild	                          | yarn add --force              |
| npm uninstall [package]	              | yarn remove [package]         |
| npm cache clean	                      | yarn cache clean [package]    |
| rm -rf node_modules && npm install    | yarn upgrade                  |         
| npm version major                     | yarn version --major          |        
| npm version minor                     | yarn version --minor          |       
| npm version patch                     | yarn version --patch          |



## 5. vue-cli 创建的项目使用 Yarn（自学、了解）

vue-cli 创建的项目『**默认使用 NPM 作为包管理器**』。

> 如何知道我的 vue-cli 是使用 NPM 作为包管理器的？
> 
> 当你使用 **vue create xxx** 创建 vue 项目结束后看到如下信息，那么毫无疑问 vue-cli 使用的就是 NPM：
> 
> ccessfully created project xxx.<br>
> Get started with the following commands:<br>
> $ cd temp-vue-cli<br>
> $ npm run serve 看这里，看这里，看这里

如果有需要，你可以通过设置，去提前『**告知**』vue-cli 创建的 vue 项目使用 Yarn 作为包管理器。

无论是通过 NPM 还是通过 Yarn 全局安装 vue-cli，vue-cli 会在你的用户的『**家目录**』目录下创建一个名为『**.vuerc**』的文件<small>（一开始可能没有，在你第一次执行 vue create 命令后就会被 vue-cli 创建）</small>。在 Windows 环境中，这个文件在 **C:\Users\\<用户名>** 目录下。

> Windows 中有一个名为 **USERPROFILE** 的环境变量，它始终代表着当前用户的家目录。你可以直接输入 **%userprofile%** 就能在『文件资源管理器』中进入到你的家目录。


用编辑器打开这个文件，你会发现其内容是一个 JSON 格式数据：

```js
{
  "useTaobaoRegistry": false,
  "latestVersion": "4.4.6",
  "lastChecked": 1596095820450,
  "packageManager": "npm"   // 看这里，看这里，看这里。
}
```

很显然 **packageManager** 项就是用来设置 vue-cli 的包管理器的，将它从 *npm* 改为 **yarn** 。

> 另外，这个配置文件中还有一项 *useTaobaoRegistry* ，表示的是『**是否使用淘宝镜像源**』，你也可以将这一项改为 **true** 启用淘宝镜像源，以加快包的下载速度。

修改完成后，使用 **vue create xxx** 创建 vue 项目，创建过程结束后，你看到的将是：

```shell
successfully created project xxx.
Get started with the following commands:

$ cd temp-vue-cli
$ yarn serve
```

> <font color="red">**注意**</font>，这里的设置对命令行有效，而对 WebStorm 和 IDEA **无影响**，因为在它们里通过 vue-cli 创建 vue 项目，它们读取的配置文件是它们自己的配置文件，并不是这里的这个配置文件，是另外单独的配置。
