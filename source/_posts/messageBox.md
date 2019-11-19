---
title: 如何实现一个简单的messageBox
date: 2019-05-13 17:55:45
tags:
  - element-ui
  - messageBox
  - 源码
---

## 前言

几个月前遇到个需求要求在项目中的多个的地方做弹窗的二次确认，并且这些弹窗的内容形式多样各不相同。但由于这些弹窗是的作用仅仅是用于二次确认，于是用了messagebox的方式来实现。想到element的messageBox的样式不太符合需求所需要样式，就直接造了个轮子，避免了dialog比较繁琐的confirm和cancel的调用。

<!-- more -->

## 实现

首先先举个messageBox最简单的调用例子：

```js
	this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
		confirmButtonText: '确定',
		cancelButtonText: '取消',
		type: 'warning'
	}).then(() => {
		this.$message({
		type: 'success',
		message: '删除成功!'
		});
	}).catch(() => {
		this.$message({
		type: 'info',
		message: '已取消删除'
		});
	});
```


### 弹窗调用

从上述调用messageBox的代码中可以看到，在执行了this.$confirm之后，代码需要返回了一个promise，相应在源码中是通过调用MessageBox的函数来实现的，如下：

```js
let msgQueue = []; // 定义一个msg队列，用于存放messageBox配置
// 对于 messageBox有两种回调方式：Promise和callback（本文不对callback情况展开讨论）
const MessageBox = function (options, callback) {
	// ...
  if (typeof Promise !== 'undefined') {
    return new Promise((resolve, reject) => { // eslint-disable-line
		// 通过
      msgQueue.push({
        options: merge({}, defaults, MessageBox.defaults, options),
        callback: callback,
        resolve: resolve,
        reject: reject
      });

      showNextMsg();
    });
  }
	// ...
};
```

### 弹出窗口

在调用弹窗的代码中在返回一个Promise对象的同时，给一个名为msgQueue的数组新增了一个对象，这个对象中的属性包含了我们在调用messageBox所传的参数，会被用于后续messageBox的操作。接着执行了一个showNextMsg的方法，这个方法的主要作用是实例化弹窗，并且使用我们所传入的配置来配置messageBox实例，具体源码如下：

```js
const showNextMsg = () => {
  if (!instance) {
    initInstance();
  }
  instance.action = '';

  if (!instance.visible || instance.closeTimer) {
    if (msgQueue.length > 0) {
      currentMsg = msgQueue.shift();

      let options = currentMsg.options;
      for (let prop in options) {
        if (options.hasOwnProperty(prop)) {
          instance[prop] = options[prop];
        }
      }
      if (options.callback === undefined) {
        instance.callback = defaultCallback;
      }

      let oldCb = instance.callback;
      instance.callback = (acvtion, instance) => {
        oldCb(action, instance);
        showNextMsg();
      };
      if (isVNode(instance.message)) {
        instance.$slots.default = [instance.message];
        instance.message = null;
      } else {
        delete instance.$slots.default;
      }
      ['modal', 'showClose', 'closeOnClickModal', 'closeOnPressEscape', 'closeOnHashChange'].forEach(prop => {
        if (instance[prop] === undefined) {
          instance[prop] = true;
        }
      });
      document.body.appendChild(instance.$el);

      Vue.nextTick(() => {
        instance.visible = true; // 通过v-show="visible"的方式来控制弹窗是否显示
      });
    }
  }
};

```

通过执行以上的操作，便弹出了我们所要的messageBox。然后接下来是操作弹窗的操作：点击按钮。

### 点击按钮操作

首先点击按钮时，会触发doClose的操作。

doClose的主要作用分为关闭弹窗、执行回调。
（可以通过定义beforeClose方法使得弹窗的关闭变得可控）

#### 关闭弹窗

```js
	doClose () {
		if (!this.visible) return;
		this.visible = false;
		this._closing = true;

		this.onClose && this.onClose();
		messageBox.closeDialog(); // 解绑
		if (this.lockScroll) {
			setTimeout(this.restoreBodyStyle, 200);
		}
		this.opened = false;
		this.doAfterClose();
		setTimeout(() => {
			if (this.action) this.callback(this.action, this);
		});
	}
```

#### 执行回调

```js
// 当没有定义callback的时候会执行defaultCallback
const defaultCallback = action => {
  if (currentMsg) {
		// ...
    if (currentMsg.resolve) {
      if (action === 'confirm') {
        if (instance.showInput) {
          currentMsg.resolve({ value: instance.inputValue, action });
        } else {
          currentMsg.resolve(action);
        }
      } else if (currentMsg.reject && (action === 'cancel' || action === 'close')) {
        currentMsg.reject(action);
      }
    }
  }
};
```

可见，通过msgQueue存储messageBox的配置并且结合Promise的使用，就可以实现一个简单的messageBox。

除了本文所提到的功能以外，element-ui的messageBox还提供了很多其他功能，有兴趣可以自己去进一步探索。
