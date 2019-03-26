<template>
  <div class="example-simple">
    <h1 id="example-title" class="example-title">Simple Example</h1>
    <div class="upload">
      <ul>
        <li v-for="file in files" :key="file.id">
          <span>{{file.name}}</span> -
          <span>{{file.size | formatSize}}</span> -
          <span v-if="file.error">{{file.error}}</span>
          <span v-else-if="file.success">success</span>
          <span v-else-if="file.active">active</span>
          <span v-else-if="file.active">active</span>
          <span v-else></span>
        </li>
      </ul>
      <div class="example-btn">
        <file-upload
          class="btn btn-primary"
          :post-action="postUrl"
          :multiple="true"
          :thread="3"
          v-model="files"
          @input-filter="inputFilter"
          @input-file="inputFile"
          ref="upload">
          选择文件
        </file-upload>
        <button type="button" class="btn btn-success" v-if="!$refs.upload || !$refs.upload.active" @click.prevent="$refs.upload.active = true">
          开始上传
        </button>
        <button type="button" class="btn btn-danger"  v-else @click.prevent="$refs.upload.active = false">
          停止上传
        </button>
      </div>
    </div>
  </div>
</template>
<style>
.example-simple label.btn {
  margin-bottom: 0;
  margin-right: 1rem;
}
</style>

<script>
import FileUpload from '../components/upload/FileUpload.vue'

export default {
  components: {
    FileUpload,
  },

  data() {
    return {
      files: [],
      postUrl: 'http://192.168.6.32:8102/upload?p=BF9B458D2BA60889B5BC5DA383C18DEA'
    }
  },

  methods: {
    inputFilter(newFile, oldFile, prevent) {
      if (newFile && !oldFile) {
        // Before adding a file
        // 添加文件前

        // Filter system files or hide files
        // 过滤系统文件 和隐藏文件
        if (/(\/|^)(Thumbs\.db|desktop\.ini|\..+)$/.test(newFile.name)) {
          return prevent()
        }

        // Filter php html js file
        // 过滤 php html js 文件
        if (/\.(php5?|html?|jsx?)$/i.test(newFile.name)) {
          return prevent()
        }
      }
    },

    inputFile(newFile, oldFile) {
      if (newFile && !oldFile) {
        // add
        console.log('add', newFile)
      }
      if (newFile && oldFile) {
        // update
        console.log('update', newFile)
      }
      if (this.$refs.upload.uploaded) {
        // updated
        console.log('updated!!!')
      }

      if (!newFile && oldFile) {
        // remove
        console.log('remove', oldFile)
      }
    }
  }
}
</script>
