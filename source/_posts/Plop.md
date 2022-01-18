---
title: 前端自动化构建工具——Plop
date: 2022-01-16 17:37:54
category:
    - 前端
tags:
    - Plop
---

一款小而美的脚手架工具，一般集成在项目中使用，用于自动化创建重复类型文件

## 一、安装
```shell
# 将plop模块作为项目开发依赖安装
yarn add plop --dev
```

## 二、使用
1. 在项目根目录下创建 plopfile.js
```js
module.exports = plop => {
    // component ---> 为运行命令
    plop.setGenerator('component', {
        // 描述
        description: 'create a component',
        // 询问，用户交互 (api参照：Inquirer.js)
        prompts: [
            {
                type: 'input',
                name: 'name',
                message: 'component name',
                default: 'MyComponent'
            }
        ],
        // 执行的动作
        actions: [
            {
                type: 'add', // 代表添加文件
                path: 'src/components/{{name}}/{{name}}.vue',
                templateFile: 'plop-templates/xxx.hbs'
            }
        ]
    })
}
```

2. 创建模版
一般是在项目根目录下创建 plop-templates 文件夹
内部再创建 hbs模板[（Handlebars模板语法）](https://handlebarsjs.com/) [中文版](https://handlebarsjs.com/zh/guide/)

```vue
<template>
    <div class="{{ name }}">{{ name }}</div>
</template>

<script>
export default {
    name: '{{ name }}',
}
</script>

<style lang="less" scoped>
.{{name}} {

}
</style>
```

3. 运行cli命令生成对应模版文件
```shell
yarn plop component
```

到此结束，自动在`src/components/`目录下生成了对应模版的文件
