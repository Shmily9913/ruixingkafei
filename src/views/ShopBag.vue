<template>
  <div class="ShopBag">
    <!-- 导航栏 -->
    <van-nav-bar
      title="购物袋"
      left-text="返回"
      :right-text="ShopcartData.length <= 0 ? '' : isEdit ? '完成' : '编辑'"
      left-arrow
      @click-left="$router.back()"
      @click-right="isEdit = !isEdit"
      fixed
    />
    <!-- 空状态 -->
    <van-empty
      description="购物袋空空如也，去逛一逛!"
      v-if="ShopcartData.length <= 0"
    >
      <van-button
        round
        type="danger"
        class="bottom-button"
        @click="$router.push({ name: 'Home' })"
        >去逛一逛</van-button
      >
    </van-empty>
    <!-- 购物袋内容盒子 -->
    <div class="ShopBag-box" v-else>
      <!-- 顶部背景图 -->
      <div class="bg-box">
        <img class="auto-img" src="../assets/images/shopbag_bg.png" alt="" />
      </div>
      <!-- 购物袋的内容 -->
      <div class="ShopBag">
        <div
          class="pro-item"
          v-for="(item, index) in ShopcartData"
          :key="index"
        >
          <van-swipe-cell :disabled="!isEdit">
            <div class="ShopBag-item">
              <div class="item-fl">
                <van-checkbox
                  v-model="item.isActive"
                  @click="selectOne"
                  checked-color="#0b34ba"
                />
              </div>
              <div class="item-fr">
                <div class="item-img">
                  <img class="auto-img" :src="item.large_img" alt="" />
                </div>
                <div class="item-info">
                  <div class="item-name">
                    <div class="name">{{ item.name }}</div>
                    <div class="rule">{{ item.rule }}</div>
                  </div>
                  <div class="item-enname">{{ item.enname }}</div>
                  <div class="item-price clearflx">
                    <div class="price fl">￥{{ item.price }}</div>
                    <div class="fr">
                      <van-stepper
                        v-model="item.count"
                        theme="round"
                        button-size="22"
                        disable-input
                        @change="modifyCount(item.count, item.sid)"
                      />
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <template #right>
              <van-button
                square
                type="danger"
                text="删除"
                @click="deleteShopcart(index, item.sid)"
              />
            </template>
          </van-swipe-cell>
        </div>
      </div>
      <!-- 提交订单 -->
      <van-submit-bar
        :price="total"
        button-text="提交订单"
        v-if="!isEdit"
        @submit="payInfo()"
      >
        <van-checkbox
          v-model="isChecked"
          @click="allselect"
          checked-color="#0b34ba"
          >全选</van-checkbox
        >
      </van-submit-bar>
      <van-submit-bar
        button-text="删除选择"
        v-if="isEdit"
        @submit="deleteSelect"
      >
        <van-checkbox
          v-model="isChecked"
          @click="allselect"
          checked-color="#0b34ba"
          >全选</van-checkbox
        >
      </van-submit-bar>
    </div>
  </div>
