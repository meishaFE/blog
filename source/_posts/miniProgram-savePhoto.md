---
title: 小程序保存图片到相册优雅解决方式
date: 2019-01-08 20:00:00
tags:
- 小程序
---

## 背景说明

在小程序开发中，经常有需要保存一张动态生成的图片到用户手机相册的需求，这个时候，如何优雅的处理这一逻辑，让用户感知更明显，显得尤为重要，通过之前的开发，整理出了一套相对来说较为优雅的处理方式，希望通过此次整理，以后在开发此类需求就不用再耗费太多的时间和心思在这上面了。

<!-- more -->

### 授权处理
保存图片到相册这一操作，属于用户主动触发的机制，而且，这一操作是需要用户来授权应用获得保存图片到相册的权限的，所以，第一步，就是处理授权。
在小程序中，提供了一个api:wx.getSetting来获取用户的当前设置。在请求成功的回调返回值中只会出现小程序已经向用户请求过的权限，所以此处应该做一个是否拿到保存图片到相册的权限的判断，
如果有，则去到下载图片的逻辑，没有的话则需要去唤起用户授权的操作。
此处封装的第一个函数如下：
```
// 获取相册授权
let getSaveScope =  () => {
  let _this = this;
  if (_this.isClick) { // 防止重复点击
    return;
  };
  _this.isClick = true;
  wx.getSetting({
    success (res) {
      // 是否获取保存图片到相册的权限
      if (!res.authSetting['scope.writePhotosAlbum']) {
        _this.userAuthorize(); // 唤起用户授权
      } else {
        _this.saveToAlbum(); // 保存图片到相册操作
      };
    },
    fail (error) {
      _this.isClick = false;
      console.log(error);
    }
  });
}
```

### 唤起用户授权

这里先处理程序没有拿到保存图片到相册的权限的情况，这里需要考虑一个情况，即用户之前是否是第一次拒绝过授权，如果是第一次拒绝，则只会简单提示用户需要授权才能进行保存的这一操作，如果不是第一次，才弹出让用户去打开这一权限的设置，代码实现如下：

```
const userAuthorize = () => {
  let _this = this;
  let isFirst = wx.getStorageSync('isFirst') || 0; // 是否第一次拒绝
  // 提前向用户发起授权请求
  wx.authorize({
    scope: 'scope.writePhotosAlbum',
    success () {
      // 保存图片到相册操作
      _this.saveToAlbum();
    },
    fail (error) { // 拒绝授权，第二次才提醒用户去打开授权
      console.error(error);
      if (isFirst === 0) {
        wx.setStorageSync('isFirst', 1);
        wx.showToast({
          title: '需要授权才能继续保存',
          icon: 'none',
          duration: 2000
        });
        _this.isClick = false;
      } else {
        wx.showModal({
          title: '提示',
          content: '请打开「保存到相册」权限',
          confirmText: '去设置',
          success (res) {
            if (res.confirm) {
              _this.toSetPage(); // 去设置页
            } else {
              _this.isClick = false;
            }
          }
        });
      }
    }
  });
}

/** 
用户自主设置授权操作页
调起客户端小程序设置界面，返回用户设置的操作结果。
设置界面只会出现小程序已经向用户请求过的权限
**/
const toSetPage =  () => {
  this.isClick = false;
  wx.openSetting({
    success (res) {
      console.log(res.authSetting);
    },
    fail (errorMsg) {
      console.error(errorMsg);
    }
  });
}

```

### 下载图片到相册

如果用户已经授权保存图片到相册的权限后，就可以开始下载图片的操作了。
由于微信提供的api =》 saveImageToPhotosAlbum不支持保存网络图片地址，但一般来说，我们生成的图片都是网络图片，所以我们需要通过downloadFile这api将网络图下载下来生成一个临时图片路径,具体的逻辑代码实现如下：

```
const saveToAlbum = () => {
  if (this.loading) { // 防止重复调用
    return;
  };
  const imgSrc = this.shareImgSrc; // 网络图地址
  let _this = this;
  this.loading = true;
  wx.downloadFile({
    url: imgSrc,
    success: function (res) {
      // 图片保存到本地
      wx.saveImageToPhotosAlbum({
        filePath: res.tempFilePath,
        success: function (data) {
          _this.loading = false;
          _this.isClick = false;
          wx.showToast({
            title: '图片已保存',
            icon: 'none',
            duration: 2000
          });
        },
        fail: function (err) {
          console.error(err);
          _this.loading = false;
          _this.isClick = false;
          if (err.errMsg === 'saveImageToPhotosAlbum:fail auth deny') {
            // 无法访问你的手机相册，图片无法保存到你的相册
            wx.showModal({
              title: '提示',
              showCancel: false,
              content: '无法访问你的手机相册，名片海报无法保存到你的相册',
              confirmText: '知道了',
              success (res) {
                console.log(res);
              }
            });
          };
        }
      });
    }
  });
}

```

### 总结

通过以上几个步骤，实现保存动态图片到相册这一操作基本就完整了，如果其中还有其他没有考虑到或者改进的地方，希望大家能够指出来。此外，小程序开发和平时的前端开发的习惯还是不太一样，里面的坑或者细节也挺多，很多的流程也要结合微信官方给出的api或者限制来做最优解，所以，在开发的时候一定要特别的注意，然后，再解决一个问题后应该能够及时的总结出来，给自己也给其他人留下一个经验。


