<template>
  <div class="login-wrap">
    <div class="login-form-wrap">
      <div class="login-head">
        <img src="./logo_index.png" alt="黑马头条">
      </div>
      <div class="login-form">
        <!--
          表单验证：
          rules 配置验证规则
          将需要验证的字段通过 prop 属性配置到 el-form-item 组件上
          ref   获取表单组件，可以手动调用表单组件的验证方法
         -->
        <el-form :model="form" :rules="rules" ref="ruleForm">
          <el-form-item prop="mobile">
            <el-input v-model="form.mobile" placeholder="手机号"></el-input>
          </el-form-item>
          <el-form-item prop="code">
            <el-input class="input-code" v-model="form.code" placeholder="验证码"></el-input>
            <el-button
              class="btn-code"
              @click="handleSendCode"
              :disabled="!!codeTimer"
            >{{ codeTimer ? `剩余${countDown}秒` : '获取验证码' }}</el-button>
          </el-form-item>
          <el-form-item>
            <!-- 给组件加 class，会作用到它的根元素 -->
            <el-button
              class="btn-login"
              type="primary"
              @click="handleLogin"
              :loading="loginLoading"
            >登录</el-button>
          </el-form-item>
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
      form: { // 表单数据
        mobile: window.localStorage.getItem('mobile') || '',
        code: ''
      },
      loginLoading: false, // 登录按钮的 loading 状态
      codeTimer: null,
      countDown: 10,
      rules: { // 表单验证规则
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { len: 11, message: '长度必须为11个字符', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入验证码', trigger: 'blur' },
          { len: 6, message: '长度必须为6个字符', trigger: 'blur' }
        ]
      },
      sendMobile: '',
      captchaObj: null // 通过 initGeetest 得到的极验验证码对象
    }
  },

  created () {
    // 检查本地存储的倒计时
    const codeTimer = window.localStorage.getItem('codeTimer')

    if (codeTimer) {
      const goneTime = Number.parseInt((+new Date() - codeTimer) / 1000)
      if (goneTime < 10) {
        this.countDown = 10 - goneTime
        this.codeCountDown()
      } else {
        window.localStorage.removeItem('codeTimer')
      }
    }
  },

  methods: {
    handleLogin () {
      // 表单组件有一个方法 validate 可以用于获取当前表单的验证状态
      this.$refs['ruleForm'].validate(valid => {
        if (!valid) {
          return
        }

        // 表单验证通过，提交登录
        this.submitLogin()
      })
    },

    submitLogin () {
      this.loginLoading = true
      axios({
        method: 'POST',
        url: 'http://ttapi.research.itcast.cn/mp/v1_0/authorizations',
        data: this.form
      }).then(res => { // >= 200 && < 400 的状态码都会进入这里
        // Element 提供的 Message 消息提示组件，这也是组件调用的一种形式
        this.$message({
          message: '登录成功',
          type: 'success'
        })

        this.loginLoading = false

        // 建议路由跳转都使用 name 去跳转，路由传参非常方便
        this.$router.push({
          name: 'home'
        })
      }).catch(err => { // >= 400 的 HTTP 状态码都会进入 catch 中
        if (err.response.status === 400) {
          this.$message.error('登录失败，手机号或验证码错误')
        }
        this.loginLoading = false
      })
    },

    handleSendCode () {
      // 1. 校验手机号是否有效
      this.$refs['ruleForm'].validateField('mobile', errorMessage => {
        if (errorMessage) {
          return
        }

        // 2. 手机号有效，初始化人机交互验证码
        this.showGeetest()
      })
    },

    codeCountDown () {
      this.codeTimer = setInterval(() => {
        this.countDown--
        if (this.countDown <= 0) {
          window.clearInterval(this.codeTimer)
          this.codeTimer = null
          this.countDown = 10
          window.localStorage.removeItem('codeTimer')
        }
      }, 1000)
    },

    showGeetest () {
      if (this.captchaObj) {
        // 检查当前手机号和自上次初始化后的号码是否不一致
        if (this.form.mobile !== this.sendMobile) {
          // 如果不一致，则重新初始化
          // 将之前的验证码 DOM 删除
          document.body.removeChild(document.querySelector('.geetest_panel'))
          this.initGeetest()
        } else {
          // 如果一致，则直接显示之前的
          this.captchaObj.verify()
        }
      } else {
        this.initGeetest()
      }
    },

    initGeetest () {
      axios({
        method: 'GET',
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${this.form.mobile}`
      }).then(res => {
        const data = res.data.data
        window.initGeetest({
          // 以下配置参数来自服务端 SDK
          gt: data.gt,
          challenge: data.challenge,
          offline: !data.success,
          new_captcha: data.new_captcha,
          product: 'bind' // 隐藏按钮式
        }, (captchaObj) => {
          this.captchaObj = captchaObj
          // 这里可以调用验证实例 captchaObj 的实例方法
          captchaObj.onReady(() => {
            this.sendMobile = this.form.mobile

            // 只有 ready 了才能显示验证码
            captchaObj.verify()
          }).onSuccess(this.sendCode)
        })
      })
    },

    sendCode () {
      const {
        geetest_challenge: challenge,
        geetest_seccode: seccode,
        geetest_validate: validate } =
      this.captchaObj.getValidate()

      // 调用 获取短信验证码 (极验 API2）接口，发送短信
      axios({
        method: 'GET',
        url: `http://ttapi.research.itcast.cn/mp/v1_0/sms/codes/${this.form.mobile}`,
        params: { // 专门用来传递 query 查询字符串参数
          challenge,
          seccode,
          validate
        }
      }).then(res => {
        // 交互成功，倒计时
        window.localStorage.setItem('codeTimer', +new Date())
        window.localStorage.setItem('mobile', this.form.mobile)
        this.codeCountDown()
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
  .login-form-wrap {
    background-color: #fff;
    padding: 50px;
    border-radius: 10px;
    width: 350px;
    .login-head {
      display: flex;
      justify-content: center;
      margin-bottom: 10px;
      img {
        width: 200px;
      }
    }
    .input-code {
      display: inline-block;
      width: 60%;
    }
    .btn-code {
      float: right;
    }
    .btn-login {
      width: 100%;
    }
  }
}
</style>
