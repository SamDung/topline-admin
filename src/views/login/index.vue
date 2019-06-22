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
export default {
  name: 'AppLogin',

  data () {
    return {
      form: {
        mobile: '18202400751',
        code: ''
      }
    }
  },

  methods: {
    onSubmit () {
      console.log('submit!')
    },
    handleSendCode () {
    //   结构赋值？
      const { mobile } = this.form
      axios({
        method: 'GET',
        // 什么意思？
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
      }).then(res => {
        console.log(res.data)
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
