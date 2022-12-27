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
      > <template slot-scope="scope">{{ scope.$index + 1 }}</template>
      </el-table-column>
      <el-table-column
        prop="title"
        label="标题"
        width="150"
        align="center"
      ><template slot-scope="scope">{{ scope.row.title }}</template>
      </el-table-column>
      <el-table-column
        prop="description"
        label="描述"
      ><template slot-scope="scope">{{ scope.row.description }}</template>
      </el-table-column>
      <el-table-column
        label="中图预览"
        align="center"
      > <template slot-scope="scope">
        <el-image
          style="width: 100px;"
          :src="scope.row.midImg2"
          fit="fill"
        /></template>
      </el-table-column>
      <el-table-column
        label="大图预览"
        align="center"
      > <template slot-scope="scope">
        <el-image
          style="width: 100px;"
          :src="scope.row.bigImg2"
          fit="fill"
        />
      </template>
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
            @click="editBannerHandle(scope.row)"
          /></el-tooltip>
      </template>
        </el-tooltip>
      </el-table-column>
    </el-table>
    <!-- 弹出框 -->
    <el-dialog title="请编辑信息" :visible.sync="dialogFormVisible" top="5vh">
      <el-form :model="form">
        <el-form-item label="标题">
          <el-input v-model="form.title" />
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="form.description" />
        </el-form-item>
        <!-- 中图 -->
        <el-row>
          <el-col :span="12">
            <el-form-item label="首页中图">
              <Upload v-model="form.midImg" />
            </el-form-item>
          </el-col>

          <!-- 大图  -->
          <el-col :span="12">
            <el-form-item label="首页大图">
              <Upload v-model="form.bigImg" />
            </el-form-item>
          </el-col>

        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="editBannerConfirm">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { getBanner, setBanner } from '@/api/banner.js'
import { server_URL } from '@/urlConfig.js'
import Upload from '@/components/Upload.vue'

export default {
  components: {
    Upload
  },
  data() {
    return {
      data: [], // 存储数据
      dialogFormVisible: false, // 编辑对话框是否显示
      form: {
        id: '',
        midImg: '',
        bigImg: '',
        title: '',
        description: ''
      }
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      getBanner().then(res => {
        this.data = res.data
        for (var item of this.data) {
          item.midImg2 = server_URL + item.midImg
          item.bidImg2 = server_URL + item.bidImg
        }
      })
    },
    editBannerHandle(bannerInfo) {
    // 1.将数据给form
    // 2.打开
      this.form = { ...bannerInfo }
      this.dialogFormVisible = true
    },
    editBannerConfirm() {
    // 从表单里边拿到用户修改的数据，发送给服务器
      const arr = [...this.data]
      for (var i = 0; i < arr.length; i++) {
        if (arr[i].id === this.form.id) {
          arr[i] = this.form
        }
      }
      setBanner(arr).then(res => {
        this.dialogFormVisible = false
        this.$message({
          message: '修改成功',
          type: 'success'
        })
        this.fetchData()
      })
    }
  }
}
</script>

<style lang="scss" scoped>

</style>
