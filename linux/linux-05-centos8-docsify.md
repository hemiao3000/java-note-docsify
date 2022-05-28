# <font color="orange">在 CentOS-8 上运行 docsify</font>


## 1. 安装 CentOS-8 系统

略

## 2. 配置 CentOS-8 系统

略

## 3. 安装、配置 nodejs（及 npm）

最简单的方案是通过 yum 安装 nodejs ，不过 yum 源中的版本较老<small>（10.24.0）</small>，当前（2021-04-01）最新的长期支持版是 14.16.0 。因此，如果你想安装最新版本的 nodejs，那么就不能从原始的 yum 源安装。

0.  删除已安装的 nodejs

    ```sh
    yum remove -y pure nodejs
    ```

1.  添加 node.js Yum 源


    ```sh
    curl -sL https://rpm.nodesource.com/setup_14.x | sudo -E bash -
    ```

2.  安装 nodejs  

    ```js
    yum install -y nodejs
    ```
    
2.  验证    

    ```js
    node -v
    
    npm -v
    ```

3.  设置 npm 的淘宝源，并验证

    ```sh
    npm config set registry https://registry.npm.taobao.org/
    
    npm info underscore
    ```
    
    
## 4. 安装 docsify-cli，并运行

```sh
npm install -g docsify-cli

docsify serve ...
```
 
