---
uuid: dbc69650-6982-11e8-8a8a-073bb38bb8a8
author: dujun
email: emolingzhu@126.com
github: https://github.com/jiandandkl
avatar: https://avatars1.githubusercontent.com/u/16009933?s=40&v=4
title: wx-getUserInfo废弃之后-弹窗获取信息
date: 2018-06-06 20:12:17
tags:
---


  > 先吐槽下微信,这么基础的API晚上11点说废就废,程序猿何苦为难程序猿.(不过最近好像又恢复了...)

  官方是推荐使用button点击获取用户信息,但有时候页面多这么一个按钮很突兀啊、或者tabbar切换的时候想要先获取用户信息再展示页面就很尴尬了。
  目前碰到的情况就是第二种,既然必须要点击button来获取,就采用模态框点击登录的方法了。
  但小程序的模态框实在鸡肋(尤其很难理解字符限制七个字),所以自定义模态框。

  * loading模态框

  目前小程序是不支持自定义的loading图,只有`success`、`loading`、`none`,所以写个公共组件,可以传入自定义的图片及文字。
  其中的关键方法为: `getCurrentPages()[getCurrentPages().length - 1]`,获取当前页实例。拿到当前页实例后便可以将自定义内容传到data中。

  showToast.js:

  ```javascript
  function showToast(obj) {
    // 获取当前page实例
    const that = getCurrentPages()[getCurrentPages().length - 1];
    that.setData({
      showToast: obj
    })
  }

  ```
  showToast.wxml:
  ```javascript
  <view class="middle" wx:if="{{showToast.icon}}">
    <image class="toast-icon" src="{{showToast.icon}}" mode="scaleToFill" wx:if="{{showToast.icon}}" />
    <text wx:if="{{showToast.content}}" class="toast-content">{{showToast.content}}</text>
  </view>
  ```

  loading.js:
  ```javascript
  const feedbackApi = require('../../components/showToast/showToast')
  feedbackApi.showToast({
    content: '正在获取数据',
    icon: '../../images/loading.svg',
    duration: 3000
  })
  ```