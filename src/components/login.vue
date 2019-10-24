<template>
  <div class="login">
    <el-form :rules="rules" ref="form" :model="form" status-icon label-width="80px">
      <img src="../assets/panda.png" alt="">
      <el-form-item label="用户名" prop="username">
        <el-input @keyup.enter.native="login" placeholder="请输入用户名" v-model="form.username"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input placeholder="请输入密码" type="password" v-model="form.password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" class="loginBtn" @click="login">登录</el-button>
        <el-button @click="reset">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
// import axios from 'axios'
export default {
  data () {
    return {
      form: {
        username: '',
        password: ''
      },
      rules: {
        // required: true 为必填项
        // message 提示消息
        // trigger: blur 失去焦点触发校验   change 修改时就触发
        // min: 最小长度,  max: 最大长度
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 12, message: '用户名长度必须是3-12位', trigger: ['change', 'blur'] }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, max: 12, message: '密码长度必须是3-12位', trigger: ['change', 'blur'] }
        ]
      }
    }
  },
  methods: {
    reset () {
      this.$refs.form.resetFields()
    },
    login () {
    // 获取到 el-form组件, 调用校验方法
    // 参数1: 是否校验成功  boolean
    // 参数2: 错误的字段信息
      this.$refs.form.validate(isValid => {
        if (!isValid) return
        // 校验成功
        console.log('发送登录ajax请求')
        console.log(this.form)
        this.$axios.post('login', this.form).then(
          res => {
            // console.log(res)
            // console.log(res.data)
            const { meta, data } = res
            if (meta.status === 200) {
              // 登录成功，跳转页面
              console.log(meta.msg)
              localStorage.setItem('token', data.token)
              this.$message({
                message: '登录成功',
                type: 'success',
                duration: 1000
              })
              this.$router.push({ name: 'index' })
            } else {
              // 给用户提示，登录失败
              console.log(meta.msg)
              this.$message.error('登录失败')
            }
          }
        )
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.login {
  width: 100%;
  height: 100%;
  background-color: #9d9d9d;
  overflow: hidden;
  .el-form {
    position: relative;
    width: 400px;
    background-color: #fff;
    padding: 20px;
    padding-top: 70px;
    margin: 0 auto;
    margin-top: 200px;
    border-radius: 20px;
    img {
      position: absolute;
      left: 50%;
      top: -55px;
      transform: translateX(-50%);
      width: 90px;
      height: 90px;
      border-radius: 50%;
      border: 1px solid #fff;
    }
  }
  .loginBtn {
    margin-right: 60px;
  }
}
</style>
