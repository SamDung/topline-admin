<template>
  <div class="login-wrap">
    <div class="login-form-wrap">
       <div class="login-head">
           <img src="./logo_index.png" alt="黑马头条">
       </div>
       <div class="login-form">
        <el-form ref="form" :model="form">
        <el-form-item>
          <el-input v-model="form.mobile" placeholder="手机号"></el-input>
        </el-form-item>
        <el-form-item>
          <el-col :span = "10">
              <el-input v-model="form.code" placeholder="验证码"></el-input>
          </el-col>
          <el-col :span = "10" :offset = "2">
                <el-button @click="handleSendCode">获取验证码</el-button>
          </el-col>
        </el-form-item>
        <el-button class="btn" type="primary" @click="onSubmit">登录</el-button>
        </el-form>
       </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import '@/vendor/gt'
export default {
  name: 'AppLogin',

  data () {
    return {
      form: {
        mobile: '18202400751',
        code: ''
      },
      captchaObj: null //  目的：点击发送验证码滑动框瞬间弹出不换图片，设置初始值    这是通过 initGEEt 得到的极验的验证码对象
    }
  },

  methods: {
    onSubmit () {
      console.log('submit!')
    },
    handleSendCode () {
    //   结构赋值？
      const { mobile } = this.form

      // 如果有captchobj对象 那么就直接让它弹出来
      if (this.captchaObj) {
        return this.captchaObj.verify()
      }
      axios({
        method: 'GET',
        // 什么意思？
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
      }).then(res => {
        // console.log(res.data)
        const data = res.data.data
        // 需要加上全局 init是极验后端给提供的一个方法
        window.initGeetest({
          gt: data.gt,
          challenge: data.challenge,
          offline: !data.success,
          new_captcha: true,
          product: 'bind'
        }, (captchaObj) => {
          this.captchaObj = captchaObj
          // 写到这了确实可以验证极验会给你返回一个captch对象但是看不见
          captchaObj.onReady(function () {
            // 只有ready了才能显示验证码
            captchaObj.verify()
          }).onSuccess(function () {
            // console.log('极验验证成功')
            console.log(captchaObj.getValidate())
            // 将cap对象结构 作为三个参数
            const {
              geetest_challenge: challenge,
              geetest_seccode: seccode,
              geetest_validate: validate } =
              captchaObj.getValidate()
            axios({
              method: 'GET',
              url: `http://ttapi.research.itcast.cn/mp/v1_0/sms/codes/${mobile}`,
              params: {
                challenge,
                validate,
                seccode
              }
            }).then(res => {
              console.log(res.data)
              // 开启倒计时效果
            })
          })
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login-wrap{
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #ccc;
    .login-head {
        display: flex;
        justify-content: center;
        margin-bottom: 10px;
        img {
            width: 200px;
        }
    }
    .login-form-wrap {
        background-color: #fff;
        padding:50px;
    }
    .btn {
        width: 100%;
    }
}
</style>
