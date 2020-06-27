---
title: veu&spring项目开发
tags: JAVA
---

[toc]

# vue安装

## [官网](https://cn.vuejs.org/v2/guide/installation.html)

> 推荐CLI,安装配置此处不赘述

# Vue项目创建

命令行输入

```bash
vue -ui
```

在打开的浏览器里创建vue项目,使用`IDEA`打开

# 登录界面开发

在`src/component`文件夹创建`Login.vue`文件,创建简单的登录界面

```vue
<template>
    <div>
        用户名:<input type="text" v-model="loginForm.username" placeholder="请输入用户名"/>
        <br><br>
        密码： <input type="password" v-model="loginForm.password" placeholder="请输入密码"/>
        <br><br>
        <button v-on:click="login">登录</button>
    </div>
</template>

<script>

    export default {
        name: 'Login',
        data () {
            return {
                loginForm: {
                    username: '',
                    password: ''
                },
                responseResult: []
            }
        },
        methods: {
            login () {
                this.axios
                    .post('/login', {
                        username: this.loginForm.username,
                        password: this.loginForm.password
                    })
                    .then(successResponse => {
                        if (successResponse.data.code === 200) {
                            this.$router.replace({path: '/index'})
                        }
                    })
                    .catch(failResponse => {
                    })
            }
        }
    }
</script>
```

> 其中的样式自己添加

# 创建主页

在`src/component`文件夹下创建`MyIndex.vue`文件

此处暂时不忙写内容,后续添加

# 创建后端项目

打开`IDEA`创建`spring web`项目,具体过程:

在 IDEA 中新建项目，选择` Spring Initializr`，点击 Next,完成项目组等配置后,点击next,然后选择`web`->`web`,最后完成配置.