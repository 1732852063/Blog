<template>
  <div class="app-container">
    <!-- 数据表格 -->
    <el-table
      :data="data"
      style="width: 100%"
      border
    >
      <el-table-column
        prop="date"
        label="序号"
        width="60"
        align="center"
      > <template slot-scope="scope">{{ scope.$index + (currentPage - 1) * eachPage + 1 }}</template>
      </el-table-column>
      <el-table-column
        prop="title"
        label="文章名称"
        width="150"
        align="center"
      ><template slot-scope="scope">
        <el-popover
          placement="top-start"
          title="预览图博客"
          width="230"
          trigger="hover"
        >
          <el-image
            style="width: 200px;"
            :src="scope.row.thumb"
            fit="contain"
            :preview-src-list="srcList"
          />
          <a slot="reference" href="#" target="_blank" @click.prevent="goToTitleHandle(scope.row)">{{ scope.row.title }}</a>
        </el-popover>
      </template>
      </el-table-column>
      <el-table-column
        prop="description"
        label="文章描述"
        align="center"
      ><template slot-scope="scope">{{ scope.row.description }}</template>
      </el-table-column>
      <el-table-column
        prop="scanNumber"
        label="浏览数"
        width="80px"
        align="center"
      ><template slot-scope="scope">{{ scope.row.scanNumber }}</template>
      </el-table-column>
      <el-table-column
        prop="commentNumber"
        label="评论量"
        width="80px"
        align="center"
      ><template slot-scope="scope">{{ scope.row.commentNumber }}</template>
      </el-table-column>
      <el-table-column
        prop="category"
        label="所属分类"
        align="center"
      ><template slot-scope="scope">{{
        scope.row.category.name === null ? '未分类' : scope.row.category.name
      }}</template>
      </el-table-column>
      <el-table-column
        prop="createDate"
        label="创建日期"
        align="center"
      ><template slot-scope="scope">{{ scope.row.createDate }}</template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
      > <template slot-scope="scope">
        <el-tooltip class="item" effect="dark" content="编辑" placement="top" :hide-after="3000">
          <el-button
            type="primary"
            icon="el-icon-edit"
            circle
            size="mini"
            @click="editBlogHandle(scope.row)"
          />
        </el-tooltip>
        <el-tooltip class="item" effect="dark" content="删除" placement="top" :hide-after="3000">
          <el-button
            type="danger"
            icon="el-icon-delete"
            circle
            size="mini"
            @click="deleteBlog(scope.row)"
          />
        </el-tooltip>
      </template>
        </el-tooltip>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <el-pagination
      style="margin-top:30px"
      background
      :page-size="eachPage"
      :page-sizes="[5,10,20]"
      layout="prev, pager, next,total,->,sizes,jumper"
      :total="count"
      @size-change="sizeChangeHandle"
      @current-page="currentChangeHandle"
      @prev-click="prevChangeHandle"
      @next-click="nextChangeHandle"
    />
  </div>
</template>

<script>
import { findBlog, delOneBlog } from '@/api/blog.js'
import { formatData } from '@/utils/tool'
import { server_URL, frontEnd_URL } from '@/urlConfig.js'
export default {
  data() {
    return {
      data: [], // 存储数据详情
      srcList: [], // 预览图链接数组
      eachPage: 5, // 默认显示的条数
      currentPage: 1, // 当前默认页数
      totalPage: 0, // 默认总页数
      count: 0, // 数据总页数
      pagerCurrentPage: 1 // 分页栏当前页码

    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      findBlog(this.currentPage, this.eachPage).then(({ data }) => {
        // console.log(data)
        this.data = data.rows
        for (var i of this.data) {
          i.createDate = formatData(i.createDate)
          i.thumb = server_URL + i.thumb
          this.srcList.push(i.thumb)
        }
        this.count = data.total
        this.totalPage = Math.ceil(this.count / this.eachPage)
        // 下面的if在删除文章时，肯能触发
        if (this.currentPage > this.totalPage) {
          this.currentPage = this.totalPage
          this.fetchData()
        }
      })
    },
    // 跳转到具体文章
    goToTitleHandle(blogInfo) {
      console.log(blogInfo, 'blog')
      window.open(`${frontEnd_URL}/article/${blogInfo.id}`)
    },
    // 编辑文章
    editBlogHandle(blogInfo) {
      this.$router.push(`/editBlog/${blogInfo.id}`)
    },
    // 删除文章
    deleteBlog(blogInfo) {
      this.$confirm('此操作将永久删除该文章, 是否继续?', '是否删除该文章', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        delOneBlog(blogInfo.id).then(() => {
          this.fetchData()
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    // 分页相关
    sizeChangeHandle(pageNum) {
      this.eachPage = parseInt(pageNum)
      this.currentPage = 1
      this.pagerCurrentPage = 1
      this.fetchData()
    },
    currentChangeHandle(pageNum) {
      this.currentPage = parseInt(pageNum)
      this.fetchData()
    },
    prevChangeHandle() { this.currentPage -= 1 },
    nextChangeHandle() { this.currentPage += 1 }
  }
}

</script>

<style lang="scss" scoped>

</style>
