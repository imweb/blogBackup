title: IMWEB团队开发规范——脚手架
date: 2014-11-04 19:42:47
comments: true
author: Kael
about: https://github.com/yorts52
---
<ul class="dev-guide-box">
    <li><a href="/dev/index.html">1) 脚手架</a></li><li><a href="/dev/design.html">2) 视觉规范</a></li><li><a href="/dev/html.html">3) HTML规范</a></li><li><a href="/dev/js.html">4) Javascript规范</a></li><li><a href="/dev/css.html">5) CSS规范</a></li><li><a href="/dev/other.html">6) 其他规范</a></li>     
</ul>

# 一、脚手架


所有组件统一使用 [脚手架](https://github.com/imweb/generator-imweb) 进行开发
### 安装
```
// 安装 Yeoman
// 如果已经安装 请忽略
npm install -g yo

// 安装 脚手架 generator-imweb
npm install -g generator-imweb

// 初始化组件模块
yo imweb
```

### 组件的目录结构
```
dist： 压缩合并后的文件
doc ： 根据注释(基于jsdoc) 和 README 生成的组件说明文档
src ： 源文件所在目录
test: 测试用例所在目录  
gruntfile.js ： grunt配置文件
package.json : 开发时包的信息跟依赖
README.md： 模块的说明文档
bower.json： 发布时,作为bower包的信息
```

### 为项目创建一个主页和example
在该组件项目的 github 仓库中新建一个分支 `gh-pages`(推荐使用`GitHub Pages`根据`README`自动生成), 用于生成项目主页, 同时在该分支下提交一个组件使用的 `example`

可以使用以下方式生成页面:
```
  git checkout gh-pages // 切换到gh-pages分支
  git checkout master dist // 拉取master中dist目录代码
  git checkout master doc // 拉取master中doc目录代码
  git checkout master example // 拉取master中example目录代码
```
这样做可以生成 example 和 doc

### 将组件提交到组件库
当组件release并提供完善的说明文档、测试用例和example后, 需要汇总到组件库, 组件库暂定为 http://imweb.io/

参考: http://imweb.io/validator