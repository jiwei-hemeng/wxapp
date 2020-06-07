<template>
  <view class="wrapper">
    <!-- 收货信息 -->
    <view class="shipment">
      <block v-if="addr">
        <view class="dt">收货人: </view>
        <view class="dd meta">
          <text class="name">{{addr.userName}}</text>
          <text class="phone">{{addr.telNumber}}</text>
        </view>
        <view class="dt">收货地址:</view>
        <view class="dd">{{addr.details}}</view>
      </block>
      <button v-else type="primary" @click="getAttr">点击添加地址</button>
    </view>
    <!-- 购物车 -->
    <view class="carts">
      <view class="item">
        <!-- 店铺名称 -->
        <view class="shopname">优购生活馆</view>
        <view class="goods" v-for="(item, index) in carts" :key="index">
          <!-- 商品图片 -->
          <image class="pic" :src="item.goods_small_logo"></image>
          <!-- 商品信息 -->
          <view class="meta">
            <view class="name">{{item.goods_name}}</view>
            <view class="price">
              <text>￥</text>{{item.goods_price}}<text>.00</text>
            </view>
            <!-- 加减 -->
            <view class="amount">
              <text class="reduce" @tap="change_number(index, -1)">-</text>
              <input type="number" :value="item.goods_number" class="number">
              <text class="plus" @tap="change_number(index, 1)">+</text>
            </view>
          </view>
          <!-- 选框 -->
          <view class="checkbox">
            <icon type="success" size="20" :color="item.goods_buy?'#ea4451':'#ccc'" @tap="change_check(index)"></icon>
          </view>
        </view>
      </view>
    </view>
    <!-- 其它 -->
    <view class="extra">
      <label class="checkall">
        <icon type="success" :color="isAllbuy?'#ea4451':'#ccc'" size="20" @tap="chang_all_buy"></icon>
        全选
      </label>
      <view class="total">
        合计: <text>￥</text><label>{{sum_price}}</label><text>.00</text>
      </view>
      <view class="pay" @tap="pay">结算({{ck_buy.length}})</view>
    </view>
  </view>
</template>

<script>
  export default {
    data () {
      return {
        carts: [],
        sum: 0,
        addr: null
      }
    },
    methods: {
      change_check(index){
        this.carts[index].goods_buy = !this.carts[index].goods_buy
        uni.setStorageSync('carts', this.carts)
      },
      change_number (index,step) {
        if(step == -1 && this.carts[index].goods_number == 1){
          return
        }
        if(step == 1 && this.carts[index].goods_number == 15){
          return
        }
        this.carts[index].goods_number += step
        uni.setStorageSync('carts', this.carts)
      },
      chang_all_buy () {
        let ck = !this.isAllbuy
        this.carts.forEach(item => {
          item.goods_buy = ck
        })
        uni.setStorageSync('carts', this.carts)
      },
      getAttr () {
        uni.chooseAddress({
          success:(res)=>{
            this.addr = res;
            // 详细地址
            this.addr.details = res.provinceName+res.cityName+res.countyName+res.detailInfo;
          }
        });
      },
      async pay () {
        if(!this.addr){
          uni.showToast({
            title: '不能没有收货地址哦！'
          })
          return
        }
        if(!this.ck_buy.length){
          uni.showToast({
            title: '请选择您想要的宝贝哦！',
            icon: 'none'
          })
          return
        }
        if(!uni.getStorageSync('token')){
          uni.navigateTo({
            url:"/pages/auth/index"
          })
          return
        }
        this.toPay()
      },
      async toPay () {
        // 1. 请求创建订单
        const {message,meta} = await this.http({
          url:"/api/public/v1/my/orders/create",
          method:"POST",
          header:{
            "Authorization" : uni.getStorageSync("token")
          },
          data:{
            order_price:this.sum,
            consignee_addr:this.addr.details,
            goods:this.ck_carts
          }
        })
        // 2. 如果订单创建成功
        if (meta.status==200) {
          // 清除购物车数据的数据 （业务上） 
          uni.removeStorageSync("carts");
          // 转跳到结算页面
          uni.navigateTo({
              url:"/pages/order/index"
          });
        }
      }
    },
    computed: {
      isAllbuy:function(){
        return this.carts.length == this.ck_buy.length
      },
      ck_buy () {
        const arr = []
        this.carts.forEach(item => {
          if(item.goods_buy){
            arr.push(item)
          }
        })
        return arr
      },
      sum_price() {
        let sum = 0
        this.carts.forEach(one => {
          if(one.goods_buy) {
            sum += one.goods_number * one.goods_price
          }
        })
        return sum
      }
    },
    onShow(){
      this.carts = uni.getStorageSync('carts')
    }
  }
