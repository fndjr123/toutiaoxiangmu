<template>
  <div class="login-wrap">
    <div class="form-wrap">
      <div class="form-head">
        <img src="./logo_index.png" alt="黑马头条">
      </div>
      <el-form class="form-conter" :model="form"  ref="form">
        <el-form-item>
          <el-input v-model="form.mobile" placeholder="手机号"></el-input>
        </el-form-item>
        <el-form-item>
          <el-col :span="14">
            <el-input v-model="form.code" placeholder="验证码"></el-input>
          </el-col>
          <el-col :offset="1" :span="9">
            <el-button @click="handleSendCode">获取验证码</el-button>
          </el-col>
        </el-form-item>
        <el-form-item>
          <el-button class="btn-login" type="primary" @click="onSubmit">登录</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
export default {
  name: 'AppLome',
  data () {
    return {
      form: {
        mobile: '',
        code: ''
      }
    }
  },
  methods: {
    onSubmit() {
      console.log('submit!')
    },
    handleSendCode() {
      // console.log(123)
      const { mobile } = this.form
      axios({
        method: 'GET',
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
      }).then(res => {
        console.log(res.data)
      })
    }
  }
}
</script>
<style lang="less" scoped>
.login-wrap {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #ccc;
  .form-wrap {
    background-color: #fff;
    padding: 10px;
    border-radius: 10px;
    width: 350px;
    .form-head {
      display: flex;
      justify-content: center;
      margin-bottom: 10px;
    }
    .form-conter {
      display: inline-block;
      width: 100%;
    }
    .btn-login{
      width: 100%;
    }
  }
}
</style>
