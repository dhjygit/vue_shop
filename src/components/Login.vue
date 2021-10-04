<template>
  <div class="login_container">
    <div class="login_box">
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="avatarImage">
      </div>
      <el-form class="form" :model="loginForm" :rules="loginFormRules" ref="loginFormRef">
        <el-form-item prop="username">
          <el-input prefix-icon="el-icon-user" v-model="loginForm.username"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input prefix-icon="el-icon-lock" v-model="loginForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item class="btns">
          <el-button type="primary" @click="loginValidate">登陆</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Login',
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [{
          required: true,
          message: '请输入用户名',
          trigger: 'blur'
        }, {
          min: 3,
          max: 10,
          message: '长度在 3 到 10 个字符',
          trigger: 'blur'
        }
        ],
        password: [{
          required: true,
          message: '请输入密码',
          trigger: 'blur'
        }, {
          min: 3,
          max: 10,
          message: '长度在 3 到 10 个字符',
          trigger: 'blur'
        }]
      }
    }
  },
  methods: {
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    loginValidate () {
      this.$refs.loginFormRef.validate(async (validate) => {
        // eslint-disable-next-line no-useless-return
        if (!validate) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登陆失败')
        this.$message.success('登陆成功')
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>
<style lang="scss" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  margin: auto;
  width: 450px;
  height: 300px;
  background-color: white;
  border-radius: 10px;

  .avatar_box {
    position: absolute;
    width: 130px;
    height: 130px;
    left: 0;
    right: 0;
    margin: auto;
    transform: translateY(-50%);
    border-radius: 50%;
    border: 1px solid #eee;
    padding: 10px;
    background-color: #fff;
    box-shadow: 0 0 10px #ddd;

    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }

  .form {
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 15px;
    box-sizing: border-box;

    .btns {
      display: flex;
      justify-content: flex-end;
    }
  }

}
</style>
