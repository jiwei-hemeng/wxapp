<template>
  <view>
    <button type="primary" open-type="getUserInfo" @getuserinfo="btn_getInfo">微信登录</button>
  </view>
</template>

<script>
  export default {
    methods: {
      // 2.2 通过点击按钮用户授权获取用户信息
      async btn_getInfo (res) {
        this.getUserToken(res.detail)
      },
      // 2.1 通过js方式获取用户信息
      async js_getInfo () {
        // uni自带的方法用于获取用户信息
        const [err,res] = await uni.getUserInfo()
        // 如果获取到请求获取token
        if(res){
          this.getUserToken(res)
        }
      },
      // 3. 获取用户token 
      async getUserToken (userInfo) {
        const [err,codeObj] = await uni.login();
        // 调用相应的接口
        const {message,meta} = await this.http({
          url:"/api/public/v1/users/wxlogin",
          method:"POST",
          data:{
            encryptedData:userInfo.encryptedData,
            iv:userInfo.iv,
            rawData:userInfo.rawData,
            signature:userInfo.signature,
            code:codeObj.code,
          }
        });
        // 4. 输出获取到的token。如果是个人小程序一定不会成功，只有企业小程序才会成功。
        console.log(message)
        if (meta.status==200) {
          // 如果获取成功存入本地
          uni.setStorageSync("token",message.token);
          // 跳转回到上一个路由
          uni.navigateBack();
        }
        // uni.navigateBack();
      }
    },
    onLoad () {
      // 1. 页面加载时通过js方式获取用户信息，
      // 如果没有获取到通过用户点击按钮授权获取
      this.js_getInfo()
    }
  }
</script>

<style lang="less" scoped>
  button {
    width: 600rpx;
    margin: 200rpx auto 0;
  }
</style>