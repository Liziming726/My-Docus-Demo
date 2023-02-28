---
title: Node-Crawler Study Share
date: 2022-10-28
tags: [文章分享]
cover: "https://imgbed.codingkelvin.fun/uPic/c9gF3x.png"
top_img: false
categories: [文章分享]
---

# 😁Hi hi , I am come back

> This time i will share my study ablout node-crawler ,it is a simple crawler tool for node.js it is very easy to use , This is the node-crawler office document 

[https://node-crawler.readthedocs.io/zh_CN/latest/]

Then I will share some questions I encountered during the use of node-crawler

## 🙂First

we shoule install node-crawler in the project

```js
npm install node-crawler
```

if the project use mysql , we should install mysql , and then create database and table

```js
npm install mysql
```

This is my project package.json

```js
"dependencies": {
    "crawler": "^1.2.1",
    "fastdfs-client": "^1.0.2",
    "moment": "^2.29.4",
    "moment-mini": "^2.24.0",
    "mysql": "^2.18.1",
    "request": "^2.88.2",
    "socks5-https-client": "^1.2.1"
  }
```

## 😃Second

Let's talk about node-crawler's features

- 支持分布式爬虫系统
- 服务端DOM和自动jQuery注入，使用Cheerio（默认）或JSDOM
- 可配置的连接池大小和重试次数
- Control rate limit
- 支持设置请求队列优先级
- forceUTF8模式可让爬虫处理字符集编码探测和转换

**For me the biggest feature is that use it so easy , cause i don't have python basic , so study Scray is not easy**

## 😆Third

Let's look at the code of the project , This demo is crawler the [https://kgo.ckcest.cn/] website. 

**Use jquery to parse the DOM , Then get the data we want ,and wirte the data to the mysql database , but the demo is not perfect , cause access the page don't have definite method , so I make the page number increasing use cycle , this is what i need improve .**

```js
var Crawler = require("crawler");
const Agent = require('socks5-https-client/lib/Agent');
var fs = require('fs');
var request = require('request');
var path = require('path');
var mysql = require('mysql');

var connection = mysql.createConnection({
    host     : 'localhost',
    user     : 'root',
    password : '123456',
    database : 'sciencedata'
});

connection.connect();
var max_page = 181;
//创建一个数组，用来存放所有的url
var url_list = [];
//将url存入数组
url_list.push(main_url);
//遍历出所有的url
for(var a=1;a<=max_page;a++){
    var main_url = 'https://www.ckcest.cn/entry/focus/list?index='+a;
    url_list.push(main_url);
    var c = new Crawler({
        maxConnections: 10, // 最大链接数 10
        retries: 5, // 失败重连5次
        // rateLimit: 200,
        // agentClass: Agent,
        callback: function (error, res, done) {
            if (error) {
                console.log(error);
            } else {
            }
            done();
        }
    });
    c.on('schedule', function (options) {
        // options.proxy = proxy_url;
    });
    
    c.queue([{
        uri: main_url,
        jQuery: true,
        // proxy: proxy_url,
        // limiter: proxy_url,
        callback: function (error, res, done) {
            if (error) {
                console.log(error);
            } else {
                var $ = res.$; // 这就可以想使用jQuery一个解析DOM了
                var len =  $(".focusNews-list>li").length;
                var str = "";
                for(var i=0;i<len;i++){
                    var lll = $(".focusNews-list>li").eq(i).find(".foucsNews-title a").text()
                    var souce = $(".focusNews-list>li").eq(i).find(".foucsNews-infor p").text()
                    var author = $(".focusNews-list>li").eq(i).find(".foucsNews-infor>span").text()
                    str+=lll+"\n"+souce+"\n"+author+"\n\n"
                }
                console.log(str)
                fs.writeFile('data.txt', str, {flag: 'a'}, function(err) {
                    if (err) {
                        console.log(err);
                    } else {
                        console.log('写入成功');
                    }
                });
                //将文件写入数据库
                var addSql = 'INSERT INTO science(title,source,author) VALUES(?,?,?)';
                var addSqlParams = [lll,souce,author];
                //增
                connection.query(addSql,addSqlParams,function (err, result) {
                    if(err){
                        console.log('[INSERT ERROR] - ',err.message);
                        return;
                    }
                    console.log('--------------------------INSERT----------------------------');
                    //console.log('INSERT ID:',result.insertId);
                    console.log('INSERT ID:',result);
                    console.log('-----------------------------------------------------------------\n\n');
                });
            }
            done();
        }
    }]);
}
```

## Bye everyone , Let's prograss together !🎉

