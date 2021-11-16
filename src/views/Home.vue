<template>
  <div class="home">
    <!-- 导航栏 -->
    <van-nav-bar left-text="返回" right-text="按钮" left-arrow fixed>
      <template #left>
        <div class="user-title">
          <div class="time">晚上好,</div>
          <div class="user-name">Allen</div>
        </div>
      </template>
      <template #right>
        <van-search placeholder="请输入商品的名称" />
      </template>
    </van-nav-bar>
    <!-- 商品区域 -->
    <div class="products-box">
      <!-- 轮播图 -->
      <div class="banner-box">
        <van-swipe class="my-swipe" indicator-color="white">
          <van-swipe-item v-for="(item, index) in bannerData" :key="index"
          @click="goDetail(item.pid)">
            <img class="auto-img" :src="item.bannerImg" alt="" />
            <div class="img-name">{{ item.name }}</div>
          </van-swipe-item>
          <template #indicator>
            <span></span>
          </template>
        </van-swipe>
      </div>
      <!-- 热门推荐 -->
      <div class="products-content">
        <div class="clearflx">
          <div class="pro-title fl">热门推荐</div>
        </div>
        <div class="pro-item-box">
          <div class="pro-item" v-for="(item,index) in productsData" :key="index"
          @click="goDetail(item.pid)">
            <div class="pro-img">
              <img
                class="auto-img"
                :src="item.largeImg"
                alt=""
              />
            </div>
            <div class="pro-name text-eli">{{item.name}}</div>
            <div class="pro-enname text-eli">{{item.enname}}</div>
            <div class="pro-price">￥{{item.price}}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import "../assets/less/Home.less";
export default {
  data() {
    return {
      bannerData: [],
      productsData:[]
    };
  },
  created() {
    // 请求轮播图数据
    this.getBannerData();
    // 请求热门推荐的数据
    this.getProductsData();
  },
  methods: {
    // 请求轮播图数据
    getBannerData() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      this.axios({
        // 请求类型
        method: "GET",
        url: "/banner",
        // 接受的字符串类型
        params: {
          appkey: this.appkey,
        },
      })
        .then((res) => {
          this.$toast.clear();
          // 请求成功，返回的数据
          if (res.data.code === 300) {
            this.bannerData = res.data.result;
          }
        })
        .catch((err) => {
          this.$toast.clear();
          // 请求失败，返回失败err
        });
    },
    // 请求热门推荐的数据
    getProductsData() {
      this.axios({
        // 请求类型
        method: "GET",
        url: "/typeProducts",
        // 接受的字符串类型
        params: {
          appkey: this.appkey,
          key: "isHot",
          value: 1,
        },
      })
        .then((res) => {
          this.$toast.clear();
          // 请求成功，返回的数据
          if (res.data.code === 500) {
            this.productsData = res.data.result;
          }
        })
        .catch((err) => {
          this.$toast.clear();
          // 请求失败，返回失败err
        });
    },
    // 跳转详情页
    goDetail(pid){
      this.$router.push({name:"Detail",params:{pid}})
    }
  },
};
</script>