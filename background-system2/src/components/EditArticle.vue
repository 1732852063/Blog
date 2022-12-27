<template>
  <div>
    <!-- 文章标题 -->
    <div class="block">文章标题</div>
    <div style="margin-bottom">
      <el-input v-model="form.title" placeholder="请输入文章标题" />
    </div>
    <!-- 文章编辑 -->
    <div class="block">文章编辑</div>
    <editor
      ref="toastuiEditor"
      :initial-value="form.editorText"
      height="600px"
      :options="editorOptions"
    />
    <!-- 文章描述 -->
    <div class="block">文章描述</div>
    <el-input v-model="form.description" type="textarea" :rows="6" />
    <!-- 文章头图 -->
    <div class="block">文章头图</div>
    <Upload v-model="form.thumb" />
    <!-- 文章分类 -->
    <div class="block">文章分类</div>
    <el-select v-model="form.select" placeholder="请选择文章分类">
      <el-option v-for="item in blogType" :key="item.id" :label="item.name" :value="item.id" />
    </el-select>
    <!-- 发布按钮 -->
    <div class="block">
      <el-button type="primary" style="margin-top:15px" @click="addArticleHandle">{{ btnContent }}</el-button>
    </div>

  </div>
</template>
<script>
import '@toast-ui/editor/dist/toastui-editor.css'
import { Editor } from '@toast-ui/vue-editor'
import Upload from '@/components/Upload.vue'
import { getBlogType } from '@/api/blogType'
import { addBlog, editBlog, findOneBlog } from '@/api/blog.js'
import '@toast-ui/editor/dist/i18n/zh-cn'
export default {
  components: {
    editor: Editor,
    Upload
  },
  props: ['mode'],
  data() {
    return {
      form: {
        title: '', // 文章标题
        editorText: '', // 用户编辑内容
        description: '', // 文章描述
        thumb: '', // 文章图片
        select: '', // 文章分类
        imageUrl: '' // 图片地址
      },
      blogType: [],
      editorOptions: {
        language: 'zh-CN'
      },
      btnContent: '发布文章'
    }
  },
  created() {
    // 拿取分类
    getBlogType().then(({ data }) => {
      this.blogType = data
    })
    if (this.mode === 'edit') {
      this.id = this.$route.params.id
      findOneBlog(this.id).then(({ data }) => {
      // 回填值表单
        this.form = data
        this.form.select = data.category === null ? '' : data.category.id
        this.$refs.toastuiEditor.invoke('setHTML', data.htmlContent)
      })
      this.btnContent = '确认修改'
    }
  },
  methods: {
    addArticleHandle() {
      // 添加文章业务逻辑 获取请求 发送请求
      const html = this.$refs.toastuiEditor.invoke('getHTML')
      const markdown = this.$refs.toastuiEditor.invoke('getMarkdown')
      // 组装传给服务器的对象
      const obj = {
        'title': this.form.title, // 文章标题
        'description': this.form.description, // 文章简介
        'createDate': new Date().getTime(), // 发布日期 时间戳
        'categoryId': this.form.select, // 分类信息
        'toc': [], // 生成目录
        'htmlContent': html, // 正文
        'thumb': this.form.thumb, // 缩略图
        'markdownContent': markdown // 文章内容
      }
      // 提交给服务器
      if (obj.title && obj.description && obj.htmlContent && obj.categoryId) {
        if (this.mode === 'add') {
          addBlog(obj).then(() => {
            this.$router.push('/blogList')// 跳转文章列表
          })
        } else {
          editBlog({ id: this.form.id, data: this.obj }).then(() => {
            this.$router.push('/blogList')// 跳转文章列表
            this.$message.success('文章修改成功')
          })
        }
      } else {
        this.$message.error('请填写所有内容')
      }
    }

  }
}
</script>

  <style lang="scss" scoped>
  .block {
      margin: 15px 0;
      font-weight: 100;
  }
  </style>

