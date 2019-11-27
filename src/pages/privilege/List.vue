<template>
  <div>
    <!-- 主页面 -->
    <el-button type="primary" size="small" @click="toaddPrivilege">添加</el-button>
    <!-- {{privilegeData}}  -->
    <el-table
      :data="privilegeData"
      style="width: 100%;margin-bottom: 20px;"
      size="small"
      :lazy="true"
      row-key="id"
      :load="lazyLoadPrivilege"
      :tree-props="{children: 'children', hasChildren: 'hasChildren'}"
    >
      <el-table-column prop="name" label="名称" />
      <el-table-column prop="route" label="路径" />
      <el-table-column prop="type" label="类型" />
      <el-table-column fixed="right" label="操作" align="center" width="120">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="removePrivilege(scope.row.id)">移除</el-button>
          <el-button type="text" size="small" @click="toUpdatePrivilege(scope.row)">修改</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- /主页面 -->

    <!-- 修改或添加模态框 -->
    <el-dialog title="添加权限" :visible.sync="addPrivilegeVisible">
      <el-form :model="form">
        <!-- {{form}} -->
        <el-form-item label="名称" :label-width="formLabelWidth">
          <el-input v-model="form.name" autocomplete="off" />
        </el-form-item>
        <el-form-item label="路径" :label-width="formLabelWidth">
          <el-input v-model="form.route" autocomplete="off" />
        </el-form-item>
        <el-form-item label="类型" :label-width="formLabelWidth">
          <el-select v-model="form.type" placeholder="请选择">
            <el-option label="菜单" value="menu" />
            <el-option label="方法" value="method" />
          </el-select>
        </el-form-item>
        <el-form-item label="父权限" :label-width="formLabelWidth">
          <el-select v-model="form.parentId" placeholder="请选择">
            <el-option
              v-for="(item,index) in privilegeData"
              :key="index"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="描述" :label-width="formLabelWidth">
          <el-input
            v-model="form.description"
            type="textarea"
            :autosize="{ minRows: 2, maxRows: 4}"
            placeholder="请输入内容"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addPrivilegeVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveOrUpdatePrivilege(form)">确 定</el-button>
      </div>
    </el-dialog>
    <!-- /修改或添加模态框 -->
  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
export default {
  data() {
    return {
      addPrivilegeVisible: false,
      formLabelWidth: '80px',
      form: {},
      privilegeData: []
    }
  },
  created() {
    this.reloadPrivilege()
  },
  methods: {
    reloadPrivilege() {
      request.get('/privilege/findPrivilegeTree').then(response => {
        response.data.forEach(item => {
          item.hasChildren = !item.parentId
        })
        this.privilegeData = response.data
      })
    },
    toaddPrivilege() {
      this.form = {}
      this.addPrivilegeVisible = true
    },
    removePrivilege(rowId) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        request
          .get('/privilege/deletePrivilegeById', { params: { id: rowId }})
          .then(response => {
            this.$message({
              message: response.message,
              type: 'success'
            })
            this.reloadPrivilege()
          })
      })
    },
    toUpdatePrivilege(row) {
      this.form = row
      this.addPrivilegeVisible = true
    },
    saveOrUpdatePrivilege(form) {
      request
        .request({
          url: '/privilege/saveOrUpdate',
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
          this.addPrivilegeVisible = false
          this.reloadPrivilege()
        })
    },
    lazyLoadPrivilege(row, treeNode, resolve) {
      request.get('/privilege/findByParentId?id=' + row.id).then(response => {
        response.data.forEach(item => {
          item.hasChildren = !item.parentId
        })
        resolve(response.data)
      })
    }
  }
}
</script>
