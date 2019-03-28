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
        <div class="upload-wrap">
          <file-upload
            class="btn btn-primary"
            :post-action="postAction"
            :multiple="true"
            :thread="3"
            :uploadType="uploadType"
            v-model="files"
            @input-filter="inputFilter"
            @input-file="inputFile"
            ref="upload">
          </file-upload>
          <button type="button" class="btn btn-primary" @click="onAddFiles">添加文件</button>
          <button type="button" class="btn btn-primary" @click="onAddFolader">添加文件夹</button>
          <button type="button" class="btn btn-primary" @click="onAddTga">添加序列</button>
          <button type="button" class="btn btn-primary" @click="onAddP2Folader">添加P2文件</button>
        </div>
        <div class="upload-footer" style="margin-top: 20px;">
          <button type="button" class="btn btn-success" v-if="!$refs.upload || !$refs.upload.active" @click.prevent="$refs.upload.active = true">
            开始上传
          </button>
          <button type="button" class="btn btn-danger"  v-else @click.prevent="$refs.upload.active = false">
            停止上传
          </button>
        </div>
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

function getFileName(str) {
    var pos = str.lastIndexOf(".");
    if (pos == -1)
        return str;

    var name = str.substr(0, pos);
    return name;
}

function getFileExt(str) {
    var pos = str.lastIndexOf(".");
    if (pos == -1)
        return "";

    var ext = str.substr(pos + 1);
    return ext;
}

function getNameNum(str) {
    // var str = "中极少00001";
    var patt = new RegExp('[0-9]+$', 'g');
    var result = null;

    var num = '';
    while ((result = patt.exec(str)) != null) {
        num = result[0];
        // name = str.substr(0, result.index);
    }

    return num;
}

export default {
  components: {
    FileUpload,
  },

  data() {
    return {
      files: [],
      postAction: '/upload',
      uploadType: ''
    }
  },

  methods: {
    inputFilter(newFile, oldFile, prevent) {
      if (newFile && !oldFile) {
        // Before adding a file
        // 添加文件前

        // Filter system files or hide files
        // 过滤系统文件 和隐藏文件
        if (/(\/|^)(Thumbs\.db|desktop\.ini|\..+)$/.test(newFile.dataList[0].name)) {
          return prevent()
        }

        // Filter php html js file
        // 过滤 php html js 文件
        if (/\.(php5?|html?|jsx?)$/i.test(newFile.dataList[0].name)) {
          return prevent()
        }

        // 图片序列判断
        if (this.uploadType == 'tga') {
          // 格式判断
          let bError = false;
          for (let i = 0; i < newFile.dataList.length; i++) {
            let ext = getFileExt(newFile.dataList[i].name);
            ext = ext.toLowerCase();
            if (ext != 'tga') {
              alert(newFile.dataList[i].name + ' 不是tga文件')
              bError = true;
              break;
            }
          }
          if (bError) {
            return prevent()
          }

          // 连续判断
          let numArray = [];
          for (let i = 0; i < newFile.dataList.length; i++) {
            let shortname = getFileName(newFile.dataList[i].name);
            let num = getNameNum(shortname);
            numArray.push(num);
          }

          numArray.sort(function (a, b) {
            return a > b ? 1 : -1;
          });

          for (let i = parseInt(numArray[0]); i < numArray.length; i++) {
            if (i != parseInt(numArray[i])) {
              alert('序列文件不是连续的')
              return prevent();
            }
          }
        }

        // p2文件判断
        if (this.uploadType == 'p2') {
          let bFindVideo = false;
          let bFindAudio = false;
          for (let i = 0; i < newFile.dataList.length; i++) {
            let path = newFile.dataList[i].file.webkitRelativePath || '';
            let relativePath = path.substring(0, path.lastIndexOf('/'));
            relativePath = relativePath.substring(relativePath.lastIndexOf('/') + 1);
            relativePath = relativePath.toUpperCase();

            if (relativePath == 'VIDEO') {
              bFindVideo = true;
            } else if (relativePath == 'AUDIO') {
              bFindAudio = true;
            }
          }

          if (!bFindVideo || !bFindAudio) {
            alert('不是P2文件')
            return prevent()
          }
        }
      }
    },
    inputFile(newFile, oldFile) {
      if (this.$refs.upload.uploaded && this.files.length) {
        // updated
        let allSuccess = this.files.every(item => {
          return item.success ? true : false;
        })

        if (!allSuccess) { return; }

        console.log('updated')
      }
    },
    onAddFiles () {
      this.uploadType = ''
      let input = this.$refs.upload.$el.querySelector('input')
      this.directory = false
      input.directory = false
      input.webkitdirectory = false

      input.onclick = null
      input.click()
    },
    // add folader
    onAddFolader() {
      this.uploadType = ''
      if (!this.$refs.upload.features.directory) {
        alert('你的浏览器不支持选择文件夹')
        return
      }

      let input = this.$refs.upload.$el.querySelector('input')
      input.directory = true
      input.webkitdirectory = true
      this.directory = true

      input.onclick = null
      input.click()
      input.onclick = (e) => {
        this.directory = false
        input.directory = false
        input.webkitdirectory = false
      }
    },
    // add p2 folader
    onAddTga() {
      this.uploadType = 'tga'
      let input = this.$refs.upload.$el.querySelector('input')
      this.directory = false
      input.directory = false
      input.webkitdirectory = false

      input.onclick = null
      input.click()
    },
    // add p2 folader
    onAddP2Folader() {
      this.uploadType = 'p2'
      if (!this.$refs.upload.features.directory) {
        alert('你的浏览器不支持选择文件夹')
        return
      }
      let input = this.$refs.upload.$el.querySelector('input')
      input.directory = true
      input.webkitdirectory = true
      this.directory = true

      input.onclick = null
      input.click()
      input.onclick = (e) => {
        this.directory = false
        input.directory = false
        input.webkitdirectory = false
      }
    },
  }
}
</script>
