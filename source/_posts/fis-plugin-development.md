---
layout: post
title: FIS插件开发：基于CMD规范的半自动打包插件
date: 2014-10-9 23:23
comments: true
author: Aaron
about: http://github.com/ousiri
tags: 
    - 前端 
    - 工具
---

#FIS插件开发: 基于CMD规范的半自动打包插件

###插件目标

这个插件是用于解决fis打包中需要手动配置文件依赖的问题. 其目标是希望通过简单配置, 达到类似r.js的作用.

####背景
[FIS](http://fis.baidu.com/)是百度FIS团队开发的一个自动构建工具. 该团队提供一个基于CMD规范的模块加载器Mod.js.

####现有方案分析
1. fis的[fis-postpackager-simple](https://github.com/hefangshi/fis-postpackager-simple)插件提供all in one的加载方式, 把所有js文件打包成一个. 但这样子不利于多页面间复用通用的JS文件.
2. fis自带的pack设置参数允许开发者手动设置通用文件, 当再使用fis-postpackager-simple的时候会把这些通用文件剔除掉. 但这个方案需要开发者手动管理文件的依赖, 当通用文件需要增加或减少依赖的时候, 容易配置错误.

###FIS的运行过程
- 编译阶段
    - parser
    - preprocessor
    - postprocessor
    - lint
    - test
- 打包阶段
    - **prepackager** <- 这个是目标
    - *packager* <- 这个是重点
    - spriter
    - postpackager

了解fis的运行过程有助于我们进行插件开发. 编译阶段是对单文件进行处理. 而打包则是对多文件. 这里我们要开发打包工具, 肯定是在打包阶段执行的插件. 
具体分析, packager阶段是根据pack参数进行处理, 例如把多个文件合成一个, 并且生成map.json, 用于跟踪文件之间的关系. 所以我们的插件可以执行在prepackager阶段, 自动生成pack参数, 这样就符合fis的工作流了.

###插件的输入输出

####输入
前面提到, 我们希望可以做得像r.js这样方便的自定义一个文件, 因此插件的参数设计为:
```js
fis.config.set('settings.prepackager.ousiri-spm-build', {
    lib: ['zepto', 'common'],
    pkg: ['courseList']
});
```
`lib`表示这个是多页面用到的库, 后面业务文件假如要依赖, 就不会把库文件打包到业务文件里面了, 相当于业务文件的exclude.
`pkg`表示这个是独立的文件, 会exclude掉lib的依赖.

`注:` 这里的`zepto`并不是zepto.js文件的路径, 而是该文件的id. fis里面的文件描述分为id和uri, 转换规则是自定义在roadmap的path参数里面的. 这里使用的是fis-pure的转换规则, 即
1. 一级同名组件，可以引用短路径，比如`modules/jquery/jquery.js`, id为`jquery`. 忽略modules目录
2. modules目录下的其它文件使用文件名, 例如`modules/common/Tip.js`, id为`common/Tip`
3. 其它目录即使用uri, 例如`lib/mod.js`

####输出
插件输出pack参数, pack参数的格式是这样的:
```js
{
    'pkg/zepto.min.js': [
        'zepto',
    ],
    'pkg/common.min.js': [
        'tools',
        'tools.ext'
    ]
});
```

###插件输入参数
```js
module.exports = function(ret, conf, settings, opt) {
    
}
```
打包阶段的插件导出一个函数, 接受以下参数:
- **ret 所有文件信息**
- conf 打包信息 -- 一般用不到
- **settings 插件用户配置信息**
- opt 命令行参数

我们需要用到的是ret和settings参数. 

#### ret
ret是一个object, 其中我们要用到ids这个key. 其大致结构是:
```js
{
    ids: {
        common: {
            requires: [
                'common/tools.ext',
                'zepto/touch.zepto',
                'common/db',
                'common/mm.report',
                'modules/common/common.scss'
            ],
            extras: {
                async: [
                    'qqapi'
                ]
            },
            isJsLike: true
            ...
        },
        ...
    }
}
```
这里每个文件的requires是我们依赖分析的依据, extras里面会存放延后加载的文件.
`注:` 这里有`modules/common/common.scss`这个文件, 是因为fis会自动把目录中和js同名的css文件也加进依赖里面. 

###依赖分析
依赖分析比较简单, 这里使用广度优先的算法. 具体代码可以参考[这里](https://github.com/ousiri/fis-prepackager-ousiri-spm-build/blob/master/index.js)

