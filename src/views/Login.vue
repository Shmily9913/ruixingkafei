<template>
  <div class="login">
    <!-- 导航栏 -->
    <van-nav-bar left-text="返回" right-text="先逛一逛" @click-right="$router.push({name:'Home'})">
      <template #left>
        <div class="logo-box">
          <div class="logo"></div>
          <div class="logo-name">Luckin Coffee</div>
        </div>
      </template>
    </van-nav-bar>
    <!-- 用户 -->
    <div class="welcome-box">
      <h2 class="welcome">欢迎回来!</h2>
      <p class="welcome-en">Please login to your accounts</p>
    </div>
    <!-- 用户登录 -->
    <van-form>
      <van-field
        v-model="userLoginInfo.phone"
        name="手机号"
        label="手机号"
        placeholder="手机号"
      />
      <van-field
        v-model="userLoginInfo.password"
        :type="loginPswShow ? '' : 'password'"
        name="密码"
        label="密码"
        placeholder="密码"
        :right-icon="loginPswShow ? 'eye-o' : 'closed-eye'"
        @click-right-icon="loginPswShow = !loginPswShow"
      />
      <div class="forget">
        <span>忘记密码?</span>
      </div>
      <div class="vant-btn">
        <van-button round block type="info" color="#0b34ba" @click="userLogin">登录</van-button>
      </div>
      <div class="vant-btn register-btn" @click="registerShow = true">
        <van-button round block type="default">注册</van-button>
      </div>
    </van-form>
    <!-- 注册弹出层 -->
    <van-popup v-model="registerShow" round position="bottom" closeable>
      <div class="register-title">注册</div>
      <van-form>
        <van-field
          v-model="userRegisterInfo.phone"
          name="手机号"
          label="手机号"
          placeholder="手机号"
        />
        <van-field
          v-model="userRegisterInfo.nickName"
          name="昵称"
          label="昵称"
          placeholder="1-10位昵称"
        />
        <van-field
          v-model="userRegisterInfo.password"
          :type="registerPswShow ? '' : 'password'"
          name="密码"
          label="密码"
          placeholder="6-10位密码且以字母开头"
          :right-icon="registerPswShow ? 'eye-o' : 'closed-eye'"
          @click-right-icon="registerPswShow = !registerPswShow"
        />
        <div class="vant-btn">
          <van-button round block type="info" color="#0b34ba" @click="register"
            >注册</van-button
          >
        </div>
      </van-form>
    </van-popup>
  </div>
</template>
<script>
// 引入样式
import "../assets/less/login.less";
import validForm from "../assets/js/validForm"
export default {
  data() {
    return {
      // 显示隐藏登录的密码
      loginPswShow: false,
      // 显示隐藏注册的密码
      registerPswShow:false,
      // 显示隐藏注册页面
      registerShow: false,
      // 用户登录信息
      userLoginInfo: {
        phone: "",
        password: "",
      },
      // 用户注册信息
      userRegisterInfo: {
        phone: "",
        nickName:"",
        password: ""
      },
    };
  },
  methods:{
    register(){
      // 构造表单的验证信息
      let o = {
         phone:{
          // 获取手机号
          value:this.userRegisterInfo.phone,
          // 手机号验证错误提醒的文本
          errorMsg:"手机号码格式不正确",
          // 正则表达式验证手机号码是否符合要求
          reg:/^1[3-9]\d{9}$/
        },
        nickName:{
          value:this.userRegisterInfo.nickName,
          errorMsg:"昵称格式化不正确",
          reg:/^[\w\u4e00-\u9fa5]{1,10}$/
        },    
        password:{
          value:this.userRegisterInfo.password,
          errorMsg:"密码格式化不正确",
          reg:/^[A-Za-z]\w{5,15}$/
        }
      }
      // 验证格式匹配，请求注册
      if(validForm.Valid(o)){
        var userInfo = Object.assign({},this.userRegisterInfo)
        userInfo.appkey = this.appkey
        // 参数序列化
        // {
        //   appkey:123,
        //   nickName:123,
        //   phone:123,
        //   password:123
        // }
        // ?appkey=123&nickName=名字&phone=11&password=123
        // let paramsString = ''
        // for(let key in userInfo){
        //   paramsString += `${key}=${userInfo[key]}&`
        // }
        // paramsString = paramsString.slice(0,-1)
        // 显示加载中
        this.$toast.loading({
          // 加载的文本
          message:"注册成功!加载中...",
          // 防穿透
          forbidClick:true,
          // 显示的时间
          duration:0
        })
        this.axios({
          // 请求类型
          method:'POST',
          // 请求地址
          url:"/register",
          // 接受的字符串类型
          data:userInfo
        }).then(res =>{
          // 请求成功,返回的数据
          
          if(res.data.code === 100){
            // 隐藏注册页面
            this.registerShow = false
            this.$toast.clear();
          }else{
            this.$toast(res.data.msg)
          }
        }).catch(err =>{
          // 请求失败,返回的数据
        })
      }
     
    },
    userLogin(){
      // 构造表单的验证信息
      let o = {
         phone:{
          // 获取手机号
          value:this.userLoginInfo.phone,
          // 手机号验证错误提醒的文本
          errorMsg:"手机格式化不正确",
          // 正则表达式验证手机号码是否符合要求
          reg:/^1[3-9]\d{9}$/
        },   
        password:{
          value:this.userLoginInfo.password,
          errorMsg:"密码格式化不正确",
          reg:/^[A-Za-z]\w{5,15}$/
        }
      }
      // 验证格式匹配，请求注册
      if(validForm.Valid(o)){
        var userInfo = Object.assign({},this.userLoginInfo)
        userInfo.appkey = this.appkey

        
        // 显示加载中
        this.$toast.loading({
          // 加载的文本
          message:"登录中...",
          // 防穿透
          forbidClick:true,
          // 显示的时间
          duration:0
        })
        this.axios({
          // 请求类型
          method:'POST',
          // 请求地址
          url:"/login",
          // 接受的字符串类型
          data:userInfo
        }).then(res =>{
          // 请求成功,返回的数据
          if(res.data.code === 200){
            this.$toast.clear()
            // 将token保存在本地浏览器，方便下一次登录
            localStorage.setItem("___tk",res.data.token)
            // 登录成功跳转首页
            this.$router.push({name:"Home"})
          }else{
            this.$toast(res.data.msg)
          }
        }).catch(err =>{
          // 请求失败,返回的数据
        })
      }
     
    }
  }
};
</script>