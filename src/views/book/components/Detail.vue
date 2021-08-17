<template>
  <el-form ref="postForm" :model="postForm" :rules="rules">
    <!--sticky :class-name="'sub-navbar'+ postForm.status" :class-name是sticky组件里-->
    <sticky :class-name="'sub-navbar'">
      <el-button v-if="!isEdit" @click="showGuide">显示帮助</el-button>
      <el-button
        :v-loading="loading"
        type="success"
        @click="submitForm"
      >
        {{ isEdit ? '编辑电子书' : '新增电子书' }}
      </el-button>
    </sticky>
    <div class="detail-container">
      <el-row>
        <warning />
        <el-col :span="24">
          <!--上传表单具体样式-->
          <ebook-upload
            :file-list="fileList"
            :disable="isEdit"
            @onSuccess="onuploadSuccess"
            @onRemove="onuploadRemove"
          />
        </el-col>
        <el-col :span="24">
          <el-form-item prop="title"><!--prop作用 表单规则校验-->
            <MdInput
              v-model="postForm.title"
              :maxlength="100"
              name="name"
              required
            >
              书名
            </MdInput><!--框架项目自带的-->
          </el-form-item>

          <el-row>
            <el-col :span="12">
              <el-form-item prop="author" label="作者:">
                <el-input v-model="postForm.author" placeholder="作者" />
              </el-form-item>
            </el-col>

            <el-col :span="12">
              <el-form-item prop="publisher" label="出版社:">
                <el-input v-model="postForm.publisher" placeholder="出版社" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="language" label="语言:">
                <el-input v-model="postForm.language" placeholder="语言" />
              </el-form-item>
            </el-col>

            <el-col :span="12">
              <el-form-item prop="rootFile" label="根文件:">
                <el-input v-model="postForm.rootFile" placeholder="根文件" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="filePath" label="文件路径:">
                <el-input v-model="postForm.filePath" placeholder="文件路径" disabled />
              </el-form-item>
            </el-col>

            <el-col :span="12">
              <el-form-item prop="unzipPath" label="解压路径:">
                <el-input v-model="postForm.unzipPath" placeholder="解压路径" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="coverPath" label="封面路径:">
                <el-input v-model="postForm.coverPath" placeholder="封面路径" disabled />
              </el-form-item>
            </el-col>

            <el-col :span="12">
              <el-form-item prop="originalName" label="文件名称:">
                <el-input v-model="postForm.originalName" placeholder="文件名称" disabled />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item prop="cover" label="封面:" label-width="120px">
                <a v-if="postForm.cover" :href="postForm.cover" target="_blank">
                  <img :src="postForm.cover" class="preview-img" alt="">
                </a>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item label="目录:" label-width="120px">
                <div v-if="contentsTree && contentsTree.length > 0" class="contents-wrapper">
                  <el-tree :data="contentsTree" @node-click="onContentClick" />
                </div>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
    </div>
  </el-form>
</template>

<script>
import Sticky from '../../../components/Sticky/index'
import Warning from './Warning'
import EbookUpload from '../../../components/EbookUpload'
import MdInput from '../../../components/MDinput'
import { createBook, getBook, updateBook } from '../../../api/book'

// 表单验证时把title等英文转换成中文
const fields = {
  title: '书名',
  author: '作者',
  publisher: '出版社',
  language: '语言'
}
export default {
  name: 'Detail',
  components: {
    Sticky,
    Warning,
    EbookUpload,
    MdInput
  },
  props: {
    isEdit: Boolean
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      if (value.length === 0) {
        callback(new Error(fields[rule.field] + '必须填写'))
      } else {
        callback()
      }
    }
    return {
      loading: false,
      postForm: {
        // status: 'draft'// 在styles/index.scss
      },
      fileList: [],
      contentsTree: [],
      // 校验规则，当标签:rules发现rules中title有校验规则则调用validateRequire，，
      // 必须有prop才能进行验证
      rules: {
        title: [{ validator: validateRequire }],
        author: [{ validator: validateRequire }],
        publisher: [{ validator: validateRequire }],
        language: [{ validator: validateRequire }]
      }
    }
  },
  created() {
    if (this.isEdit) {
      const fileName = this.$route.params.fileName
      this.getBookData(fileName)
    }
  },
  methods: {
    setData(data) {
      const {
        title,
        author,
        publisher,
        language,
        rootFile,
        cover,
        url,
        originalName,
        contents,
        contentsTree,
        fileName,
        coverPath,
        filePath,
        unzipPath
      } = data
      this.postForm = {
        ...this.postForm,
        title,
        author,
        publisher,
        language,
        rootFile,
        cover,
        url,
        originalName,
        contents,
        contentsTree,
        fileName,
        coverPath,
        filePath,
        unzipPath
      }
      this.contentsTree = contentsTree
      this.fileList = [{ name: originalName, url }]
    },
    getBookData(fileName) {
      // 向后端请求图书信息
      getBook(fileName).then(response => {
        this.setData(response.data)
      })
    },
    onContentClick(data) {
      if (data.text) {
        window.open(data.text)
      }
    },
    showGuide() {
      console.log('showGuide')
    },
    // 提交表单
    submitForm() {
      if (!this.loading) {
        this.loading = true
        this.$refs.postForm.validate((valid, fields) => {
          if (valid) { // 校验通过时
          // 浅拷贝
          // const book = { ...this.postForm } or
            const book = Object.assign({}, this.postForm)
            // delete book.contents
            delete book.contentsTree
            console.log('book', book)
            if (!this.isEdit) { // 服务端接口src/api/book.js
              createBook(book).then(response => {
                const { msg } = response
                this.$notify({
                  title: '操作成功',
                  message: msg,
                  type: 'success',
                  duration: 2000
                })
                this.loading = false
                this.setDefault()
              }).catch(() => { // 防止连点
                this.loading = false
              })
            } else {
              updateBook(book).then(response => {
                const { msg } = response
                this.$notify({
                  title: '操作成功',
                  message: msg,
                  type: 'success',
                  duration: 2000
                })
                this.loading = false
              }).catch(() => { // 防止连点
                this.loading = false
              })
            }
          } else {
            // 只拿到错误提示中的message即可
            const message = fields[Object.keys(fields)[0]][0].message
            this.$message({
              message: message,
              type: 'error'
            })
            this.loading = false
          }
        })
      }
    },
    onuploadSuccess(data) {
      console.log('onuploadSuccess', data)
      this.setData(data)
    },
    setDefault() {
      // this.postForm = Object.assign({}, defaultForm)
      this.contentsTree = []
      this.fileList = []
      // 对整个表重置，并移除校验结果
      this.$refs.postForm.resetFields()
    },
    onuploadRemove() {
      this.setDefault()
    }
  }
}
</script>

<style scoped>

</style>
