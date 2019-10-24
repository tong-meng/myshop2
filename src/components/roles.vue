<template>
  <div class="roles">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 添加按钮 -->
    <el-button @click="showAddDialog" plain type="success">添加角色</el-button>
    <!-- 表格 -->
    <el-table :data="roleList">
      <el-table-column type="expand">
        <template v-slot:default="{ row }">
          <el-row class="l1" v-for="l1 in row.children" :key="l1.id">
            <el-col :span="4">
              <el-tag closable>{{l1.authName}}</el-tag>
              <i class="el-icon-arrow-right"></i>
            </el-col>
            <el-col :span="20">
              <el-row class="l2" v-for="l2 in l1.children" :key="l2.id">
                <el-col :span="4">
                  <el-tag closable type="success">{{l2.authName}}</el-tag>
                  <i class="el-icon-arrow-right"></i>
                </el-col>
                <el-col :span="20">
                  <el-tag
                    class="l3"
                    closable
                    type="warning"
                    v-for="l3 in l2.children"
                    :key="l3.id"
                  >{{l3.authName }}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作">
        <template v-slot:default="{ row }">
          <el-button @click="showEditDialog(row)" type="primary" plain size="small" icon="el-icon-edit"></el-button>
          <el-button @click="delRole(row.id)" type="danger" plain size="small" icon="el-icon-delete"></el-button>
          <el-button @click="showAssignDialog(row)" type="success" plain size="small" icon="el-icon-check">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 对话框 1-->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="40%">
      <el-form ref="form" :rules="rules" :model="form" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="form.roleName" placeholder="请输入角色名称"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="form.roleDesc" placeholder="请输入角色描述"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button @click="addRole" type="primary">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 对话框2 -->
    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="40%">
      <el-form ref="form" :rules="rules" :model="form" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="form.roleName" placeholder="请输入角色名称"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="form.roleDesc" placeholder="请输入角色描述"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button @click="editRole" type="primary">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 角色分配对话框 -->
    <el-dialog title="提示" :visible.sync="AssignDialogVisible" width="40%">

      <el-tree
        :data="data"
        show-checkbox
        default-expand-all
        node-key="id"
        ref="tree"
        highlight-current
        :props="defaultProps"
      ></el-tree>

      <span slot="footer" class="dialog-footer">
        <el-button @click="AssignDialogVisible = false">取 消</el-button>
        <el-button @click="assignRights" type="primary">分 配</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  created () {
    this.getRoleList()
  },
  data () {
    return {
      roleList: [],
      addDialogVisible: false,
      editDialogVisible: false,
      AssignDialogVisible: false,
      form: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      rules: {
        roleName: [
          {
            required: true,
            message: '请输入角色名称',
            trigger: ['blur', 'change']
          }
        ],
        roleDesc: [
          {
            required: true,
            message: '请输入角色描述',
            trigger: ['blur', 'change']
          }
        ]
      },
      data: [],
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      roleId: ''
    }
  },
  methods: {
    async getRoleList () {
      const { data, meta } = await this.$axios.get('roles')
      if (meta.status === 200) {
        this.roleList = data
        // console.log(data)
      } else {
        this.$message.error(meta.msg)
      }
    },
    showAddDialog () {
      this.addDialogVisible = true
    },
    async addRole () {
      await this.$refs.form.validate()
      const { meta } = await this.$axios.post('roles', this.form)
      if (meta.status === 201) {
        this.$message.success(meta.msg)
        // 关闭对话框
        this.dialogVisible = false
        // 清除表格
        this.$refs.form.resetFields()
        // 重新渲染
        this.getRoleList()
      } else {
        this.$message.error(meta.msg)
      }
    },
    async showEditDialog (row) {
      this.editDialogVisible = true
      // console.log(row)
      this.$nextTick(() => {
        this.form.id = row.id
        this.form.roleName = row.roleName
        this.form.roleDesc = row.roleDesc
      })
    },
    async editRole () {
      const { id, roleName, roleDesc } = this.form
      const { meta } = await this.$axios.put(`roles/${id}`, {
        roleName,
        roleDesc
      })
      if (meta.status === 200) {
        this.editDialogVisible = false
      } else {
        this.$message.error(meta.msg)
      }
    },
    async delRole (id) {
      const { meta } = await this.$axios.delete(`roles/${id}`)
      if (meta.status === 200) {
        this.$message.success('删除成功')
        this.getRoleList()
      } else {
        this.$message.error(meta.msg)
      }
    },
    async showAssignDialog (row) {
      this.AssignDialogVisible = true
      this.roleId = row.id
      const { data, meta } = await this.$axios.get('rights/tree')
      if (meta.status === 200) {
        // 成功了, 将数据存起来
        this.data = data
        // console.log(data)
        const ids = []
        row.children.forEach(l1 =>
          l1.children.forEach(l2 =>
            l2.children.forEach(l3 =>
              ids.push(l3.id)
            )))
        // console.log(ids)
        this.$refs.tree.setCheckedKeys(ids)
      } else {
        this.$message.error(meta.msg)
      }
    },
    async assignRights () {
      const ids = this.$refs.tree.getCheckedKeys()
      // console.log(ids)
      const halfs = this.$refs.tree.getHalfCheckedKeys()
      // console.log(halfs)
      const rids = [...ids, ...halfs].join(',')
      // console.log(rids)
      const { meta } = await this.$axios.post(`roles/${this.roleId}/rights`, { rids })
      if (meta.status === 200) {
        this.$message.success(meta.msg)
        this.AssignDialogVisible = false
        this.getRoleList()
      } else {
        this.$message.error(meta.msg)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.roles {
  .addBtn {
    margin-bottom: 30px;
  }
  .l1 {
    margin-bottom: 5px;
    border-bottom: 1px dotted #ccc;
  }
  .l2 {
    margin-bottom: 5px;
  }
  .l3 {
    margin-right: 5px;
    margin-bottom: 5px;
  }
}
</style>
