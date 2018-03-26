---
title: database
category: database
date: 2018-1-19
tags: [数据库,课设]
---

数据库课设笔记

<!-- more -->

# 前言

这学期学习了 **数据库系统** 这门课程, 学习到了关于数据库的知识, 因为课堂学习的是SQL的知识, 做实验也是用到SQL Server, 所以这次课设就选择了MySQL来写,首先当然是知识准备咯.MySQL的语法跟SQL的语法一样.在课堂学习的时候就学习了SQL的语法，先用phpStudy的adminer把数据库及表建立起来，学习Node.js连接MySQL，前端利用ElementUI来快速完成页面的设计.

# 设计过程
因为选题选的是学生信息管理系统,而做实验的时候就有类似的学生信息系统,所以参考实验并加以修改,总共有 Student,Department,Speciality,Class,Course,SC,User7个表.
其他就不着重讲了.课程设计报告有详细的。

# 服务端笔记
Express采用**body-parser**,一个HTTP请求体解析中间件,使用这个模块可以解析JSON、Raw、文本、URL-encoded格式的请求体.
``` javascript
const bodyParser = require('body-parser');
const express = require('express');
const app = express();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: false}))
```
引入mysql
``` javascript
var mysql = require('mysql');
var conn mysql.createConnection(models.mysql);  //建立MySQL连接
```
设计API接口 (使用express的router)
``` javascript
var router = express.Router(); //引入 router

router.post('/selectAll', (req, res) => {
    var params = req.body;  // 前端传过来的参数，进行提取
    var sql = `SELECT * from ${params.table}`; //使用模版字符串写SQL语句
    //进行对数据库的查询
    conn.query(sql, function(err, result) {
        if(err) {
            console.log(err);
            res.send(500,{error: '出错: '+err.sqlMessage});
        }
        if(result) {
            res.json({
                code: 200,
                data: result,
                msg: '查询成功'
            })
        }
    })
})


```
当查询出错时，一直不了解怎么返回提示信息给前端，后面观察服务端的node进程，`console.log(err)` 打印了一些出错信息，有个字段为sqlMessage，所以就把这个`err.sqlMessage`直接传给前端，有时候出错信息比较复杂，这个因为时间原因没有对这个信息再进行加工，就直接服务端数据库报什么错就直接传给前端了，有点不妥。 

``` javascript
if(result) {
    if(result.affectedRows>0){
        res.json({
            code: 200,
            data: result,
            msg: '删除成功'
        })
    }
    if(result.affectedRows ===0){
        res.json({
            code: 200,
            data: result,
            msg: '操作没有影响，删除的数据原本就不存在~'
        })
    }
}
            
``` 

设计*修改*，*删除*API时，对数据库的操作成功会返回个result对象，其有个affectedRows字段，代表操作后所影响到的表的数据的行数，当`affectedRows > 0`说明数据有受到影响，而其为0时，则操作并没有对数据库产生修改，从侧面说明欲修改、删除的数据本就不存在，返回`操作没有影响到数据库，数据不存在`给前端。

# 前端笔记
前端采用Vue-cli 和 vue-router

整个的管理页面只有中间的具体操作的部分是根据管理模块的类型来按需显示的，又因为管理模块的类型已经在`side-menu`分得很清楚，点击菜单栏上的某个管理模块，中间的部分就显示特定的内容。一开始没有什么思路，后面查阅到`ElementUI`的`el-menu`支持路由模式，点击菜单栏的item，路由会跳转到特定的此item的index值，如下
```html
<el-menu
router >
<el-submenu index="1">
		<template slot="title">
		<i class="el-icon-setting"></i>
		<span>学生管理</span>
		</template>
		<el-menu-item index="add_student" v-if="userType === 2">新增学生</el-menu-item>
		<el-menu-item index="search_student" >查询学生信息</el-menu-item>
		<el-menu-item index="update_student" v-if="userType === 2">修改学生信息</el-menu-item>
        <el-menu-item index="delete_student" v-if="userType === 2">删除学生</el-menu-item>
	</el-submenu>
<el-menu>
```
所以就在`management`组件，也就是管理页面上检测路由变化来进行中间部分特定模块的显示

```
// management.vue
export default {
    ...
    watch: {
        $route(to, from){
            this.index = to.params.msg;
			this.table = this.index.split('_')[1];
        }
    }
}
```
index 可以获取到跳转后的路由，似`add_student` ， 从而 `this.table`的值为 student，所以中间的数据部分显示student表。

# 心得

