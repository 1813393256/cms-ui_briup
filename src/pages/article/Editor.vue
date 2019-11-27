<template>
  <div>
    <el-button type="text" @click="back">返回</el-button>
    <el-form ref="form" :model="form" label-width="80px">
      <el-form-item label="所属栏目">

        <el-select v-model="form.categoryId" placeholder="请选择活动区域">
          <el-option v-for="c in cateList" :key="c.id" :label="c.name" :value="c.id" />
        </el-select>

      </el-form-item>
      <el-form-item label="标题">
        <el-input v-model="form.title" />
      </el-form-item>

      <el-form-item label="正文">
        <el-input v-model="form.content" type="textarea" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">发布</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
import request from '@/utils/request'
import qs from 'querystring'
export default {
  data() {
    return {
      form: {},
      cateList: []
    }
  },
  created() {
    this.form = this.$route.query
    request.get('/category/findAll').then(response => {
      this.cateList = response.data
    })
  },
  methods: {
    back() {
      this.$router.go(-1)
    },
    onSubmit() {
      // 交互
      request.request({
        url: '/article/saveOrUpdate',
        method: 'post',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        data: qs.stringify(this.form)
      })
        .then(response => {
        // 提示成功
          this.$message({
            message: response.message,
            type: 'success'
          })
          // 返回列表页
          this.back()
        })
    }
  }
}
</script>
<style scoped>

</style>
