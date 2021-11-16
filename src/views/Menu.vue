<template>
  <div class="Menu">
    <!-- 导航栏 -->
    <van-nav-bar left-text="返回" left-arrow fixed>
      <template #left>
        <van-search placeholder="请输入商品名称" />
      </template>
    </van-nav-bar>
    <!-- 商品类型 -->
    <div class="pro-type">
      <div class="type-item">
        <div>
          <img src="../assets/images/icons_21.gif" alt="">
        </div>
        <div class="active">推荐</div>
      </div>
      <div class="type-item">
        <div>
          <img src="../assets/images/icons_21.gif" alt="">
        </div>
        <div class="active">拿铁</div>
      </div>
      <div class="type-item">
        <div>
          <img src="../assets/images/icons_21.gif" alt="">
        </div>
        <div class="active">咖啡</div>
      </div>
      <div class="type-item">
        <div>
          <img src="../assets/images/icons_21.gif" alt="">
        </div>
        <div class="active">冰</div>
      </div>
    </div>
    <!-- 商品 -->
    <div class="pro-type-box">
      <div class="pro-item" v-for="item in 10" :key="item">
        <div class="pro-img"></div>
        <div class="pro-info clearflx">
          <div class="info-name fl">
            <div class="name">焦糖</div>
            <div class="enname">冷</div>
          </div>
          <div class="info-price fr">￥11.00</div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import '../assets/less/menu.less'
export default {
  
}
</script>