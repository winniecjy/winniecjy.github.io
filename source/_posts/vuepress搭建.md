---
title: vuepress搭建
date: 2018-08-10 19:13:12
tags: 小项目
category: tools
toc: true
comments: true
description: vuepress搭建过程记录
---

## 介绍
1. `vuepress`支持html、md、vue。这些格式最终都会被转换成vue组件：`markdown -> html -> vue`。
 
## 搭建
1. 初始化：`npm init`
2. 安装vuepress  
*全局安装`-g`，在已有的项目中安装`-d`。*  
    ```bash
    # 全局安装
    npm install -g vuepress
 
    # 创建一个专门的docs目录存放md文件
    # mkdir docs
 
    # 创建一个 markdown 文件
    # echo '# Hello Vuepress' > docs/README.md
    echo '# Hello Vuepress' > README.md
 
    # 编写文档
    # npx vuepress dev docs
    npx vuepress dev
 
    # 构建
    # npx vuepress build docs
    npx vuepress build
    ```
    为了防止以后忘记命令，建议在`package.json`中添加`scripts`脚本。
    ```JavaScript
    {
        "scripts": {
            "dev": "vuepress dev docs",
            "build": "vuepress build docs"
        }
    }
    ```
3. 在上面两步之后就可以在本地查看部署效果了。现在还只是一个简单的markdown界面显示。做到像vuepress官网一样的界面还需要在`.vuepress`文件下添加配置文件`config.js`。以下代码简单的添加了标题、头部导航和侧边导航，具体可参照[官方文档](https://vuepress.docschina.org/config/)。  
    ```JavaScript
    module.exports = {
        base: '/gh-pages/', // github仓库名
        title: 'Hello VuePress', // 标题
        description: 'Justing do it!',
 
        // 顶部导航栏nav
        themeConfig: {
            nav: [
                {text: '指南', link: '/guide/'},
                {text: '配置参考', link: '/config/'},
                {text: '默认主题配置', link: '/default-theme-config/'},
                {text: 'Github', link: 'https://github.com/docschina/vuepress/'},
            ],
 
            // 左侧sidebar
            sidebar: [
                '/',
                'pageA.html',
                'pageB.html'
            ]
        },
    }
    ```
4. 为了让这个系统能够在外部访问，可以借助github。
    - 建立好一个github仓库，如：https://github.com/winniecjy/technical-docs，在`config.js`文件中添加`base: [仓库名]`  
    - 同步远程仓库到本地输出目录下`.vuepress/dist`  
        ```bash
        cd .vuepress/dist
        git init
        git pull -f
        ```
    - 将`dist`目录下的文件上传到github的gh-pages分支下`.vuepress/`  
        ```bash
        git add -A
        git commit -m 'deploy'
        git push -f git@github.com:winniecjy/technical-docs.git master:gh-pages
        ```
    - 在远程仓库`Settings -> Github Pages -> Source`设置为`gh-pages branch`，点击save。
    - 这样就大功告成啦，效果：[DEMO](https://winniecjy.github.io/technical-docs/)。
## 问题记录
1. 不使用`README.md`作为名字时，按照官方文档部署网站之后总是跳转到默认的404页面？ 
README.md 默认生成index.html为默认首页，不可更改。 
2. 注意文件重新生成之后需要重新部署个git仓库，并上传才能更新文档，因为dist文件夹都是删除重新生成的。`vuepress`比较适合不需要经常修改的静态文档。