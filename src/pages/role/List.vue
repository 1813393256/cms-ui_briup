<template>
  <div>
    <!-- 主页面 -->
    <el-button type="primary" size="small" @click="toaddRole">添加</el-button>
    <el-table :data="roleData" style="width: 100%" size="small">
      <el-table-column prop="name" label="角色名称" />
      <el-table-column fixed="right" label="操作" align="center" width="120">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="removeRole(scope.row.id)">移除</el-button>
          <el-button type="text" size="small" @click="toAuthorization(scope.row)">授权</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- /主页面 -->

    <!-- 添加角色模态框 -->
    <el-dialog title="添加角色" :visible.sync="addRoleVisible">
      <el-form :model="role">
        <el-form-item label="角色名称" label-width="100px">
          <el-input v-model="role.name" autocomplete="off" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole(role)">确 定</el-button>
      </div>
    </el-dialog>

    <!-- /添加角色模态框 -->
    <!-- 角色授权 -->
    <el-dialog title="角色授权" :visible.sync="addAuthorizationVisible" :before-close="cancle">
      <el-form :model="authorization" size="small">
        <el-form-item label="角色名称" label-width="100px">{{ authorization.name }}</el-form-item>
        <el-form-item label="权限" label-width="100px">
          {{ authorization }}
          <el-cascader-panel
            v-model="authorization.privileges"
            :options="options"
            :props="props"
            clearable
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancle">取 消</el-button>
        <el-button type="primary" @click="addAuthorization(authorization)">确 定</el-button>
      </div>
    </el-dialog>
    <!-- /角色授权 -->
  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
export default {
  data() {
    return {
      roleData: [],
      addRoleVisible: false,
      addAuthorizationVisible: false,
      role: {},
      authorization: {},
      props: { multiple: true, value: 'id', label: 'name', emitPath: false },
      options: []
    }
  },
  created() {
    this.reloadData()
    this.reloadPrivilegeTree()
  },

  methods: {
    reloadData() {
      request.get('/role/selectAllRoleWithPrivilege').then(response => {
        response.data.forEach(item => {
          item.privileges = item.privileges.map(p => p.id)
        })
        this.roleData = response.data
      })
    },
    reloadPrivilegeTree() {
      request.get('/privilege/findPrivilegeTree').then(response => {
        this.options = response.data
      })
    },
    toaddRole() {
      this.role = {}
      this.addRoleVisible = true
    },
    addRole(role) {
      request
        .request({
          url: '/role/saveOrUpdate',
          method: 'post',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          },
          data: qs.stringify(role)
        })
        .then(response => {
          this.$message({
            message: response.message,
            type: 'success'
          })
          this.reloadData()
          this.addRoleVisible = false
        })
    },

    removeRole(rid) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        request
          .get('/role/deleteRoleByRoleId', { params: { id: rid }})
          .then(response => {
            this.$message({
              message: response.message,
              type: 'success'
            })
            this.reloadData()
          })
      })
    },
    toAuthorization(row) {
      this.authorization = row
      this.addAuthorizationVisible = true
    },
    addAuthorization(authorization) {
      request.request({
        url: '/role/authorization',
        method: 'post',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        data: qs.stringify(authorization)
      }).then(response => {
        this.$message({
          message: response.message,
          type: 'success'
        })
        this.addAuthorizationVisible = false
        this.reloadData()
      })
    },
    cancle() {
      this.addAuthorizationVisible = false
      this.reloadData()
    }
  }
}
</script>
