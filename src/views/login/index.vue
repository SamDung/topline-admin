<template>
  <div class="login-wrap">
    <div class="login-form-wrap">
       <div class="login-head">
           <img src="./logo_index.png" alt="黑马头条">
       </div>
       <div class="login-form">
         <!-- rules配置验证规则 将需要验证的字段通过prop属性配置到组件上
         ref 获取表单组件 可以手动调用表单组件的验证方法
          -->
        <el-form :model="form" :rules="rules" ref="ruleForm">
          <el-form-item prop="mobile">
            <el-input v-model="form.mobile" placeholder="手机号"></el-input>
          </el-form-item>
          <el-form-item prop="code">
            <el-col :span = "10">
              <el-input v-model="form.code" placeholder="验证码"></el-input>
            </el-col>
            <el-col :span = "10" :offset = "2">
                <el-button @click="handleSendCode">获取验证码</el-button>
            </el-col>
          </el-form-item>
          <el-form-item prop="agree">
            <el-checkbox v-model="form.agree"></el-checkbox>
            <span>我已阅读并同意<a href="#">用户协议</a>和<a href="#">隐私条款</a></span>
          </el-form-item>
          <el-button class="btn" type="primary" @click="handleLogin" :loading="loginLoading">登录</el-button>
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
        code: '',
        agree:''
      },
      loginLoading: false,
      rules: {
        //通过父标签设置rules规则然后通过子标签的prop传递
        mobile: [
          { required: true, message: '请输入电话号', trigger: 'blur' },
          { min: 11, max: 11, message: '长度在11个字符', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 6, message: '长度在6个字符', trigger: 'blur' }
        ],
        agree: [
          { required: true, message: '请同意用户协议', trigger: 'change' },
          { pattern: /true/, message: '请同意用户协议', trigger: 'change' }
        ]
      },
      captchaObj: null //  目的：点击发送验证码滑动框瞬间弹出不换图片，设置初始值    这是通过 initGEEt 得到的极验的验证码对象
    }
  },

  methods: {
    // 最后一步输入验证码点击登录
    handleLogin () {
      this.loading = true
      // 表单组件有一个方法validata 可以用于获取g表单的状态  说白了就是通过与否
      this.$refs['ruleForm'].validate(valid => {
        // 判断如果！valid 那么验证失败
        if (!valid) {
          return
        }
        this.login()
      })
    },
    // 封装的提交登录
    login () {
      axios({
        method: 'POST',
        url: 'http://ttapi.research.itcast.cn/mp/v1_0/authorizations',
        data: this.form
      }).then(res => {
        this.loading = false
        this.$router.push({
          name: 'home'
        })
      }).catch(err => {
        if (err.response.status === 400) {
          this.$message.error('登录失败')
        }
        this.loading = false
      })
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
          // 写到这了确实可以验证极验会给你返回一个captch对象但是看不见图片
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
              // element提供的一种message消息组件
              // console.log(res.data) 此时给你返回一组id...name...的数据
              // this.$router.push({
              //   name: 'home'
              // })  // this.$router.pusn('/')
              console.log(res.data)
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
