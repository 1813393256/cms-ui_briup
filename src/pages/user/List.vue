<template>
  <div>
    <el-button type="primary" size="small" @click="addUser">添加</el-button>
    <el-table :data="userData" style="width: 100%">
      <el-table-column prop="username" size="small" label="用户名" />
      <el-table-column prop="realname" size="small" label="姓名" />
      <el-table-column prop="gender" size="small" label="性别" />
      <el-table-column prop="telephone" size="small" label="手机号" />
      <el-table-column prop="status" size="small" label="状态" />
      <el-table-column fixed="right" size="small" align="center" label="操作" width="170">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="removeUser(scope.row.id)">移除</el-button>
          <el-button type="text" size="small">详情</el-button>
          <el-button type="text" size="small" @click="updateUser(scope.row)">修改</el-button>
          <el-button type="text" size="small" @click="setUserRole(scope.row)">设置</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 添加或更新用户模态框 -->
    <el-dialog title="添加用户" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input v-model="form.username" />
        </el-form-item>
        <el-form-item label="姓名" :label-width="formLabelWidth">
          <el-input v-model="form.realname" />
        </el-form-item>
        <el-form-item label="密码" :label-width="formLabelWidth">
          <el-input v-model="form.password" />
        </el-form-item>
        <el-form-item label="性别" :label-width="formLabelWidth">
          <el-radio v-model="form.gender" label="男">男</el-radio>
          <el-radio v-model="form.gender" label="女">女</el-radio>
        </el-form-item>
        <el-form-item label="手机号" :label-width="formLabelWidth">
          <el-input v-model="form.telephone" />
        </el-form-item>
        <el-form-item label="出生日期" :label-width="formLabelWidth">
          <el-date-picker
            v-model="form.birth"
            value-format="timestamp"
            type="date"
            placeholder="选择日期"
            style="width: 150px"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="toUpdateOrSave(form)">确 定</el-button>
      </div>
    </el-dialog>
    <!-- /添加或更新用户模态框 -->

    <!-- 设置用户角色模态框 -->
    <el-dialog title="设置角色" :visible.sync="setRoleVisible">
      <el-form ref="userRole" :model="userRole">
        <el-form-item label="用户名" :label-width="formLabelWidth">{{ userRole.realname }}</el-form-item>
        <el-form-item label="角色" :label-width="formLabelWidth">
          <el-cascader v-model="userRole.roles" :options="roles" :props="props" clearable />
        </el-form-item>
        {{ userRole.roles }}
        {{ roles }}
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="toCancle">取 消</el-button>
        <el-button type="primary" @click="toSetUserRole">确 定</el-button>
        <!-- toSetUserRole -->
      </div>
    </el-dialog>
    <!--  /设置用户角色模态框 -->
  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
export default {
  data() {
    return {
      userData: [],
      dialogFormVisible: false,
      setRoleVisible: false,
      form: {},
      formLabelWidth: '80px',
      userRole: {},
      // userRole2: {},
      roles: [],
      props: {
        multiple: true,
        value: 'id',
        label: 'name',
        emitPath: false
      }
    }
  },
  created() {
    /* request.get('/user/findUser').then(response => {
      this.userData = response.data
    }) */
    this.reloadData()
    this.reloadRoles()
  },

  methods: {
    toCancle() {
      this.setRoleVisible = false
      // this.userRole.roles = [];
      this.reloadData()
    },
    reloadRoles() {
      request.get('/role/findAll').then(response => {
        this.roles = response.data
      })
    },
    reloadData() {
      request.get('/user/cascadeRoleFindAll').then(response => {
        this.userData = response.data
        response.data.forEach(item => {
          item.roles = item.roles.map(r => r.id)
        })
      })
    },

    removeUser(id) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        request
          .get('/user/deleteUserById', { params: { id }})
          .then(response => {
            this.$message({
              message: response.message,
              type: 'success'
            })
            this.reloadData()
          })
      })
    },

    addUser() {
      this.form = { gender: '男' }
      this.dialogFormVisible = true
    },
    updateUser(row) {
      this.dialogFormVisible = true
      this.form = row
    },
    toUpdateOrSave(form) {
      request
        .request({
          url: '/user/updateOrSaveUser',
          method: 'post',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          },
          data: qs.stringify(form)
        })
        .then(response => {
          this.$message({
            message: response.message,
            type: 'success'
          })
          this.dialogFormVisible = false
          this.reloadData()
        })
    },
    setUserRole(row) {
      this.setRoleVisible = true
      this.userRole = row
      console.log(row)
    },
    toSetUserRole() {
      request
        .request({
          url: '/user/setRoles',
          method: 'post',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          },
          data: qs.stringify({
            id: this.userRole.id,
            roles: this.userRole.roles
          })
        })
        .then(response => {
          this.$message({
            message: response.message,
            type: 'success'
          })
          this.setRoleVisible = false
          this.reloadData()
        })
    }
  }
}
</script>
