<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="~assets/img/logo.png" />
      </div>
      <!-- 登录区域 login  model-数据源 rules-表单验证规则 ref-当前组件的引用 -->
      <el-form
        ref="loginFormRef"
        :rules="loginFormRules"
        label-width="0px"
        class="login_form"
        :model="loginForm"
      >
        <!-- userName  prop- 对当前输入框验证 -->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="el-icon-s-custom"></el-input>
        </el-form-item>
        <!-- passWord 1102-->
        <el-form-item prop="password">
          <el-input
            v-model="loginForm.password"
            prefix-icon="el-icon-lock"
            type="password"
          ></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //登录数据
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      //表单校验规则
      loginFormRules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 10,
            message: '长度在 3-10 个字符',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 5,
            max: 18,
            message: '长度在 5-18 个字符',
            trigger: 'blur'
          }
        ]
      },
      //表单验证通过时变为true
      valid: false,
      token: ''
    }
  },
  methods: {
    //登录
    async login() {
      this.$refs.loginFormRef.validate(valid => {
        this.valid = valid
      })
      //表单验证不通过直接退出函数
      if (!this.valid) return
      //发送登录验证请求
      let { data: res } = await this.$http.post('login', this.loginForm)
      if (res.meta.status !== 200) return this.$Message.error('亲~ 用户名或密码错误,请重新输入！')
      //登录成功的操作 在会话层保存token值,发送路由跳转
      this.token = res.data.token
      //在sessionStorage保存当前token
      window.sessionStorage.setItem('token', this.token)
      //登录成功跳转路由
      this.$router.push('/home')
      this.$Message.success('亲~ 登录成功! ')
    },
    // 点击重置按钮 重置登录表单
    resetLoginForm() {
      this.$refs.loginFormRef.resetFields()
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #409eff;
  height: 100%;
  overflow-x: hidden;
}
.login_box {
  height: 300px;
  width: 450px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
  }
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eee;
  }
}
.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
