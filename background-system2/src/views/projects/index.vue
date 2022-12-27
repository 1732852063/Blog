<template>
  <div class="app-container">
    <!-- 数据表格 -->
    <el-table
      v-loading="listLoading"
      :data="data"
      element-loading-text="加载数据中"
      border
      fit
      highlight-current-row
    >
      <!-- 序号 -->
      <el-table-column
        prop="date"
        label="序号"
        width="60"
        align="center"
      > <template slot-scope="scope">{{ scope.$index + 1 }}</template>
      </el-table-column>
      <!-- 项目名称 -->
      <el-table-column
        prop="title"
        label="项目名称"
        width="150"
        align="center"
      ><template slot-scope="scope">
        <a :href="scope.row.url" target="_blank" class="proName">{{ scope.row.name }}</a>
      </template>
      </el-table-column>
      <!-- 项目描述 -->
      <el-table-column
        prop="title"
        label="项目描述"
        align="center"
      ><template slot-scope="scope">
        {{ scope.row.description.toString() }}
      </template>
      </el-table-column>
      </el-table-column>
      <!-- 预览图 -->
      <el-table-column
        prop="title"
        label="预览图"
        align="center"
      ><template slot-scope="scope">
        <el-image
          style="width:120px"
          :src="scope.row.thumb2"
          :preview-src-list="srcList"
        />
      </template>
      </el-table-column>
      <!-- 操作 -->
      <el-table-column
        label="操作"
        align="center"
        width="150px"
      > <template slot-scope="scope">
        <el-tooltip class="item" effect="dark" content="github" placement="top" :hide-after="3000">
          <el-button
            type="info"
            icon="iconfont icon-github"
            circle
            size="mini"
            @click="openGithubHandle(scope.row)"
          />
        </el-tooltip>
        <el-tooltip class="item" effect="dark" content="编辑" placement="top" :hide-after="3000">
          <el-button
            type="primary"
            icon="el-icon-edit"
            circle
            size="mini"
            @click="projectHandle(scope.row)"
          />
        </el-tooltip>
        <el-tooltip class="item" effect="dark" content="删除" placement="top" :hide-after="3000">
          <el-button
            type="danger"
            icon="el-icon-delete"
            circle
            size="mini"
            @click="deleteProject(scope.row)"
          />
        </el-tooltip>
      </template>
        </el-tooltip>
      </el-table-column>
    </el-table>
    <!-- 编辑对话框 -->
    <el-dialog
      title="请编辑该项目信息"
      :visible.sync="dialogFormVisible"
      fullscreen
    >
      <el-form :model="form">
        <el-form-item label="项目名称">
          <el-input v-model="form.name" />
        </el-form-item>
        <el-form-item label="项目描述(每一项描述以英文逗号分隔)">
          <el-input v-model="form.description" />
        </el-form-item>
        <el-form-item label="项目链接">
          <el-input v-model="form.url" />
        </el-form-item>
        <el-form-item label="github 地址">
          <el-input v-model="form.github" />
        </el-form-item>
        <el-form-item label="预览图地址">
          <Upload v-model="form.thumb" />
        </el-form-item>
        <el-form-item label="等级排序">
          <el-select v-model="form.order" placeholder="分类等级">
            <el-select label="1" value="1" />
            <el-select label="2" value="2" />
            <el-select label="3" value="3" />
            <el-select label="4" value="4" />
            <el-select label="5" value="5" />
          </el-select>
        </el-form-item>

      </el-form>
      <!-- 确认取消按钮 -->
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="confirmEditProjectHandle">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { getProject, setProject, deleteProject } from '@/api/project.js'
import { server_URL } from '@/urlConfig'
import Upload from '@/components/Upload.vue'
export default {
  components: { Upload },
  data() {
    return {
      data: [], // 存储数据
      listLoading: false, //
      srcList: [], // 对应数组
      dialogFormVisible: false, // 一开始编辑对话框不可见
      form: {
        name: '',
        description: '',
        url: '',
        github: '',
        thumb: '',
        order: 1

      }
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      this.listLoading = true
      getProject().then(({ data }) => {
        this.data = data
        this.listLoading = false
        for (const i of this.data) {
          i.thumb2 = server_URL + i.thumb
          this.srcList.push(i.thumb2)
        }
      })
    },
    openGithubHandle(projectInfo) {
      window.open(projectInfo.github)
    },
    projectHandle(projectInfo) {
      this.dialogFormVisible = true
      this.form = { ...projectInfo, description: projectInfo.description.toString() }
    },
    deleteProject(projectInfo) {
      this.$confirm('此操作将永久删除该项目, 是否继续?', '是否删除该项目', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteProject(projectInfo.id).then(() => {
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
    // 确认修改
    confirmEditProjectHandle() {
      const obj = { ...this.form }
      obj.description = this.form.description.split(',')
      obj.order = parseInt(this.form.order)
      setProject(obj.id, obj).then(() => {
        this.dialogFormVisible = false
        this.fetchData()
        this.$message.success('修改成功')
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.proName:hover{
    color: rgb(163, 163, 163);
}
</style>
