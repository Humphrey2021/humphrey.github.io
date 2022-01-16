---
title: Yeoman入门--实现属于自己的脚手架
date: 2022-01-15 13:37:52
category:
    - 前端
tags:
    - 前端自动化构建
    - yeoman
    - 脚手架
---

## 脚手架工具开发
> 脚手架的本质作用就是为了创建项目的基本结构，提供项目的规范和约定
**一切技术都是为了解决问题而存在的。**
我们在不同的项目中，可以会存在很多相同的地方，如：组织结构、开发范式、模块依赖、工具配置、基础代码。为了方便，避免做太多重复操作，所以就出现了脚手架。

目前常用的脚手架工具
react   ==>  creat-reate-app
vue     ==>  vue-cli
angular ==>  angular-cli

但今天我们要说说另外一种【`Yeoman`】，这是一款通用型的脚手架

## [Yeoman](https://yeoman.io/)
< THE WEB'S SCAFFOLDING TOOL FOR MODERN WEBAPPS >
可以搭配不同的`Generator`去生成不同的项目

### 基本使用
- 第一步当然是安装了，但在安装之前需要检查是否安装了`node`和`npm`
```shell
node -v
npm -v
# 因为比较习惯用 yarn，后续代码也是使用yarn去代替npm
yarn -v
# 安装 yeoman
yarn global add yo
```
前面说了`yeoman`需要搭配不同的`Generator`去生成项目
- 安装`generator`模块
这里以`node`为例
```shell
yarn global add generator-node
```

- 运行`generator`生成项目
```shell
# 运行generator就是使用yo命令后面跟着 去掉generator-后的内容
yo node
# 在命令行选择填写对应的内容，即可生成对应的项目
```
到此我们就生成了一个node项目

### 使用Yeoman步骤
因为使用`yeoman`可以生成任何项目，所以我们在使用之前需要明确的步骤
1. 明确项目需求
2. 找到合适的`generator`
3. 全局范围安装找到的`generator`
4. 通过`yo`运行对应的`generator`
5. 通过命令行交互填写选项
6. 生成所需要的项目结构

### 自定义自己的Generator
<基于Yeoman搭建自己的脚手架>
创建 Generator 模块 <generator模块本质上就是一个npm模块>
generator模块规定名称格式为 generator-<name>
```shell
mkdir generator-self # 创建generator模块文件
cd generator-self # 进入这个文件夹
yarn init # 生成package.json文件
yarn add yeoman-generator # 安装yeoman的generator模块
mkdir generators # 创建生成器目录
mkdir generators/app # 创建默认生成器目录
touch generators/app/index.js # 创建默认生成器实现的文件
```
至此，生成了一个目录结构
```m
├── generators          // 生成器目录
│   └── app             // 默认生成器目录
│       └── index.js    // 默认生成器实现
├── node_modules
└── package.json
```
- 最主要的文件就是 generators/app/index.js

此文件作为 Generator 的核心入口文件
需要导出一个继承自 Yeoman Generator 的类型
Yeoman Generator 在工作时会自动调用我们在此类型中定义的一些生命周期方法
我们在这些方法中可以通过调用父类提供的一些工具方法来实现一些功能，例如文件写入

```js
const Generator = require('yeoman-generator')

module.exports = class extends Generator {
    writing () {
        // Yeoman 自动在生成文件阶段调用此方法
        // 我们这里尝试往项目目录中写入文件
        this.fs.write()
        this.destinationPath('temp.txt')
        Math.random().toString()
    }
}
```
编写完成之后，需要将此模块链接到全局，将其变为全局模块包
```shell
yarn link
```
此时我们换一个文件夹
```shell
yo self # self 为刚刚写的生成器的名字
```
这就是yeoman生成器最基本的开发过程

#### 根据模版去创建文件
一个项目的基础结构是有很多个文件的，如果都手动去创建文件写入内容，那将会变得特别繁琐，所以我们可以将基础结构直接写好，然后通过模版的方式去创建整个项目结构,这样可以大大的提高效率
```shell
mkdir generators/app/templates # 创建模版文件夹
# 这个文件夹内部的文件就将会作为模版文件去解析

# 例：创建一个 foo.txt 模版
touch generators/app/templates/foo.txt
```
在 foo.txt 中
```txt
内部可以使用 EJS 模版标记输出数据
例如: <%= title %>
其他的 EJS 语法也支持
<% if (success) { %>
啦啦啦
<% } %>
注意：如果内部有属于自己的模版语法，则需要再加一个%
<%= BASE_URL %> ===> <%%= BASE_URL %>
```
回到 app/index.js 中
```js
... // 省略
writing () {
    // 通过模版方式写入文件到目标目录
    // 模版文件路径
    const tmpl = this.templatePath('foo.txt')
    // 输出目标路径
    const outPut = this.destinationPath('foo.txt')
    // 模版数据上下文
    const context = { title: 'hello', success: true }
    this.fs.copyTpl(tmpl, outPut, context)
}
...
```
#### 接收用户输入数据
通过命令行交互的方式，获取用户的输入。提示模块由 [Inquirer.js](https://github.com/SBoudrias/Inquirer.js) 提供。可以参考[官方文档查询API](https://github.com/SBoudrias/Inquirer.js)
```js
// 依然是在 app/index.js 文件中
prompting() {
    // Yeoman 在询问用户环节会自动调用此方法
    // 在此方法中可以调用父类的 prompt() 方法发出对用户的命令行询问
    // 参数为一个数组
    return this.prompt([
        {
            type: 'input',
            name: 'name',
            message: 'Your project name',
            default: this.appname // appname 为项目生成目录名称
        }
    ]).then(answers => {
        // answers => { name: 'user input value' }
        this.answers = answers
    })
}
writing () {
    ...
    const context = this.answers
}
```
到此，我们就已经可以去实现一个自己的`Generator`了

#### 发布 Generator
当写好了自己的`Generator`后,一般会将此项目托管到远程的仓库中。
```shell
# 创建排除文件
echo node_modules > .gitignore
git init
git add .
git commit -m 'init'
git remote add origin https://github.com/user/generator-self
git push -u origin master

yarn publish # 发布
# 国内一般会使用淘宝镜像源取代官方的镜像，所以可能会报错
yarn publish --registry=https://registry.yarnpkg.com
# 在npm官网就可以查看到这个模块
# 随后就可以全局安装使用了
```
如果想让自己的 Generator 在 yeoman 官方的仓库列表中也出现的话，可以在项目中添加一个 yeoman-generator 的关键词，这个时候yeoman官方会发现你的项目。
