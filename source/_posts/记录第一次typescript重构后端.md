---
title: 记录第一次typescript重构后端
date: 2019-08-05 18:00:00
tags: [数据分析工具项目, 项目复盘]
category: review
toc: true
comments: true
description: typescript使用心得
---
## 入门   
学习typescript最主要的是学习类型声明方法，官方文档很详细，适合作为字典，哪里不是特别理解的时候进行查阅。   
快速入门的教程网上也有很多了，个人参考的是[typescript入门教程]( https://ts.xcatliu.com/)，适合有一定Java/C语言基础的同学食用，介绍比较简单明了。   

## 代码检查eslint+prettier
项目中使用的是thinkjs 3.0框架，默认配置了`tslint`，由于typescript官方决定转向eslint，tslint不再维护，所以还是修改了一下配置，采用`eslint+prettier`来对代码进行检查。`tslint`的检查规则使用的是腾讯的[AlloyTeam Eslint规则](https://github.com/AlloyTeam/eslint-config-alloy)，同时加入了一些项目配置。
eslint主要负责语法检查，配置如下：   
```javascript   
module.exports = {
  extends: [
      'eslint-config-alloy/typescript',
      // "prettier" // eslint-config-prettier:  关闭一些不必要的或者是与prettier冲突的lint选项，由于AlloyTeam的typescript规则与标准规则有重复，无法起到应有的作用，建议根据prettier配置对
  ],
  globals: {
      // 这里填入你的项目需要的全局变量

      // 这里值为 false 表示这个全局变量不允许被重新赋值，比如：
      //
      // jQuery: false,
      // $: false
  },
  rules: {
      // 项目个性化设置
      'prefer-promise-reject-errors': 'warn',
      '@typescript-eslint/prefer-for-of': 'warn',
      'max-depth': 'warn',
      'complexity': 'warn',
      
      // 关闭与prettier重复的格式化配置
      "semi": 'off',
      '@typescript-eslint/semi': 'off',
      'indent': 'off',
      '@typescript-eslint/indent': 'off',
      // 配置部分格式化规则
      'array-bracket-spacing': ['error', 'always', { 
        "singleValue": false,
        "objectsInArrays": false,
        "arraysInArrays": false 
      }] // 数组对象内首尾空格
  }
};
```
prettier主要负责代码风格统一，配置如下：      
```javascript
module.exports = {
  "printWidth": 100, // 一行宽度
  "tabWidth": 2, // 缩进宽度
  "semi": false, // 末尾分号
  "quoteProps": "consistent", // object中的属性是否需要引号保持一致
  "trailingComma": "es5", // 在对象或数组最后一个元素后面是否加逗号（在ES5中加尾逗号）
  "bracketSpacing": true, // 对象括号内首尾空格
  "arrowParens": "always", // 箭头函数参数需括号
  "proseWrap": "preserve", // 代码超出是否要换行 preserve保留
  "endOfLine": "auto", // 结尾是 \n \r \n\r auto
}
```

## 流程化代码检查和格式化：husky
为了防止不规范的代码被commit并push到云端，增加了`precommit`钩子。   
首先需要安装husky：   
```
npm install husky --save-dev
```      

在`package.json`文件中添加脚本，在每次`git commit`前都会先prettier进行格式化，然后用eslint检查语法规范等，最后编译检查：   

```javascript
{
    "script": {
        "precommit": "npm run compile",
        "compile": "npm run fix:eslint && npm run fix:prettier && tsc --skipLibCheck",
        "fix:eslint": "eslint \"src/**/*.ts\" --ext .ts --fix",
        "fix:prettier": "prettier --config \".prettierrc.js\" --write \"src/**/*.ts\""
    }
}
```

## 总结
1. typescript是一个强类型语言，类型定义是最好的接口文档，对于多人开发/大型项目来说，能够很好的协助开发。   
2. 从javascript迁移到typescript很容易，即使没有进行任何定义，直接将.js文件重命名为.ts即可，使用JavaScript的大型项目可以很快的迁移到typescript，可以不需要立即重构全部旧代码。   
3. 使用typescript增加了编译阶段，可以在编译阶段发现大部分可能的错误，但是即便编译出错也可以生成JavaScript文件。   
4. typescript入门容易，但是想要用得好比较难。对于一些复杂需求的类型的定义，需要深入研究和探讨。   