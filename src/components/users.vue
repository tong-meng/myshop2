<template>
  <div class="users">
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索框 -->
    <el-input v-model="query" class="input-with-select" placeholder="请输入搜索关键字">
      <el-button @click="searchUser" slot="append" icon="el-icon-search"></el-button>
    </el-input>
    <!-- 添加用户 -->
    <el-button @click="shwoAddUsers" type="success" plain>添加用户</el-button>
    <!-- 表格 -->
    <el-table :data="userList">
      <el-table-column prop="username" label="姓名"></el-table-column>
      <el-table-column prop="email" label="邮箱"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>
      <el-table-column label="用户状态">
        <template v-slot:default="obj">
          <el-switch @change="changeState(obj.row)" v-model="obj.row.mg_state" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
        </template>
      </el-table-column>
      <el-table-column prop label="操作">
        <template v-slot:default="{row}">
          <el-button @click="showEditDialog(row)" plain size="small" icon="el-icon-edit" type="primary"></el-button>
          <el-button @click="delUser(obj.row.id)" plain size="small" icon="el-icon-delete" type="danger"></el-button>
          <el-button plain size="small" icon="el-icon-check" type="success"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <div class="block">
      <span class="demonstration"></span>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="pagenum"
        :page-sizes="[2, 4, 6, 8]"
        :page-size="pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </div>
    <!-- 添加-给对话框提供表单 -->

    <el-dialog @close="closeDialog" title="添加用户" :visible.sync="dialogVisible" width="40%">
      <el-form ref="form" :rules="rules" :model="form" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="form.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="form.password" type="password" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="form.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="form.email" placeholder="请输入手机"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 显示一个对话框 -->
    <el-dialog title="修改用户" :visible.sync="editVisible" width="40%">

      <el-form ref="editForm" :rules="rules" :model="editForm" label-width="80px">
        <el-form-item label="用户名">
          <el-tag type="info">{{ editForm.username }}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile" placeholder="请输入手机"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button @click="editUser" type="primary">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// import axios from 'axios'
export default {
  created () {
    this.getUserList()
  },
  data () {
    return {
      query: '',
      pagenum: 1,
      pagesize: 4,
      userList: [],
      total: 0,
      dialogVisible: false,
      form: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editVisible: false,
      editForm: {
        id: '',
        username: '',
        email: '',
        mobile: ''
      },
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 12,
            message: '用户名长度在 3 到 12 个字符',
            trigger: ['blur', 'change']
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 3,
            max: 12,
            message: '密码长度在3-12个字符',
            trigger: ['blur', 'change']
          }
        ],
        email: [
          {
            type: 'email',
            message: '请输入正确的邮箱地址',
            trigger: ['blur', 'change']
          }
        ],
        mobile: [
          {
            pattern: /^1[3-9]\d{9}/,
            message: '请输入正确的手机号',
            trigger: ['blur', 'change']
          }
        ]
      }
    }
  },
  methods: {
    getUserList () {
      // axios.post(url, data, config)   config中一般可以配置一些额外的参数 (请求头)
      // axios.get(url, config)   可以配置params, 也可以配置请求头等额外参数
      // 发送 get 获取用户的 axios
      this.$axios
        .get('users', {
          // 配置 params
          params: {
            query: this.query,
            pagenum: this.pagenum,
            pagesize: this.pagesize
          }
          // 配置 请求头
          // headers: {
          //   Authorization: localStorage.getItem('token')
          // }
        })
        .then(res => {
          // console.log(res)
          const { data, meta } = res
          this.total = data.total
          if (meta.status === 200) {
            this.userList = data.users
          } else {
            this.$message.error(meta.msg)
          }
        })
    },
    // 修改每页的条数
    handleSizeChange (val) {
      // console.log(`每页 ${val} 条`)
      this.pagesize = val
      // 将当前页更新成第一页
      this.pagenum = 1
      // 重新渲染
      this.getUserList()
    },
    // 当前页变化时, 触发
    handleCurrentChange (val) {
      // val 用户选择的当前页
      this.pagenum = val
      // 重新渲染
      this.getUserList()
    },
    async delUser (id) {
      try {
        await this.$confirm('亲, 你确认要进行删除吗?', '温馨提示', {
          type: 'warning'
        })
        const { meta } = this.$axios.delete(`users/${id}`)
        if (meta.status === 200) {
          this.$message.success('删除成功')
          if (this.userList.length === 1 && this.pagenum > 1) {
            this.pagenum--
          }
          // 重新渲染当前页
          this.getUserList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    searchUser () {
      // 搜索后，需要将当前页改为第一页
      this.pagenum = 1
      this.getUserList()
    },
    async changeState (row) {
      const { meta } = await this.$axios.put(
        `users/${row.id}/state/${row.mg_state}`
      )
      if (meta.status === 200) {
        this.$message.success('修改成功')
      } else {
        this.$message.error(meta.msg)
      }
    },
    shwoAddUsers () {
      this.dialogVisible = true
    },
    async addUser () {
      try {
        await this.$refs.form.validate()
        // 说明校验成功了
        // 发送添加的 ajax
        const { meta } = await this.$axios.post('users', this.form)
        if (meta.status === 201) {
          // 成功的提示
          this.$message.success(meta.msg)
          // 关闭对话框
          this.dialogVisible = false
          // 将当前页, 更新成最后一页    总条数1/每页多少条5
          // 先让 total + 1
          this.total++
          this.pagenum = Math.ceil(this.total / this.pagesize)
          // 重新渲染
          this.getUserList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    },
    closeDialog () {
      this.$refs.form.resetFields()
    },
    showEditDialog (row) {
      console.log(row)
      this.editVisible = true
      this.editForm.username = row.username
      this.editForm.email = row.email
      this.editForm.mobile = row.mobile
      this.editForm.id = row.id
    },
    async editUser () {
      try {
        await this.$refs.editForm.validate() // 校验表格 成功之后往下执行
        const { id, email, mobile } = this.editForm
        const { meta } = await this.$axios.put(`users/${id}`, { email, mobile })
        if (meta.status === 200) {
          this.$message.success(meta.msg)
          // 关闭对话框
          this.editVisible = false
          // 重新渲染
          this.getUserList()
        } else {
          this.$message.error(meta.msg)
        }
      } catch (e) {
        console.log(e)
      }
    }
  }
}
</script>

<style lang='scss' scoped>
.users {
  .input-with-select {
    width: 300px;
    margin-bottom: 10px;
  }
  .addBtn {
    margin: 0 20px;
  }
}
</style>
