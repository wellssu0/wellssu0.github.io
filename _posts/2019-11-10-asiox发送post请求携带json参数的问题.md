---
layout:     post
title:     axios发送POST请求，携带json参数的问题
subtitle:   
date:       2019-11-10
author:     wells
header-img: img/c22.jpg
catalog: true
tags:

    -  Axios
---


### 问题：

发送一个正常的post请求

```js
axios.request({
    url:'/login',
    method:'POST',
    data: {
    	username:"susu",
    	password:'susu'
	},
    headers: {
      'Content-Type': 'application/json; charset=utf-8',
      Accept: 'application/json',
    },
})
```

后台的admin账号是有的，请求响应显示账号或密码错误，说明后台接受的参数不对或者没有接收到参数。

### 问题原因：

1. 服务端要求的 'Content-Type': `'application/x-www-form-urlencoded' `以及 `@RequestParam` ，就是后端只能请求的地址中得到请求参数，也就是说只能从` username=susu&password=susu`这种字符串解析出参数，

2. 而我们的请求头的'Content-Type'是`application/json; charset=utf-8`， axios会帮我们 **转换请求数据和响应数据** 以及 **自动转换 JSON 数据** ， axios 使用 post 发送数据时，默认是直接把 json 放到请求体中提交到后端的 

### 解决方案：

1. 用 URLSearchParams 传递参数 

    ```js
    let param = new URLSearchParams()
    param.append('username', 'admin')
    param.append('pwd', 'admin')
    axios({
        method: 'post',
        url: '/api/lockServer/search',
        data: param
    })
    ```

	> 注意： `URLSearchParams` 不支持所有的浏览器，但是总体的支持情况还是 OK 的



2. 将自己请求头中的Content-Type改成application/x-www-form-urlencoded，并且将json参数转换成query参数。

    ```js
    import qs from 'qs' //引入qs库

    const configs = { //
        url:'/login',
        method:'POST',
        data: {
            username:"susu",
            password:'susu'
        },
        headers: {
          'Content-Type': 'application/json; charset=utf-8',
          Accept: 'application/json',
        }
    }

    if (configs.method.toLowerCase() === 'post' && typeof configs.data==='object') {
        configs.data = qs.stringify(configs.data) // username=susu&password=susu
        headers['Content-Type'] = 'application/x-www-form-urlencoded' //更改content-type
     }

    axios.request(configs)
    ```



3. 通过修改 `transformRequest` 来达到我们的目的
	在 axios 的请求配置项中，是有 `transformRequest` 的配置的：

    ```js
    axios.request({
        url:'/login',
        method:'POST',
        transformRequest: [function (data) {
            // 对 data 进行任意转换处理
            return qs.stringify(data)
        }],
        data: {
            username:"susu",
            password:'susu'
        },
        headers: {
          'Content-Type': 'application/json; charset=utf-8',
          Accept: 'application/json',
        },
    })
    ```

4.  重写一个 axios 实例，重新实现属于我们自己的 transformReques 

   ```js
   import axios from 'axios'
   let instance = axios.create({
   	transformRequest: [function transformRequest(data, headers) {
   	    normalizeHeaderName(headers, 'Content-Type');
   	    if (utils.isFormData(data) ||
   	      utils.isArrayBuffer(data) ||
   	      utils.isBuffer(data) ||
   	      utils.isStream(data) ||
   	      utils.isFile(data) ||
   	      utils.isBlob(data)
   	    ) {
   	      return data;
   	    }
   	    if (utils.isArrayBufferView(data)) {
   	      return data.buffer;
   	    }
   	    if (utils.isURLSearchParams(data)) {
   	      setContentTypeIfUnset(headers, 'application/x-www-form-urlencoded;charset=utf-8');
   	      return data.toString();
   	    }
   	    /*改了这里*/
   	    if (utils.isObject(data)) {
   	      setContentTypeIfUnset(headers, 'application/x-www-form-urlencoded;charset=utf-8');
   	      let _data = Object.keys(data)
   	      return encodeURI(_data.map(name => `${name}=${data[name]}`).join('&'));
   	    }
   	    return data;
   	}],
   })
   ```




5. 直接在参数这写

   ```js
    axios.post('/api/lockServer/search',"userName='admin'&pwd='admin'")
   ```

   

6. 我们知道现在我们服务端同学接收参数用的是 `@RequestParam`（通过字符串中解析出参数）
   其实还有另一种是 `@RequestBody`（从请求体中获取参数）。我们让后端的同学改成 `@RequestBody` 