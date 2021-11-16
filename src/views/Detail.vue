<template>
  <div class="detail">
    <van-nav-bar
      title="商品详情"
      left-text="返回"
      left-arrow
      @click-left="$router.back()"
    />
    <!-- 背景图片 -->
    <div class="bg-box">
      <img :src="detailData.large_img" alt="" class="auto-img" />
      <div class="img-name">{{ detailData.name }}</div>
    </div>
    <!-- 商品规格盒子 -->
    <div class="products-rule-box">
      <!-- 商品规格 -->
      <div class="pro-rule">
        <div
          class="pro-item"
          v-for="(item, index) in detailData.rules"
          :key="index"
        >
          <div class="item-name">{{ item.title }}</div>
          <div class="item-rule">
            <div
              class="rule"
              :class="{ active: item.currentIndex === i }"
              v-for="(n, i) in item.rule"
              :key="i"
              @click="toggleRule(item, i)"
            >
              {{ n }}
            </div>
          </div>
        </div>
      </div>
      <!-- 商品描述 -->
      <div class="pro-desc-box">
        <div class="desc-title">商品描述</div>
        <div class="desc-list">
          <div v-for="(item, index) in detailData.desc" :key="index">
            {{ `${index + 1}、${item}` }}
          </div>
        </div>
      </div>
      <!-- 价格 -->
      <div class="pro-price-box clearflx">
        <div class="pro-price fl">￥{{ detailData.price }}</div>
        <div class="fr">
          <van-stepper v-model="proCount" theme="round" button-size="22" disable-input />
        </div>
      </div>
    </div>
    <!-- 商品导航 -->
    <van-goods-action>
      <van-goods-action-icon
        :class="ShopCount > 0 ? '#0b34ba' : '#989898'"
        :badge="ShopCount > 0 ? ShopCount : ''"
        icon="bag"
        text="购物袋"
        @click="$router.push({name:'ShopBag'})"
      />
      <van-goods-action-icon
        :color="isLike ? '#0b34ba' : '#989898'"
        icon="like"
        :text="isLike ? '已收藏' : '未收藏'"
        @click="getLike"
      />
      <van-goods-action-button
        color="#587FCD"
        type="warning"
        text="加入购物袋"
        @click="addShopcart"
      />
      <van-goods-action-button color="#0b34ba" type="danger" text="立即购买" />
    </van-goods-action>
  </div>
</template>

