## 小程序的生命周期

+ onLaunch 启动就执行
+ onShow 切换到前台运行时
+ onHide 切后台时
+ onError 小程序抛出错误时，捕获错误时
+ onPageNotFound 没用找到页面时

小程序中将生命周期分成两类：应用级别App；页面级别Page。

## 小程序原生的api

* 加载框

  ```js
  wx.showLoading({
    title: '加载中',
  })
  
  setTimeout(function () {
    wx.hideLoading()；
  }, 2000)
  ```

+ 显示一个弹出框，确定和取消按钮

  ```js
  wx.showModal({
    title: '提示',
    content: '这是一个模态弹窗',
    // 点击按钮的执行函数；
    success (res) {
      if (res.confirm) {
        console.log('用户点击确定')
      } else if (res.cancel) {
        console.log('用户点击取消')
      }
    }
  })
  ```

+ 点击组件，后简单的信息提示

  ```js
  wx.showToast({
      title: '成功',
      icon: 'success',
      duration: 2000
  })
  ```

+ 模拟类似于系统的菜单，菜单项可以进行设置，选择后知道选择是哪个

  ```js
  wx.showActionSheet({
    itemList: ['A', 'B', 'C'],
    success (res) {
      console.log(res.tapIndex);
    },
    fail (res) {
      console.log(res.errMsg)
    }
  })
  ```

+ 选择图片

  ```js
  wx.chooseImage({
   // 选择几张照片
    count: 1,
    // 所选的图片的尺寸：原图，压缩图
    sizeType: ['original', 'compressed'],
    // 来源：相册、相机
    sourceType: ['album', 'camera'],
    // 选择其中一项后的回调
    success (res) {
      // 临时的文件地址
      const tempFilePaths = res.tempFilePaths;
    }
  })
  ```

+ 给服务器传递数据

  ```js
  wx.uploadFile({
      url: 'https://example.weixin.qq.com/upload', // 仅为示例，非真实的接口地址
      filePath: "https://xx.com/asd/xxx.png",   // 上传的文件！形式：网络地址形式；
      name: 'image_file',  // 后台接受图片文件的字段；后台定；
      success (res){
  		// 请求成功的时候回调
      }
  })
  ```

## 地址参数

在页面上的使用

```html
<navigator wx:for="{{List}}" to="/page/index?id={{item.id}}">{{item.name}}</navigator>
```

在js中的获取

```js
Page({
    onLoad: function(query){
        console.log(query)
    }
})
```

