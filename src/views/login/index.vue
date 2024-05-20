<template>
  <div class="login-container">
    <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form" autocomplete="on" label-position="left">

      <div class="title-container">
        <h3 class="title">Login Form</h3>
      </div>

      <el-form-item prop="phonenumber">
        <span class="svg-container">
          <i class="el-icon-mobile-phone" />
        </span>
        <el-input
          ref="phonenumber"
          v-model="loginForm.phonenumber"
          placeholder="Phonenumber"
          name="phonenumber"
          type="text"
          tabindex="1"
        />
      </el-form-item>

      <el-form-item prop="msgcode">
        <span class="svg-container">
          <svg-icon icon-class="password" />
        </span>
        <el-input
          ref="msgcode"
          v-model="loginForm.msgcode"
          class="clear-number-input"
          type="number"
          placeholder="请输入验证码"
          name="msgcode"
          tabindex="2"
          maxlength="4"
          style="width: 60%"
          on-keypress="return(/[\d]/.test(String.fromCharCode(event.keyCode)))"
          @input.native="changeNum"
        />
        <el-button
          style="width: 30%"
          :disabled="loginForm.codeTime !== 0 && sendButtonIsDisabled"
          :loading="buttonLoading"
          @click="sendCodeFunc"
        >
          {{ getButtonTxt() }}
        </el-button>
      </el-form-item>
      <el-button :loading="loading" type="primary" style="width:100%;margin-bottom:30px;" @click.native.prevent="handleLogin">Login</el-button>

    </el-form>

    <el-dialog title="Or connect with" :visible.sync="showDialog">
      Can not be simulated on local, so please combine you own business simulation! ! !
      <br>
      <br>
      <br>
      <social-sign />
    </el-dialog>
  </div>
</template>

<script>
import { validPhonenumber } from '@/utils/validate'
export default {
  data() {
    const validatePhonenumber = (rule, value, callback) => {
      if (!validPhonenumber(value)) {
        callback(new Error('请输入正确的手机号'))
      } else {
        callback()
      }
    }
    const validateMsgcode = (rule, value, callback) => {
      if (value.length < 6) {
        callback(new Error('请输入短信内的验证码'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        phonenumber: '18210512166',
        msgcode: '888666',
        codeTime: 0
      },
      loginRules: {
        phonenumber: [{ required: true, trigger: 'blur', validator: validatePhonenumber }],
        msgcode: [{ required: true, trigger: 'blur', validator: validateMsgcode }]
      },
      capsTooltip: false,
      loading: false,
      buttonLoading: false,
      isSendCode: false,
      showDialog: false,
      redirect: undefined,
      otherQuery: {},
      tips: {
        success: '验证码发送成功！',
        fail: '验证码发送失败！',
        coolingTime: '请耐心等待验证码！'
      },
      // 发送验证码期间是否禁用
      sendButtonIsDisabled: false
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        const query = route.query
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    }
  },
  created() {
  },
  mounted() {
    if (this.loginForm.phonenumber === '') {
      this.$refs.phonenumber.focus()
    } else if (this.loginForm.msgcode === '') {
      this.$refs.msgcode.focus()
    }
  },
  destroyed() {
  },
  methods: {
    alert(msg) {
      this.$notify({
        title: '警告',
        message: msg,
        type: 'warning'
      })
    },
    info(msg) {
      this.$notify.info({
        title: '消息',
        message: msg
      })
    },

    sendCodeFunc() {
      const { fail, success, coolingTime } = this.tips
      console.info('fail:' + fail + ' success:' + success + ' coolingTime:' + coolingTime + 'codeTime:' + this.loginForm.codeTime)
      if (this.loginForm.codeTime !== 0) {
        this.alert(coolingTime)
        return false
      }
      // 手机号校验
      this.$refs.loginForm.validateField('phonenumber', (errorMessage) => {
        const valid = errorMessage === ''
        if (valid) {
          this.buttonLoading = true
          console.log('submit!!')
          this.$store.dispatch('user/sendMsgCode', this.loginForm.phonenumber)
            .then(() => {
              this.loading = false
              this.buttonLoading = false
              // 时间开始
              this.loginForm.codeTime = 10
              // 定时器
              const time = () =>
                setTimeout(() => {
                  this.loginForm.codeTime -= 1
                  if (this.loginForm.codeTime !== 0) {
                    time()
                  }
                }, 1000)
              // 执行定时器
              time()
              this.isSendCode = true
              this.info('短信已发送，请注意查收')
            })
            .catch(() => {
              this.loading = false
              this.buttonLoading = false
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getButtonTxt() {
      if (this.loginForm.codeTime !== 0) {
        return this.loginForm.codeTime
      } else if (this.buttonLoading) {
        return '发送中'
      } else if (this.isSendCode) {
        return '重新发送'
      } else {
        return '发送验证码'
      }
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
          this.$store.dispatch('user/login', this.loginForm)
            .then(() => {
              this.$router.push({ path: this.redirect || '/', query: this.otherQuery })
              this.loading = false
            })
            .catch(() => {
              this.loading = false
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    },
    changeNum() {
      console.info('changeNum')
      if (this.loginForm.msgcode.length > 6) {
        this.loginForm.msgcode = this.loginForm.msgcode.slice(0, 6)
      }
    }
  }
}
</script>

<style lang="scss">
/* 设置全局 */
.clear-number-input.el-input::-webkit-outer-spin-button,
.clear-number-input.el-input::-webkit-inner-spin-button {
  margin: 0;
  -webkit-appearance: none !important;
}
.clear-number-input.el-input input[type="number"]::-webkit-outer-spin-button,
.clear-number-input.el-input input[type="number"]::-webkit-inner-spin-button {
  margin: 0;
  -webkit-appearance: none !important;
}
.clear-number-input.el-input {
  -moz-appearance: textfield;
}
.clear-number-input.el-input input[type="number"] {
  -moz-appearance: textfield;
}

/* 修复input 背景不协调 和光标变色 */
/* Detail see https://github.com/PanJiaChen/vue-element-admin/pull/927 */

$bg:#283443;
$light_gray:#fff;
$cursor: #fff;

@supports (-webkit-mask: none) and (not (cater-color: $cursor)) {
  .login-container .el-input input {
    color: $cursor;
  }
}

/* reset element-ui css */
.login-container {
  .el-input {
    display: inline-block;
    height: 47px;
    width: 85%;

    input {
      background: transparent;
      border: 0px;
      -webkit-appearance: none;
      border-radius: 0px;
      padding: 12px 5px 12px 15px;
      color: $light_gray;
      height: 47px;
      caret-color: $cursor;

      &:-webkit-autofill {
        box-shadow: 0 0 0px 1000px $bg inset !important;
        -webkit-text-fill-color: $cursor !important;
      }
    }
  }

  .el-form-item {
    border: 1px solid rgba(255, 255, 255, 0.1);
    background: rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    color: #454545;
  }
}
</style>

<style lang="scss" scoped>
$bg:#2d3a4b;
$dark_gray:#889aa4;
$light_gray:#eee;

.login-container {
  min-height: 100%;
  width: 100%;
  background-color: $bg;
  overflow: hidden;

  .login-form {
    position: relative;
    width: 520px;
    max-width: 100%;
    padding: 160px 35px 0;
    margin: 0 auto;
    overflow: hidden;
  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 26px;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }

  .thirdparty-button {
    position: absolute;
    right: 0;
    bottom: 6px;
  }

  @media only screen and (max-width: 470px) {
    .thirdparty-button {
      display: none;
    }
  }
}
</style>