<script>
import "../assets/less/detail.less";
export default {
  data() {
    return {
      pid: 0,
      detailData: {},
      // 是否收藏
      isLike: false,
      // 购物袋商品数量
      ShopCount:0,
      // 商品的数量
      proCount:0
    };
  },
  created() {
    this.pid = this.$route.params.pid;
    // 商品详情信息
    this.getDetailData();
    this.findLike();
    this.getshopcartRows()
  },
  methods: {
    getDetailData() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      this.axios({
        // 请求类型
        method: "GET",
        url: "/productDetail",
        // 接受的字符串类型
        params: {
          appkey: this.appkey,
          pid: this.pid,
        },
      })
        .then((res) => {
          this.$toast.clear();
          // 请求成功，返回的数据
          if (res.data.code === 600) {
            // 处理备注信息
            let data = res.data.result[0];
            data.desc = data.desc.split(/\n/);
            // 处理数据
            let rules = [];
            let ruleData = ["cream", "milk", "tem", "sugar"];
            ruleData.map((v) => {
              let r = {};
              r.title = data[v + "_desc"]; // {title:"温度"},{title:"糖"}...
              r.currentIndex = 0;
              r.rule = []; // {title:"温度",rule:[]}...
              let ruleString = data[v].split("/"); // {title:"温度",rule:['冷','热']}...
              ruleString.map((o) => {
                if (!o == "") {
                  r.rule.push(o);
                }
              });
              if (r.rule.length > 0) {
                rules.push(r); // [{title:"温度",rule:['冷','热']},{},{}]
              }
            });
            data.rules = rules;
            this.detailData = data;
          }
        })
        .catch((err) => {
          this.$toast.clear();
          // 请求失败，返回失败err
        });
    },
    // 切换规格
    toggleRule(item, i) {
      item.currentIndex = i;
    },
    // 收藏接口
    getLike() {
      // 获取token
      let tokenString = localStorage.getItem("___tk");
      // 未登录(只验证有没有token，没有验证合法性)
      if (!tokenString) {
        this.$toast("请先登录!");
        return this.$router.push({ name: "Login" });
      }
      let url = this.isLike ? "/notlike" : "like";
      this.axios({
        method: "POST",
        url,
        data: {
          appkey: this.appkey,
          pid: this.pid,
          tokenString,
        },
      })
        .then((res) => {
          this.$toast(res.data.msg);
          if (res.data.code === 800) {
            this.isLike = true;
          } else if (res.data.code === 900) {
            this.isLike = false;
          }
          // 请求成功，返回的数据
        })
        .catch((err) => {
          // 请求成功，返回失败err
        });
    },
    // 查询商品是否被收藏
    findLike() {
      // 获取token
      let tokenString = localStorage.getItem("___tk");
      // 未登录
      if (!tokenString) {
        return;
      }
      this.axios({
        method: "GET",
        url: "/findlike",
        params: {
          appkey: this.appkey,
          pid: this.pid,
          tokenString,
        },
      })
        .then((res) => {
          if (res.data.code === 700) {
            this.$toast(res.data.msg);
            // 非法token，跳转登录，重新登录
            this.$router.push({ name: "Login" });
          } else if (res.data.code === 1000 && res.data.result.length > 0) {
            this.isLike = true;
          }
          // 请求成功，返回的数据
        })
        .catch((err) => {
          // 请求成功，返回失败err
        });
    },
    // 查询购物车商品条目
    getshopcartRows() {
      let tokenString = localStorage.getItem("___tk");
      // 未登录
      if (!tokenString) {
        return;
      }
      this.axios({
        method: "GET",
        url: "/shopcartRows",
        params: {
          appkey: this.appkey,
          tokenString
        },
      })
        .then((res) => {
          // 请求成功，返回的数据
          if (res.data.code === 700) {
            this.$toast(res.data.msg);
            // 非法token，跳转登录，重新登录
            this.$router.push({ name: "Login" });
          }else if(res.data.code === 8000){
            this.ShopCount = res.data.result
          }
          // 请求成功，返回的数据
        })
        .catch((err) => {
          // 请求成功，返回失败err
        });
    },
    // 将商品加入购物车
    addShopcart(){
      // 获取token
      let tokenString = localStorage.getItem("___tk");
      // 未登录(只验证有没有token，没有验证合法性)
      if (!tokenString) {
        this.$toast("请先登录!");
        return this.$router.push({ name: "Login" });
      }
      // 处理规格的数据
      let rule = this.detailData.rules.map(v=>{
        return v.rule[v.currentIndex]
      })
      rule = rule.join('/')
      this.axios({
        method: "POST",
        url:"/addShopcart",
        data: {
          appkey: this.appkey,
          pid: this.pid,
          count:this.proCount,
          tokenString,
          rule
        },
      })
        .then((res) => {
        //  请求成功，返回的数据
          if (res.data.code === 700) {
             this.$toast(res.data.msg);
             // 非法token，跳转登录，重新登录
             this.$router.push({ name: "Login" });
          } else if (res.data.code === 3000) {
            if(res.data.status === 1){
              this.$toast(res.data.msg);
              this.ShopCount++
            }else{
              this.$toast("购物袋存在该商品，商品数量已更新!");
            }
          }
        })
        .catch((err) => {
          // 请求成功，返回失败err
        });
    }
  },
};
</script>

<style>
</style>