</script>

<style scoped lang="less">
  .shipment {
    height: 100rpx;
    line-height: 2;
    padding: 30rpx 30rpx 40rpx 30rpx;
    font-size: 27rpx;
    color: #333;
    background-color: #fff;
    background-image: url(http://static.botue.com/ugo/images/cart_border%402x.png);
    background-size: contain;
    background-repeat: no-repeat;
    background-position: bottom;

    .dt {
      width: 140rpx;
      float: left;
      clear: both;
    }

    .dd {
      padding-left: 160rpx;
    }

    .meta {
      padding-right: 50rpx;
    }

    text.phone {
      float: right;
    }
  }

  .carts {
    background-color: #f4f4f4;
    padding-bottom: 110rpx;
    overflow: hidden;

    .shopname {
      padding: 30rpx;
      margin-top: 20rpx;
      font-size: 30rpx;
      color: #333;
      background-color: #fff;
      border-top: 1rpx solid #eee;
      border-bottom: 1rpx solid #eee;
    }

    .goods {
      display: flex;
      padding: 30rpx 20rpx 30rpx 0;
      margin-left: 100rpx;
      border-bottom: 1rpx solid #eee;
      background-color: #fff;
  
      position: relative;

      .checkbox {
        width: 101rpx;
        height: 100%;
        background-color: #fff;

        display: flex;
        justify-content: center;
        align-items: center;

        position: absolute;
        left: -100rpx;
        top: 0;
      }

      &:last-child {
        border-bottom: none;
      }

      .pic {
        width: 200rpx;
        height: 200rpx;
        margin-right: 30rpx;
      }

      .meta {
        flex: 1;
        font-size: 27rpx;
        color: #333;
        position: relative;
      }

      .name {
        width: 100%;
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
      }

      .price {
        position: absolute;
        bottom: 0;

        color: #ea4451;
        font-size: 33rpx;

        text {
          font-size: 22rpx;
        }
      }

      .amount {
        position: absolute;
        bottom: 0;
        right: 20rpx;

        height: 48rpx;
        text-align: center;
        border: 1rpx solid #ddd;
        border-radius: 8rpx;

        display: flex;
        align-items: center;

        text {
          display: block;
          width: 60rpx;
          line-height: 48rpx;
          font-size: 36rpx;
          color: #ddd;
          text-align: center;
        }

        input {
          width: 60rpx;
          height: 48rpx;
          min-height: 48rpx;
          font-size: 27rpx;
          border-left: 1rpx solid #ddd;
          border-right: 1rpx solid #ddd;
        }
      }
    }
  }

  .extra {
    position: fixed;
    bottom: 0;
    /* #ifdef H5 */
    bottom: 50px;
    /* #endif */
    left: 0;
    z-index: 9;

    width: 750rpx;
    height: 96rpx;
    text-align: center;
    line-height: 96rpx;
    font-size: 36rpx;
    border-top: 1rpx solid #eee;
    background-color: #fff;
    color: #333;
    display: flex;

    .checkall {
      width: 140rpx;
      line-height: 1;
      margin-left: 25rpx;
      display: flex;
      align-items: center;

      icon {
        margin-right: 20rpx;
      }
    }

    .total {
      display: flex;
      justify-content: center;
      flex: 1;

      label, text {
        color: #ea4451;
        vertical-align: bottom;
        position: relative;
        bottom: -2rpx;
      }

      text {
        position: relative;
        bottom: -3rpx;
        font-size: 24rpx;

        &:first-child {
          margin-left: 10rpx;
        }
      }
    }

    .pay {
      width: 200rpx;
      background-color: #ea4451;
      color: #fff;
    }
  }
</style>

