<template>
  <view :style="{height: pageHeight, overflow: 'hidden'}">
    <search @search="disableScroll" />
    <!-- 焦点图 -->
    <swiper
      class="banner"
      indicator-dots
      indicator-color="rgba(255, 255, 255, 0.6)"
      indicator-active-color="#fff"
      duration="1500"
      circular
      autoplay
    >
      <swiper-item v-for="item in swiper_arr" :key="item.goods_id">
        <navigator :url="'/pages/goods/index?id=' + item.goods_id ">
          <image :src="item.image_src"></image>
        </navigator>
      </swiper-item>
    </swiper>
    <!-- 导航条 -->
    <view class="navs">
      <navigator open-type="switchTab" url="/pages/category/index" v-for="(item, index) in nav_arr" :key="index">
        <image :src="item.image_src"></image>
      </navigator>
    </view>
    <!-- 楼层 -->
    <view class="floors">
      <view class="floor" v-for="(item, index) in floor_arr" :key="index">
        <view class="title">
          <image :src="item.floor_title.image_src"></image>
        </view>
        <view class="items">
          <navigator url="/pages/list/index" v-for="one in item.product_list" :key="one.name">
            <image :src="one.image_src"></image>
          </navigator>
        </view>
      </view>
    </view>
    <!-- 回到顶部 -->
    <view class="goTop icon-top" @click="goTop" v-if="scroll>200"></view>
  </view>
</template>

<script>
  import search from '@/components/search';

  export default {

    data () {
      return {
        pageHeight: 'auto',
        swiper_arr: [],
        nav_arr: [],
        floor_arr: [],
        scroll: 0
      }
    },

    components: {
      search
    },
    
    methods: {
      disableScroll (ev) {
        this.pageHeight = ev.pageHeight + 'px';
      },
      goTop () {
        uni.pageScrollTo({
          scrollTop: 0
        })
      },
      async getSwiper () {
        // const [err, res] = await uni.request({
        //   url: 'https://www.uinav.com/api/public/v1/home/swiperdata'
        // })
        const { message } = await this.http({
          url: '/api/public/v1/home/swiperdata'
        })
        this.swiper_arr = message
      },
      async getNavList () {
        const { message } = await this.http({
          url: '/api/public/v1/home/catitems'
        })
        this.nav_arr = message
      },
      async getFloorList () {
        const { message } = await this.http({
          url: '/api/public/v1/home/floordata'
        })
        this.floor_arr = message
      }
    },
    onShareAppMessage () {
      return {
        title: '山未动，心已远!一起和我旅游吧！',
        imageUrl: 'https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=3823400770,3555259268&fm=26&gp=0.jpg',
        path: '/pages/index/index'
      }
    },
    onLoad () {
      this.getSwiper()
      this.getNavList()
      this.getFloorList()
    },
    async onPullDownRefresh() {
      await this.getSwiper()
      await this.getNavList()
      await this.getFloorList()
      uni.stopPullDownRefresh()
    },
    onPageScroll (e) {
      this.scroll = e.scrollTop
    }
  }
</script>

<style scoped lang="less">
  .banner {
    width: 100%;
    height: 340rpx;

    image {
      width: 100%;
      height: 340rpx;
    }
  }

  .navs {
    display: flex;
    justify-content: space-between;
    padding: 30rpx 44rpx;

    image {
      width: 128rpx;
      height: 140rpx;
    }
  }

  .floor {

    .title {
      width: 750rpx;
      height: 60rpx;
      padding: 20rpx 0 0 8rpx;
      background-color: #f4f4f4;
    }

    .items {
      padding: 20rpx 16rpx;
      overflow: hidden;

      navigator {
        width: 193rpx;
        height: 188rpx;
        margin-left: 10rpx;
        margin-bottom: 10rpx;
        float: left;
      }

      navigator:first-child {
        width: 232rpx;
        height: 386rpx;
        margin-left: 0rpx;
      }

      navigator:nth-child(2),
      navigator:nth-child(5) {
        width: 273rpx;
      }
    }

    &:first-child {

      .items {

        navigator {
          width: 233rpx;
        }
      }
    }
  }

  .goTop {
    position: fixed;
    bottom: 30rpx;
    /* #ifdef H5 */
    bottom: 65px;
    /* #endif */
    right: 30rpx;
  
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100rpx;
    height: 100rpx;
    font-size: 48rpx;
    color: #666;
    border-radius: 50%;
    background-color: rgba(255, 255, 255, 0.8);
  }
</style>