<template>
  <div>
    <el-button type="primary" size="small" @click="addEditor">新增栏目</el-button>
    <el-button type="danger" size="small" @click="batchDelete">批量删除</el-button>
    <el-table :data="tableData" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" />

      <el-table-column prop="id" label="编号" width="120" />
      <el-table-column prop="name" label="栏目名称" width="200" />
      <el-table-column prop="description" label="描述" />
      <el-table-column prop="parentId" label="父栏目" />
      <el-table-column fixed="right" label="操作" width="100" align="center">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="toEditor(scope.row)">编辑</el-button>
          <el-button type="text" size="small" style="color:#F56C6C" @click="toDelete(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 新增模态框 -->
    <el-dialog title="添加栏目" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="栏目名称" :label-width="formLabelWidth">
          <el-input v-model="form.name" autocomplete="off" />
        </el-form-item>

        <el-form-item label="描述" :label-width="formLabelWidth">
          <el-input v-model="form.description" type="textarea" autocomplete="off" />
        </el-form-item>

        <el-form-item label="父栏目" :label-width="formLabelWidth">
          <el-select v-model="form.parentId" clear="" placeholder="请选择父栏目">
            <el-option v-for="c in tableData" :key="c.id" :label="c.name" :value="c.id" />
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveOrUpdateCategory(form)">确 定</el-button>
      </div>
    </el-dialog>

    <!-- /新增模态框 -->

    <!-- 编辑模态框 -->
    <!-- <el-dialog title="编辑栏目" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="栏目名称" :label-width="formLabelWidth">
          <el-input v-model="form.name" autocomplete="off" />
        </el-form-item>

        <el-form-item label="描述" :label-width="formLabelWidth">
          <el-input v-model="form.description" type="textarea" autocomplete="off" />
        </el-form-item>

        <el-form-item label="父栏目" :label-width="formLabelWidth">
          <el-select v-model="form.parentId" clearable="" placeholder="请选择父栏目">
            <el-option v-for="c in tableData" :key="c.id" :label="c.name" :value="c.id" />
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateEditor(form)">确 定</el-button>
      </div>
    </el-dialog> -->

    <!-- /编辑模态框 -->
  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
export default {
  data() {
    return {
      ids: [],
      tableData: [],
      dialogFormVisible: false,
      formLabelWidth: '80px',
      form: {}
    }
  },

  created() {
    this.reloadData()
  },
  methods: {
    handleSelectionChange(val) {
      this.ids = val.map(temp => temp.id)
    },

    reloadData() {
      request.get('/category/findAll').then(response => {
        this.tableData = response.data
      })
    },
    batchDelete() {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        request
          .request({
            url: '/category/batchDeleteById',
            method: 'post',
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
            },
            data: qs.stringify({ ids: this.ids })
          })
          .then(response => {
            this.$message({
              message: response.message,
              type: 'success'
            })
            this.reloadData()
          })
      })
    },
    addEditor() {
      this.form = {}
      this.dialogFormVisible = true
    },
    /* addCategory(form) {
      const url = '/category/insertCategory'
      request
        .request({
          url: url,
          method: 'post',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          },
          data: qs.stringify(this.form)
        })
        .then(response => {
          this.$message({
            message: response.message,
            type: 'success'
          })
          this.reloadData()
          this.dialogFormVisible = false
        })
    }, */
    toEditor(row) {
      this.form = row
      this.dialogFormVisible = true
    },
    saveOrUpdateCategory(form) {
      request.request({
        url: '/category/saveOrUpdate',
        method: 'post',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        data: qs.stringify(form)
      }).then(response => {
        this.$message({
          message: response.message,
          type: 'success'
        })
        this.reloadData()
        this.dialogFormVisible = false
      })
    },
    /* updateEditor(form) {
      request.request({
        url: '/category/updateCategoryById',
        method: 'post',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        data: qs.stringify(form)
      }).then(response => {
        this.$message({
          message: response.message,
          type: 'success'
        })
        this.reloadData()
        this.dialogFormVisible = false
      })
    }, */
    toDelete(id) {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        request
          .get('/category/deleteCategoryById', {
            params: { id: id }
          })
          .then(response => {
            this.$message({
              message: response.message,
              type: 'success'
            })
            this.reloadData()
          })
      })
    }
  }
}
</script>

<style scoped>
</style>