</template>
<script>
import "../assets/less/shopBag.less";
export default {
  data() {
    return {
      // 全选
      isChecked: false,
      // 购物袋数据
      ShopcartData: [],
      // 是否处于编辑状态
      isEdit: false,
      // 总价
      total: 0,
    };
  },
  created() {
    this.findAllShopcart();
  },
  methods: {
    findAllShopcart() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      // 获取token
      let tokenString = localStorage.getItem("___tk");
      // 未登录(只验证有没有token，没有验证合法性)
      if (!tokenString) {
        return;
      }
      this.axios({
        // 请求类型
        method: "GET",
        url: "/findAllShopcart",
        // 接受的字符串类型
        params: {
          appkey: this.appkey,
          tokenString,
        },
      })
        .then((res) => {
          this.$toast.clear();
          if (res.data.code === 700) {
            this.$router.push({ name: "Login" });
          } else if (res.data.code === 5000) {
            res.data.result.map((v) => {
              v.isActive = false;
            });
            this.ShopcartData = res.data.result;
          }
        })
        .catch((err) => {
          this.$toast.clear();
          // 请求失败，返回失败err
        });
    },
    // 全选
    allselect() {
      this.ShopcartData.map((v) => {
        v.isActive = this.isChecked;
      });
      this.sum();
    },
    // 单选
    selectOne() {
      this.sum();
      for (let i = 0; i < this.ShopcartData.length; i++) {
        // 只要有一个false，取消全选
        if (!this.ShopcartData[i].isActive) {
          return (this.isChecked = false);
        }
      }
      // 所有商品都被选中
      this.isChecked = true;
    },
    // 删除单个
    // deleteOne(sid){
    //   // this.ShopcartData.splice(index,1)
    //   // this.sum()
    //   this.deleteShopcart([sid])
    // },
    // 删除选择
    deleteSelect() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      // 获取token
      let tokenString = localStorage.getItem("___tk");
      // 未登录(只验证有没有token，没有验证合法性)
      if (!tokenString) {
        return;
      }
      let isHas = false;
      let sid = [];
      this.ShopcartData.map((v) => {
        if (v.isActive) {
          isHas = true;
          sid.push(v.sid);
        }
      });

      if (!isHas) {
        return this.$toast("请选择删除的商品");
      }

      this.axios({
        // 请求类型
        method: "POST",
        url: "/deleteShopcart",
        // 接受的字符串类型
        data: {
          appkey: this.appkey,
          tokenString,
          sids: JSON.stringify([sid]),
        },
      })
        .then((res) => {
          this.$toast.clear();
          if (res.data.code === 700) {
            this.$router.push({ name: "Login" });
          } else if (res.data.code === 7000) {
            for (let i = 0; i < this.ShopcartData.length; i++) {
              if (this.ShopcartData[i].isActive) {
                isHas = true;
                this.ShopcartData.splice(i, 1);
                i--;
              }
            }
            this, (total = 0);
          }
        })
        .catch((err) => {
          this.$toast.clear();
          // 请求失败，返回失败err
        });
    },
    // 计算总价
    sum() {
      this.total = 0;
      this.ShopcartData.map((v) => {
        if (v.isActive) {
          // 小数点后两位 需要100
          this.total += v.price * v.count * 100;
        }
      });
    },
    // 修改商品数量
    modifyCount(count, sid) {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      // 获取token
      let tokenString = localStorage.getItem("___tk");
      // 未登录(只验证有没有token，没有验证合法性)
      if (!tokenString) {
        return;
      }
      this.axios({
        // 请求类型
        method: "POST",
        url: "/modifyShopcartCount",
        // 接受的字符串类型
        data: {
          appkey: this.appkey,
          tokenString,
          count,
          sid,
        },
      })
        .then((res) => {
          this.$toast.clear();
          if (res.data.code === 700) {
            this.$router.push({ name: "Login" });
          } else if (res.data.code === 6000) {
            this.sum();
          }
        })
        .catch((err) => {
          this.$toast.clear();
          // 请求失败，返回失败err
        });
    },
    // 删除
    deleteShopcart(index, sid) {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      // 获取token
      let tokenString = localStorage.getItem("___tk");
      // 未登录(只验证有没有token，没有验证合法性)
      if (!tokenString) {
        return;
      }
      this.axios({
        // 请求类型
        method: "POST",
        url: "/deleteShopcart",
        // 接受的字符串类型
        data: {
          appkey: this.appkey,
          tokenString,
          sids: JSON.stringify([sid]),
        },
      })
        .then((res) => {
          this.$toast.clear();
          if (res.data.code === 700) {
            this.$router.push({ name: "Login" });
          } else if (res.data.code === 7000) {
            this.ShopcartData.splice(index, 1);
            this.sum();
          }
        })
        .catch((err) => {
          this.$toast.clear();
          // 请求失败，返回失败err
        });
    },
    // 提交订单
    payInfo() {
      let isHas = false;
      let sid = []
      this.ShopcartData.map((v) => {
        if (v.isActive) {
          isHas = true;
          sid.push(v.sid)
        }
      });

      if (!isHas) {
        return this.$toast("请选择需要提交的商品");
      }

      this.$dialog
        .confirm({
          title: "订单",
          message: "是否提交订单",
        })
        .then(() => {
          this.$toast.loading({
            message: "加载中...",
            forbidClick: true,
            duration: 0,
          });
          // 获取token
          let tokenString = localStorage.getItem("___tk");
          // 未登录(只验证有没有token，没有验证合法性)
          if (!tokenString) {
            return;
          }
          this.axios({
            // 请求类型
            method: "POST",
            url: "/pay",
            // 接受的字符串类型
            data: {
              appkey: this.appkey,
              tokenString,
              sids:JSON.stringify(sid),
              phone:'12141351',
              address:'天空岛',
              receiver:'莹'
            },
          })
            .then((res) => {
              this.$toast.clear();
              if (res.data.code === 700) {
                this.$router.push({ name: "Login" });
              } else if (res.data.code === 60000) {
                this.$toast(res.data.msg)
                setTimeout(()=>{
                  this.$router.push({name:'Home'})
                },500)
              }
            })
            .catch((err) => {
              this.$toast.clear();
              // 请求失败，返回失败err
            });
        })
        .catch(() => {
          // on cancel
        });
    },
  },
};
</script>