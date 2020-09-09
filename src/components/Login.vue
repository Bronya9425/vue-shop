<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt />
      </div>
      <!-- 表单区域 -->
      <el-form :model="loginForm" :rules="loginRule" ref="loginFromRef" label-width="70px" class="login_form">
        <!-- 用户名 -->
        <el-form-item label="用户名:" prop="username">
          <el-input v-model="loginForm.username" prefix-icon="el-icon-user"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item label="密码:" prop="password">
          <el-input v-model="loginForm.password" prefix-icon="el-icon-lock" type="password" ></el-input>
        </el-form-item>
        <!-- 按钮 -->
        <el-form-item label-width="0px" class="buts">
          <el-button type="primary" plain @click="setFromlogin">登陆</el-button>
          <el-button type="info" plain @click="setFromRef">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
    // 登陆表单的数据绑定
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 表单的验证规则对象 注：规则名需和数据绑定名一致否则会没有效果
      loginRule: {
        // 验证用户名是否合法
        username: [
          { required: true, message: '登陆名不能为空', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        // 验证密码是否合法
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 0, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 重置按钮执行
    setFromRef () {
      // console.log(this)
      this.$refs.loginFromRef.resetFields()
    },
    setFromlogin () {
      this.$refs.loginFromRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('Login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登陆失败')
        this.$message.success('登陆成功')
        console.log(res)
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}

</script>

<style lang="less" scoped>
.login_container {
  background-color: aquamarine;
  height: 100%;
}

.login_box {
  width: 450px;
  height: 300px;
  background-color: azure;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);

  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eeeeee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #eeeeee;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: aliceblue;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: rgb(215, 226, 225);
    }
  }
}

.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 35px;
  padding-left: 10px;
  box-sizing: border-box;
}

.buts {
  display: flex;
  justify-content: flex-end;
}
</style>
