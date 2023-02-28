---
title: Axios source Drop
date: 2023-01-24
tags: [文章分享]
cover: ""
top_img: false
categories: [文章分享]
---

# 🎉Happy CNY!

> today we are going to talk about axios

![](http://rp1kb3lu6.bkt.gdipper.com/1.png)

> 图床实在是个问题，

## What is Axios?

> axios的入口文件

```js
module.exports = require('./lib/axios');
```


> 接下来让我们一步一步来看看axios的源码

1. 导入模块

```js
//引入工具
var utils = require('./utils');
//引入绑定函数  创建函数
var bind = require('./helpers/bind');// 创建函数的
//引入 Axios 主文件
var Axios = require('./core/Axios');
// 引入合并配置的函数
var mergeConfig = require('./core/mergeConfig');
// 导入默认配置
var defaults = require('./defaults');
```

2. 通过createInstance来创建Axios实例对象

```js
/**
 * Create an instance of Axios
 * 创建一个 Axios 的实例对象
 * @param {Object} defaultConfig The default config for the instance
 * @return {Axios} A new instance of Axios
 */
function createInstance(defaultConfig) {
    //创建一个实例对象 context 可以调用 get  post put delete request
    var context = new Axios(defaultConfig);// context 不能当函数使用  
    // 将 request 方法的 this 指向 context 并返回新函数  instance 可以用作函数使用, 且返回的是一个 promise 对象
    var instance = bind(Axios.prototype.request, context);// instance 与 Axios.prototype.request 代码一致
    // instance({method:'get'});  instance.get() .post()
    // Copy axios.prototype to instance
    // 将 Axios.prototype 和实例对象的方法都添加到 instance 函数身上
    utils.extend(instance, Axios.prototype, context);// instance.get instance.post ...
    // instance()  instance.get()
    // 将实例对象的方法和属性扩展到 instance 函数身上
    utils.extend(instance, context);

    return instance;
}
// axios.interceptors

// Create the default instance to be exported
// 通过配置创建 axios 函数
var axios = createInstance(defaults);

// Expose Axios class to allow class inheritance
// axios 添加 Axios 属性, 属性值为构造函数对象  axios.CancelToken = CancelToken    new axios.Axios();
axios.Axios = Axios;
```

3. 如何创建？

![](http://rp1kb3lu6.bkt.gdipper.com/3.png)

首先实例化 进入Axios.js

![](http://rp1kb3lu6.bkt.gdipper.com/4.png)


传递参数defaults

![](http://rp1kb3lu6.bkt.gdipper.com/5.png)

defaults的值

![](http://rp1kb3lu6.bkt.gdipper.com/6.png)

添加拦截器的属性  此时运行到这一步已经存在defaults和interceptors两个属性

interceptors : ![](http://rp1kb3lu6.bkt.gdipper.com/7.png)

```js 
defaults :
//默认配置
        axios.defaults.method = 'get'
        //设置默认请求类型为get
        axios.defaults.baseURL = 'http://localhost:3000'
        //设置默认请求地址
        axios.defaults.params = {id:100}
        //设置默认请求参数
        axios.defaults.timeout = 1000
        //设置默认请求超时时间
```

向Axios原型对象上加方法

![](http://rp1kb3lu6.bkt.gdipper.com/8.png)

为context添加方法  （函数原型对象上面加上了这些方法， 所以可以在实例对象身上使用这些方法）

![](http://rp1kb3lu6.bkt.gdipper.com/9.png)

```js 
// 将 request 方法的 this 指向 context 并返回新函数  instance 可以用作函数使用, 且返回的是一个 promise 对象
    var instance = bind(Axios.prototype.request, context);// instance 与 Axios.prototype.request 代码一致
```

将目标对象身上的方法复制

```js
// Copy axios.prototype to instance
    // 将 Axios.prototype 和实例对象的方法都添加到 instance 函数身上
    utils.extend(instance, Axios.prototype, context);// instance.get instance.post ...
```

至此再回看instance，不再是单纯的函数，身上添加了许多方法，utils华丽转身

![](http://rp1kb3lu6.bkt.gdipper.com/10.png)

将默认配置对象与拦截器加入instance

![](http://rp1kb3lu6.bkt.gdipper.com/11.png)

既可以当函数，也可以当方法，原理与jQuery相似，先造一个函数出来，再在函数身上添加对应的方法与属性
```js
    axios.defaults.method = 'get'
    axios({
        url:'/posts',
    }).then(res => {
        console.log(res);
    })
    $.
```

以下操作都是锦上添花的，向axios上添加属性,完善结构

```js
path: /axios/lib/axios.js
// Factory for creating new instances
// 工厂函数  用来返回创建实例对象的函数
axios.create = function create(instanceConfig) {
    return createInstance(mergeConfig(axios.defaults, instanceConfig));
};

// Expose Cancel & CancelToken
axios.Cancel = require('./cancel/Cancel');
axios.CancelToken = require('./cancel/CancelToken');
axios.isCancel = require('./cancel/isCancel');

// Expose all/spread
axios.all = function all(promises) {
    return Promise.all(promises);
};
axios.spread = require('./helpers/spread');

module.exports = axios;

// Allow use of default import syntax in TypeScript
module.exports.default = axios;
```

## 模拟实现axios创建过程

```js
<script>
        //构造函数
        function Axios(config){
            //初始化配置
            this.defaults = config;
            this.intercepters = {
                request: {},
                response: {}
            }   
        }
        //原型添加方法,在其内部调用了request方法
        Axios.prototype.request = function(config){
            console.log('发送ajax请求,请求的类型为' + config.method);
        }
        Axios.prototype.get = function(config){
            return this.request({method:'get'});
        }
        Axios.prototype.post = function(config){
            return this.request({method:'post'});
        }

        //声明函数
        function createInstance(config){
            //实例化一个对象
            let context = new Axios(config);
            //此时可以context.get() context.post() 但是不能当做函数使用context()
            //创建请求函数
            let instance = Axios.prototype.request.bind(context);
            //instance是一个函数,并且可以instance({})，但是不能instance.get() instance.post()
            //将Axios的原型上的方法添加到instance上
            Object.keys(Axios.prototype).forEach(key =>{
                // console.log(key); //request get post
                instance[key] = Axios.prototype[key].bind(context);
            })
            //此时还缺少instance.defaults instance.intercepters
            // console.dir(instance);//ƒ bound ()
            // console.log(instance);//ƒ (config){console.log('发送ajax请求,请求的类型为' + config.method);}

            //为instance添加属性 defaults intercepters
            Object.keys(context).forEach(key =>{
                instance[key] = context[key];
            })
            // console.dir(instance);
            return instance;
        }
        let axios = createInstance();
        //完成后instance既可以当函数使用,也可以当对象使用
        //当函数使用来发送请求
        axios({method:'post'});
        //当对象使用来发送请求
        axios.get();
    </script>
```

## axios发送请求过程

![](http://rp1kb3lu6.bkt.gdipper.com/12.png)

里面的config是传递进来的

![](http://rp1kb3lu6.bkt.gdipper.com/13.png)

this.defaults其实就是默认配置对象

将默认配置对象与传入配置对象合并

![](http://rp1kb3lu6.bkt.gdipper.com/14.png)

创建promise对象
![](http://rp1kb3lu6.bkt.gdipper.com/15.png)

执行后就存在配置属性了

![](http://rp1kb3lu6.bkt.gdipper.com/16.png)

dispatchRequest方法

```js
//获取适配器对象
    var adapter = config.adapter || defaults.adapter;

    //发送请求， 返回请求后 promise 对象
    return adapter(config).then(function onAdapterResolution(response) {
        throwIfCancellationRequested(config);

        // Transform response data
        response.data = transformData(
            response.data,
            response.headers,
            config.transformResponse
        );
        //设置 promise 成功的值为 响应结果
        return response;
    }, function onAdapterRejection(reason) {
        if (!isCancel(reason)) {
            throwIfCancellationRequested(config);

            // Transform response data
            if (reason && reason.response) {
                reason.response.data = transformData(
                    reason.response.data,
                    reason.response.headers,
                    config.transformResponse
                );
            }
        }
        //设置 promise 为失败, 失败的值为错误信息
        return Promise.reject(reason);
    });

```

获取适配器对象
![](http://rp1kb3lu6.bkt.gdipper.com/17.png)

此处调用xhrAdapter方法，可以发送Ajax请求 函数内部返回的是promise对象。接着使用then返回成功或失败的回调

request → dispatch → xhr 再次返回结果，最后对结果做成功或失败的处理

## 总结

1. axios与Axios的区别

    axios是一个函数，Axios是一个类，axios的原型指向Axios.prototype，axios的实例是Axios的实例。axios是Axios.prototype.request函数bind返回后的函数，axios作为对象有Axios原型对象身上的属性和方法

2. instance与axios的区别

    相同: 都是一个能发请求的函数:request(config),都能发特定请求的各种方法:get,post,put,delete,都有默认配置和拦截器属性,dispatch/interceptors
    
    不同: 默认配置不一样,instance没有axios后面添加的一些方法:create(),CancelToken()

3. axios的运行流程
    
    1. 创建axios实例
    2. 添加默认配置
    3. 添加拦截器
    4. 发送请求
    5. 返回响应

    request(config) ==> dispatchRequest(config) ==> xhrAdapter(config)

4. axios的拦截器
    
    1. 请求拦截器
    2. 响应拦截器

