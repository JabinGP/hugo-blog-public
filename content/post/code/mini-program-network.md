---
title: "小程序异步网络请求问题的解决方案"
date: 2020-12-08T15:38:11+08:00
images: []
categories: ["code"]
tags: ["javascript","小程序"]
---

### 业务逻辑

最近开发一个便签小程序的时候，有这样一个需求：用户可以在写便签的时候添加一个或多个图片。

对于这个需求，我们用户按下保存键时，内部具体的实现上是这样的逻辑：

1. 首先检测用户是否传入了图片，如果存储本地图片地址的数组长度>=1,则将图片数组放入上传图片的函数。

2. 由于小程序网络请求大小限制，我们只能采取循环上传单文件，然后收集每次请求的结果--图片在服务器的地址，最后将结果放在一个数组中供后续的操作使用。

3. 当图片上传函数全部执行完毕后，将数组中的图片数组取出来，赋值到日记对象中，再将整个日记对象提交到服务器。

4. 服务器返回保存成功或失败。

思路其实非常清晰简单，但是在代码实现上却翻了大跟头。

### 异步带来的问题

小程序的网络请求是异步的：我们无法通过return来将网络请求结果返回出来使用。

```js

    wx.request({

          //...省略其他属性

          success: function (res) {

          },

          fail: function (res) {

          }

    })

 ```

例如在微信中发送网络请求，我们只能使用微信提供的方法wx.xxx，其中请求的结果保存在res中，而res无法直接return得到。

**解决：**res虽然无法直接获取，但是我们能通过将需要使用到这个请求结果的业务逻辑代码放入这个网络请求的回调函数中直接读取网络请求结果，也就是一切都需要通过回调来解决。

```js

    wx.request({

          //...省略其他属性

          success: function (res) {

            console.log(res);

            //接业务逻辑代码

          },

          fail: function (res) {

            console.log(res);

          }

    })
 ```

例如这个微信的网络请求，我们可以通过success和fail的回调函数来读取res的值从而完成依赖res结果的业务逻辑。

#### 回调地狱

虽然解决了结果获取的问题，但是又产生了另一个问题，当多个请求中有明确的先后顺序时，回调会嵌套的很厉害，造成回调地狱，代码可读性和可维护性都会很差。
例如对于一个日记页面，需要先请求到页面的数据（里面包含了图片数据和其他数据的地址），再根据页面数据去请求图片数据后再请求音频数据。例如以下代码：

 ```js

    //请求页面整体数据

    wx.request({

          //...省略其他属性

          success: function (res) {//成功

                //请求图片数据

                wx.request({

                  success: function (res) {//成功

                      //请求音频数据

                      wx.request({

                          success: function (res) {//成功

                          },

                          fail: function (res) {//失败

                              console.log("请求失败:"+res);

                          }

                      })

                  },

                  fail: function (res) {//失败

                      console.log("请求失败:"+res);

                  }

                })

          },

          fail: function (res) {//失败

              console.log("请求失败:"+res);

          }

    })
```

**如何优化？**幸运的是，在es6里面我们可以用promise去优化我们的回调，用then代替回调，首先将网络请求封装成一个Promise：

```js

    // 后台post请求

    function postRequest(posturl, postdata) {

      return new Promise((resolve, reject) => {

        wx.request({

          //省略其他属性

          success: function (res) {

            console.log("at post request: 请求成功")

            resolve(res.data)//设置promise成功标志

          },

          fail: function (res) {

            console.log("at post request: 请求失败")

            reject(res.data)//设置promise失败标志

          }

        })

      });

    }

```

这样封装以后，我们的网络请求会在success和fail后回调resolve,这样可以告诉promise，“hey，我完成我的工作了，你可以进行你的then操作了”，这样就可以用then来简化嵌套逻辑。使用promise来完成上面那个问题的请求将会是这样的：

```js

    postRequest(posturl,postdata)

    .then(function(res){

      //业务逻辑

      //调用下一个请求

      return postRequest(next_posturl,next_postdata);

    })

    .then(function(res){

      //业务逻辑

      //调用下一个请求

      return postRequest(next_next_posturl,next_next_postdata);

    })

    .then(function(res){

      //业务逻辑

    });

```

