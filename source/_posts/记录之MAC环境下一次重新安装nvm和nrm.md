---
title: 记录之MAC环境下一次重新安装nvm和nrm
date: 2019-09-05 18:00:00
tags: [工具, 记录]
category: review
toc: true
comments: true
description: 本地已存在node版本时，希望加入nvm和nrm管理。   
---
## 简短介绍
nvm：node版本管理器，允许快速地在同一台设备上进行多个node版本之间切换。   
nrm：npm源管理器，允许快速地在 npm 源间切换。   

## 重新安装
> 前情提要：本地环境已有一个安装失败的nvm和单独安装的node。   
### 1. 删除本地的nvm和node版本
```
nvm：rm -rf ~/.nvm
node（brew安装）：brew uninstall node
node（官网安装）：sudo rm -rf /usr/local/{bin/{node,npm},lib/node_modules/npm,lib/node,share/man/*/node.*}
```
### 2. 删除`$NVM_DIR`
可检查以下目录   
```
~/.zshrc
~/.bash_profile
~/.bashrc
...
```
删除后重启终端，或通过如`source ~/.zshrc`操作，使得环境变量生效。检查是否删除成功可以通过：`echo $NVM_DIR`   
### 3. 安装nvm   
按照官网说明即可：https://github.com/nvm-sh/nvm
### 4. 设置默认node版本
通过官方的安装方法，此时已经可以使用nvm下载/切换不同版本的node，为了避免每次使用前都要通过`nvm use [version]`方法来设置版本，我们可以为其设置一个默认版本：   
```
nvm alias default v8.9.1
```
### 5. 使用nvm时的全局安装
由于使用了nvm，与系统默认的全局安装路径不一致，所以会发生全局安装的包找不到的情况，需要设置全局文件夹（注意版本号替换为当前使用的版本）：  
```
npm config set prefix $NVM_DIR/versions/node/v8.9.1
```
此时，不同版本的node之前全局安装包不共享，每一个版本下的全局文件夹路径都需要单独设置。