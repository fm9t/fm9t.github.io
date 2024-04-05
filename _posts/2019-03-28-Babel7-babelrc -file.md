---
layout:     post
title:      Babel7 babelrc文件设置和常用插件
subtitle:   
date:       2019-03-28
author:     ZSJ
header-img: img/post-bg-debug.png
catalog: true
tags:
    - babel 7
---
## Babel7 babelrc文件
    {
        "presets": [
            "@babel/preset-env",
            "@babel/preset-react"
        ],
        "plugins": [
        [
            "@babel/plugin-proposal-decorators",
            {
            "legacy": true
            }
        ],
        [
            "@babel/plugin-proposal-class-properties",
            {
                "loose": true
            }
        ],
        ["@babel/plugin-syntax-dynamic-import"],
        ["@babel/plugin-transform-runtime",
        {
            "regenerator": true
        }
        ]
        ]
    }

## presets部分:
### @babel/preset-env
    npm install --save-dev @babel/preset-env

不再需要设置ES版本，直接使用@babel/preset-env就可以自动编译

### @babel/preset-react
    npm install --save-dev @babel/preset-react

针对react使用

## plugins部分
### @babel/plugin-proposal-decorators
    npm install --save-dev @babel/plugin-proposal-decorators

针对ES7的decorator

### @babel/plugin-proposal-class-properties
    npm install --save-dev @babel/plugin-proposal-class-properties

针对ES7的类属性

### @babel/plugin-syntax-dynamic-import
    npm install --save-dev @babel/plugin-syntax-dynamic-import

针对动态导入

### @babel/plugin-transform-runtime {"regenerator": true}
    npm install --save-dev @babel/plugin-transform-runtime
    npm install --save @babel/runtime

针对async, await生成regenerator函数

