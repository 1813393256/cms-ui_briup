<template>
  <div>
    <el-button type="primary" size="small" @click="toPublishArticle">发布资讯</el-button>
    <el-table
      :data="articleForm"
      style="width:100%"
    >
      <el-table-column prop="id" label="编号" width="120" />
      <el-table-column prop="title" label="标题" width="120" />
      <el-table-column prop="authorId" label="作者" />
      <el-table-column prop="category.name" label="所属栏目" />
      <el-table-column
        fixed="right"
        label="操作"
        align="center"
        width="100"
      >
        <template slot-scope="scope"> <!-- scope -->
          <el-button type="text" size="small">查看</el-button>
          <el-button type="text" size="small" @click="toEditArticle(scope.row)">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import request from '@/utils/request'
export default {
  data() {
    return {
      articleForm: [],
      row: {}
    }
  },
  created() {
    request.get('/article/findAll')
      .then(response => {
        this.articleForm = response.data
      })
  },
  methods: {
    toPublishArticle() {
      // 跳转页面
      this.$router.push({ path: '/article/Editor'
      })
    },
    toEditArticle(row) {
      this.$router.push({
        path: '/article/Editor',
        query: row
      })
    }
  }
}
</script>

<style scoped>

</style>