是不是简洁的多~

### 一个看似简单的需求

我们的有一个很简单的需求是需要对一组数量不定的图片做分别上传（因为微信限制所以无法做多上传），并且在上传完成以后需要获取到所有的返回结果。

那么用我们前面的回调函数+then的话，很自然的想到这样的写法

```js

postRequest(posturl,postdata)

.then(function(res){

    //获取返回res

    //上传下一个图片

    return postRequest(next_posturl,next_postdata);

})

.then(function(res){

    //获取返回res

    //上传下一个图片

    return postRequest(next_next_posturl,next_next_postdata);

})

.then(function(res){

    //获取返回res

});

```

这样看起来很简单明了，但是我的图片数量是不定的，怎么动态的构建.then.then.then这样的链式调用呢？经过我的研究后发现可以通过一个辅助的promise链去完成主链的链式构建。

```js

//多文件上传

function jabingp_upLoad(uploadurl, files) {

  return new Promise((resolve, reject) => {

    //初始化promise链

    var mergedAjax = Promise.resolve();

    var response = [];

    // 循环上传

   
    // 这里一定要使用let来为没一次循环构建一个块级作用域
    // 使用var则需要配合立即执行函数
    for (let i = 0; i < files.length; i++) {

      mergedAjax = mergedAjax.then(() => {

        return jabingp_upLoadSingle(uploadurl, files[i]).then((res) => {

          response.push(res);

        });

      });

    }

    //当前面循环中所有的then执行完毕时会执行这个then

    mergedAjax.then(() => {

      resolve(response);            //设置这个函数的promise对象为完成状态并放入数据

    });

  });

}

```

通过这个函数，就完成了多个请求依次执行并收集结果的效果。这个函数的重点在于利用另外一个已经处于完成状态的promise，不断的迭代自身，在每次迭代的then内部通过return来完成辅助链到业务链的切换。

> 2019-04-27 更新

### 使用await/async更加优雅地处理异步吧

在es7标准中，引入了await和async这对兄弟，它们可以让我们的异步代码看起来和同步代码一样。让我们来看看await和async都能做什么吧。

await可以等待一个promise运行到完成状态并且获取结果，或者等待一个async修饰的函数运行完成并获取结果，但是使用await的时候，必须在async函数体内部。比如我有这样一个网络请求:

```js
 function postRequest(posturl, postdata) {

     return new Promise((resolve, reject) => {

       wx.request({

         //省略其他属性

         success: function (res) {

           console.log("at post request: 请求成功")

           resolve(res.data)//设置promise成功标志

         },

         fail: function (res) {

           console.log("at post request: 请求失败")

           reject(res.data)//设置promise失败标志

         }

       })

     });

   }
```

那么如果不使用await，我就需要这样取得请求结果

```js
function test(){
  postRequest(xxx,xxx).then(function(res){
      // 这里面可以读取请求结果res了
      console.log(res);
  });
}
test();
```

可以看到，这样的代码不太符合常规逻辑，我们希望函数作用是返回数据，这样更清晰明了，有了await，我们的愿望就可以实现了。

```js
async function test(){
 let res = await  postRequest(xxx,xxx);
 // 下面就可以正常写对res的读取了
 console.log(res);
}
test();
```

注意我给函数加上了async，有了async和await，我们就可以像同步代码一样使用异步请求了~
那么上面那个通过复杂的构建链完成的需求，通过await实现将会变得非常简单易懂。

```js
async function jabingp_upLoad(uploadurl, files) {
    let response = [];
  
    // 循环依次等待上传结果
    for (let i = 0; i < files.length; i++) {
        let  res = await jabingp_upLoadSingle(uploadurl, files[i]);
        // 结果放入数组
        response.push(res);
    }
    // 返回结果
    return  response ;
}

```

代码一下子变得简洁易懂了，注意调用的时候也同样需要在一个async函数内部执行await。

```js
async function test(){
 let response = await  jabingp_upLoad(xxx,xxx);
 console.log(response );
}
test();
```

是不是非常简单呢，赶紧在你的异步请求中使用async和await吧！
