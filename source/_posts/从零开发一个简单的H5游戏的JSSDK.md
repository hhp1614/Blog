---
title: 快速配置项目的 prettier 和 eslint
date: 2021-11-16 19:02:10
tags:
    - JS
---

# 从零开发一个简单的 H5 游戏的 JS SDK

H5SDK 一般是由渠道方开发，提供登录、支付等服务给研发方使用

> 本教程使用简单的 `HTML` + `JS` 实现，实际开发可使用 `webpack` 等构建工具搭建，好处就是可以集成一些工具链进去方便开发，比如 `Vue`、`Sass`。。。有空再写一篇使用 `webpack` 构建的吧

## 基本结构

研发方提供一个游戏链接，在渠道方的页面上使用 `iframe` 加载，就像这样

```html
<!-- index.html -->
<iframe id="game_frame" src="./game.html"></iframe>
```

研发方的页面需要引入渠道方提供的 SDK 文件，像这样

```html
<!-- game.html -->
<script src="./sdk.js"></script>
```

这个 SDK 文件主要负责提供一些方法给研发方调用，研发方调用之后，会给父页面（也就是上面的 `index.html`）发送消息，然后在父页面内处理各种逻辑，完事之后再发送消息回来，研发方通过调用方法传入的回调函数（当然也可以使用 Promise 来实现）来收到父页面的处理的结果

> PS: 这里就是整个 SDK 最核心的部分了，主要是使用 `postMessage` 整个 API

这里我们使用 `index.html` 表示渠道方的页面（也就是父页面），`game.html` 表示研发方的页面（也就是游戏页面），使用下面的目录结构

```text
- index.html
- index.js
- sdk.js
- game.html
```

## index.html

这是渠道方的页面，这里可以写登录、支付等界面

```html
<iframe id="game_frame" src="./game.html"></iframe>
<script src="./index.js"></script>
```

## index.js

我们先实现一下 `index.js`

```js
// 封装一下给 sdk.js 发生信息的方法
const postMsg = (action, data) => {
    const gameFrame = document.getElementById('game_frame');
    const postData = { action, data };
    if (gameFrame.postMessage) {
        gameFrame.postMessage(postData, '*');
    } else if (gameFrame.contentWindow.postMessage) {
        gameFrame.contentWindow.postMessage(postData, '*');
    } else {
        console.log('数据发送失败：当前浏览器不支持 postMessage');
    }
};

// 监听子页面传过来的信息
window.addEventListener('message', e => {
    const ed = e.data;
    const action = ed.action ? ed.action : '';
    const data = ed.data ? ed.data : {};
    if (!action) {
        return;
    }
    switch (action) {
        case 'init':
            postMsg('init', { msg: '初始化成功' });
            break;
        case 'login':
            postMsg('login', { msg: '登录成功' });
            break;
        case 'pay':
            postMsg('pay', { msg: '支付成功' });
            break;
    }
});
```

## sdk.js

下面我们开始写 `sdk.js` 文件，`sdk.js` 需要提供一个全局对象 SDK，对象内有 `init`，`login`，`pay` 三个方法

```js
// 用于存储研发方传入的回调函数
const callbacks = {};
// 简单封装一下 postMessage 这个方法
const postMsg = (action, data) => parent.postMessage({ action, data }, '*');

// 对外暴露 SDK 这个全局对象
window.SDK = {
    init(params, callback) {
        // 将研发方传入的回调函数存起来
        callbacks.init = callback;
        // 给父页面（index.html）发送信息
        postMsg('init', params);
    },
    login(params, callback) {
        callbacks.login = callback;
        postMsg('login', params);
    },
    pay(params, callback) {
        callbacks.pay = callback;
        postMsg('pay', params);
    },
};

// 接收父页面传过来的信息
window.addEventListener('message', e => {
    const ed = e.data;
    const action = ed.action ? ed.action : '';
    const data = ed.data ? ed.data : {};
    // 如果 action 不存在，就不做任何操作
    if (!action) {
        return;
    }
    // 当父页面（index.html）传过来的 action 为下面这些时，执行研发方传入的回调函数，并把结果作为参数传进去，这样研发方就可以在回调函数那里取到父页面返回的结果了
    switch (action) {
        case 'init':
            // 初始化
            callbacks.init && callbacks.init(data);
            break;
        case 'login':
            // 登录
            callbacks.login && callbacks.login(data);
            break;
        case 'pay':
            // 支付
            callbacks.pay && callbacks.pay(data);
            break;
    }
});
```

## game.html

最后，这是研发方的页面，需要在页面引入 `sdk.js`，这个文件需要渠道方开发，然后调用这个文件里的方法

```html
<iframe src="./game.html"></iframe>
<script src="./sdk.js"></script>
<script>
    // 这段代码用来测试，看到这应该明白，sdk.js 需要暴露 init，login，pay 三个方法出来供研发方调用
    SDK.init({ msg: '测试调用初始化方法' }, initRes => {
        console.log('初始化结果', initRes); // { msg: '初始化成功' }

        SDK.login({ msg: '测试调用登录方法' }, loginRes => {
            console.log('登录结果', loginRes); // { msg: '登录成功' }

            SDK.pay({ msg: '测试调用支付方法' }, payRes => {
                console.log('支付结果', loginRes); // { msg: '支付成功' }
            });
        });
    });
</script>
```
