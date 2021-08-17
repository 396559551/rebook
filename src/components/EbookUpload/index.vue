<template>
  <div class="upload-container">
    <el-upload
      :action="action"
      :headers="headers"
      :multiple="false"
      :limit="1"
      :before-upload="beforeUpload"
      :on-success="onSuccess"
      :on-error="onError"
      :on-remove="onRemove"
      :file-list="fileList"
      :on-exceed="onExceed"
      :disabled="disable"
      drag
      show-file-list
      accept="application/epub+zip"
      class="image-upload"
    >
      <i class="el-icon-upload"></i>
      <div v-if="fileList.length === 0" class="el-upload__text">请将电子书拖入或 <em>点击上传</em></div>
      <div v-else class="el-upload__text">图书已上传</div>
    </el-upload>
  </div>
</template>

<script>
import { getToken } from '../../utils/auth';

export default {
  name: "index",
  props: {
    fileList: {
      type: Array,
      default() {
        return []
      }
    },
    disable: {
      type: Boolean,
      default() {
        return false;
      }
    }
  },
  data() {
    return {
      action: `${process.env.VUE_APP_BASE_API}/book/upload`
    }
  },
  computed: {
    headers() {
      return {
        Authorization: `Bearer ${getToken()}`
      }
    }
  },
  methods: {
    beforeUpload(file) {
      this.$emit('beforeUpload', file)
    },
    onSuccess(response, file) {
      const { code, msg, data } = response
      if (code === 0) {
        this.$message({
          message: msg,
          type: 'success'
        })
        this.$emit('onSuccess', data)// 传给父组件，进行处理
      } else {
        this.$message({
          message: (msg && `上传失败，失败原因: ${msg}`) || '上传失败',
          type: 'error'
        })
        this.$emit('onError', file)// 传给父组件，进行处理
      }
    },
    onError(err) {
      const errMsg = err.message && JSON.parse(err.message)
      this.$message({
        message: (errMsg && errMsg.msg && `上传失败，失败原因: ${errMsg.msg}` || '上传失败'),
        type: 'error'
      })
      this.$emit('onError', err)// 传给父组件，进行处理
    },
    onRemove() {
      this.$message({
        message: '删除成功',
        type: 'success'
      })
      this.$emit('onRemove')// 传给父组件，进行处理
    },
    onExceed() {
      this.$message({
        message: '每次只能上传一本电子书',
        type: 'warning'
      })
    },

  }
}
</script>

<style scoped>

</style>
