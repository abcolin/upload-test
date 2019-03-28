<template>
  <span :class="className">
    <slot></slot>
    <label :for="inputId || name"></label>
    <input-file></input-file>
  </span>
</template>
<style>
.file-uploads {
  overflow: hidden;
  position: relative;
  text-align: center;
  display: inline-block;
  vertical-align: middle;
}
.file-uploads.file-uploads-html4 input, .file-uploads.file-uploads-html5 label {
  /* background fix ie  click */
  background: #fff;
  opacity: 0;
  font-size: 20em;
  z-index: 1;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  position: absolute;
  width: 100%;
  height: 100%;
}
.file-uploads.file-uploads-html5 input, .file-uploads.file-uploads-html4 label {
  /* background fix ie  click */
  background: rgba(255, 255, 255, 0);
  overflow: hidden;
  position: fixed;
  width: 1px;
  height: 1px;
  z-index: -1;
  opacity: 0;
}
</style>
<script>
import ChunkUploadDefaultHandler from './chunk/ChunkUploadHandler'
import InputFile from './InputFile.vue'

//生成Guid
function NewGuid() {
    var guid = "";
    for (var i = 1; i <= 32; i++) {
        var n = Math.floor(Math.random() * 16.0).toString(16);
        guid += n;
    }
    return guid;
}

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

function getNamePre(str) {
    var patt = new RegExp('[0-9]+$', 'g');
    var result = null;

    var name = '';
    while ((result = patt.exec(str)) != null) {
        name = str.substr(0, result.index);
        if (name.length > 0) {
            var temp = name.substr(name.length - 1);
            if (temp == '_' || temp == '-') {
                name = name.substr(0, name.length - 1);
            }
        }
    }

    return name;
}

function findShortname(shortname, allFileList) {
    var bfind = false;

    for (var n = 0; n < allFileList.length; n++) {
        var shortname2 = allFileList[n].name;
        if (shortname2.toLowerCase() == shortname.toLowerCase()) {
            bfind = true;
            break;
        }
    }

    return bfind;
}

function parseFileList(files, uploadType = '') {
    if (uploadType == 'tga') {
        var filelist = [];

        var dataList = [];
        for (var i = 0; i < files.length; i++) {
            dataList.push(files[i]);
        }

        var guid = NewGuid();

        filelist.push({
            guid: guid,
            uploadType: uploadType,
            dataList: dataList
        });

        return filelist;
    }

    if (uploadType == 'p2') {
        var filelist = [];

        var dataList = [];
        for (var i = 0; i < files.length; i++) {
            dataList.push(files[i]);
        }

        var guid = NewGuid();

        filelist.push({
            guid: guid,
            uploadType: uploadType,
            dataList: dataList
        });

        return filelist;
    }

    var wavNum = [];
    var filelist = [];
    for (var i = 0; i < files.length; i++) {
        var ext = getFileExt(files[i].name);
        ext = ext.toLowerCase();
        // 判断后缀名
        if (ext == 'avi'
            || ext == 'm2v'
            || ext == 'mp2'
            || ext == 'wav'
            || ext == 'mpg'
            || ext == 'mxf'
            || ext == 'm2p'
            || ext == 'vob'
            || ext == 'wmv'
            || ext == 'rmvb'
            || ext == 'rm'
            || ext == 'mpeg4'
            || ext == 'mov'
            || ext == 'ts'
            || ext == 'mp3'
            || ext == 'mp4'
            || ext == 'wma') {
            if (ext != 'wav') {
                // 找对应音频 wav
                var dataList = [];
                dataList.push(files[i]);
                if (ext == 'm2v'
                    || ext == 'mp2'
                    || ext == 'avi'
                    || ext == 'm2p') {
                    for (var j = 0; j < files.length; j++) {
                        if ((getFileName(files[j].name).toLowerCase() == getFileName(files[i].name).toLowerCase()
                                || getFileName(files[j].name).toLowerCase() == getFileName(files[i].name).toLowerCase() + '.a3')
                            && i != j && getFileExt(files[j].name).toLowerCase() == 'wav') {
                            dataList.push(files[j]);
                            wavNum.push(j);
                        }
                    }
                }

                var guid = NewGuid();
                filelist.push({
                    guid: guid,
                    uploadType: uploadType,
                    dataList: dataList
                });
            }
        } else {
            var guid = NewGuid();
            var dataList = [];
            dataList.push(files[i]);

            filelist.push({
                guid: guid,
                uploadType: uploadType,
                dataList: dataList
            });
        }
    }

    for (var i = 0; i < files.length; i++) {
        var ext = getFileExt(files[i].name);
        ext = ext.toLowerCase();
        if (ext == 'wav') {
            var flag = 0;
            for (var j = 0; j < wavNum.length; j++) {
                if (wavNum[j] == i) {
                    flag = 1;
                }
            }
            if (flag == 0) {
                var a = [];
                a.push(files[i]);
                // filelist.push(a);
                var guid = NewGuid();
                filelist.push({
                    guid: guid,
                    uploadType: uploadType,
                    dataList: a
                });
            }
        }
    }
    return filelist;
}

const CHUNK_DEFAULT_OPTIONS = {
  headers: {},
  action: '',
  minSize: 1048576,
  maxActive: 3,
  maxRetries: 5,

  handler: ChunkUploadDefaultHandler
}

export default {
  components: {
    InputFile,
  },
  props: {
    inputId: {
      type: String,
    },

    name: {
      type: String,
      default: 'file',
    },

    accept: {
      type: String,
    },

    capture: {
    },

    disabled: {
    },

    multiple: {
      type: Boolean,
    },

    maximum: {
      type: Number,
      default() {
        return this.multiple ? 0 : 1
      }
    },

    addIndex: {
      type: [Boolean, Number],
    },

    directory: {
      type: Boolean,
    },

    postAction: {
      type: String,
    },

    putAction: {
      type: String,
    },

    customAction: {
      type: Function,
    },

    headers: {
      type: Object,
      default: Object,
    },

    data: {
      type: Object,
      default: Object,
    },

    timeout: {
      type: Number,
      default: 0,
    },


    drop: {
      default: false,
    },

    dropDirectory: {
      type: Boolean,
      default: true,
    },

    size: {
      type: Number,
      default: 0,
    },

    extensions: {
      default: Array,
    },


    value: {
      type: Array,
      default: Array,
    },

    thread: {
      type: Number,
      default: 1,
    },

    // Chunk upload enabled
    chunkEnabled: {
      type: Boolean,
      default: false
    },

    // Chunk upload properties
    chunk: {
      type: Object,
      default: () => {
        return CHUNK_DEFAULT_OPTIONS
      }
    },

    uploadType: {
      type: String,
      default: ''
    },
  },

  data() {
    return {
      files: this.value,
      features: {
        html5: true,
        directory: false,
        drag: false,
      },

      active: false,
      dropActive: false,

      uploading: 0,

      destroy: false,
    }
  },


  /**
   * mounted
   * @return {[type]} [description]
   */
  mounted() {
    let input = document.createElement('input')
    input.type = 'file'
    input.multiple = true

    // html5 特征
    if (window.FormData && input.files) {
      // 上传目录特征
      if (typeof input.webkitdirectory === 'boolean' || typeof input.directory === 'boolean') {
        this.features.directory = true
      }

      // 拖拽特征
      if (this.features.html5 && typeof input.ondrop !== 'undefined') {
        this.features.drop = true
      }
    } else {
      this.features.html5 = false
    }

    // files 定位缓存
    this.maps = {}
    if (this.files) {
      for (let i = 0; i < this.files.length; i++) {
        let file = this.files[i]
        this.maps[file.id] = file
      }
    }

    this.$nextTick(function () {

      // 更新下父级
      if (this.$parent) {
        this.$parent.$forceUpdate()
      }

      // 拖拽渲染
      this.watchDrop(this.drop)
    })
  },

  /**
   * beforeDestroy
   * @return {[type]} [description]
   */
  beforeDestroy() {
    // 已销毁
    this.destroy = true

    // 设置成不激活
    this.active = false
  },

  computed: {
    /**
     * uploading 正在上传的线程
     * @return {[type]} [description]
     */

    /**
     * uploaded 文件列表是否全部已上传
     * @return {[type]} [description]
     */
    uploaded() {
      let file
      for (let i = 0; i < this.files.length; i++) {
        file = this.files[i]
        if (file.fileObject && !file.error && !file.success) {
          return false
        }
      }
      return true
    },

    chunkOptions () {
      return Object.assign(CHUNK_DEFAULT_OPTIONS, this.chunk)
    },

    className() {
      return [
        'file-uploads',
        this.features.html5 ? 'file-uploads-html5' : 'file-uploads-html4',
        this.features.directory && this.directory ? 'file-uploads-directory' : undefined,
        this.features.drop && this.drop ? 'file-uploads-drop' : undefined,
        this.disabled ? 'file-uploads-disabled' : undefined,
      ]
    }
  },


  watch: {
    active(active) {
      this.watchActive(active)
    },

    dropActive() {
      if (this.$parent) {
        this.$parent.$forceUpdate()
      }
    },

    drop(value) {
      this.watchDrop(value)
    },

    value(files) {
      if (this.files === files) {
        return
      }
      this.files = files

      let oldMaps = this.maps

      // 重写 maps 缓存
      this.maps = {}
      for (let i = 0; i < this.files.length; i++) {
        let file = this.files[i]
        this.maps[file.id] = file
      }

      // add, update
      for (let key in this.maps) {
        let newFile = this.maps[key]
        let oldFile = oldMaps[key]
        if (newFile !== oldFile) {
          this.emitFile(newFile, oldFile)
        }
      }

      // delete
      for (let key in oldMaps) {
        if (!this.maps[key]) {
          this.emitFile(undefined, oldMaps[key])
        }
      }
    },
  },

  methods: {

    // 清空
    clear() {
      if (this.files.length) {
        let files = this.files
        this.files = []

        // 定位
        this.maps = {}

        // 事件
        this.emitInput()
        for (let i = 0; i < files.length; i++) {
          this.emitFile(undefined, files[i])
        }
      }
      return true
    },

    // 选择
    get(id) {
      if (!id) {
        return false
      }

      if (typeof id === 'object') {
        return this.maps[id.id] || false
      }

      return this.maps[id] || false
    },

    // 添加
    add(_files, index = this.addIndex) {
      let files = _files
      let isArray = files instanceof Array

      // 不是数组整理成数组
      if (!isArray) {
        files = [files]
      }
      
      files = parseFileList(files, this.uploadType)

      // 遍历规范对象
      let addFiles = []
      for (let i = 0; i < files.length; i++) {
        let file = files[i]
        let fileObject = false
        if (file.fileObject === false) {
          // false
        } else if (file.fileObject) {
          fileObject = true
        } else if (typeof Element !== 'undefined' && file.dataList[0].el instanceof Element) {
          fileObject = true
        } else if (typeof Blob !== 'undefined' && file.dataList[0].file instanceof Blob) {
          fileObject = true
        }
        let size = 0;
        for (let j = 0; j < file.dataList.length; j++) {
          size += file.dataList[j].size
        }
        let shortname = getFileName(file.dataList[0].name)
        if (this.uploadType == 'tga') {
            shortname = getNamePre(shortname)
        } else if (this.uploadType == 'p2') {
            let path = file.dataList[0].file.webkitRelativePath || ''
            shortname = path.substring(0, path.indexOf('/'));
            if (shortname == 'CONTENTS') {
              shortname = 'P2盘'
            }
        }
        let n = 1;
        let oldshortname = shortname;
        while (findShortname(shortname, this.files)) {
          shortname = oldshortname + '(' + n + ')';
          n++;
        }

        if (fileObject) {
          file = {
            fileObject: true,
            size: size,
            name: shortname,
            type: file.dataList[0].type || '',
            active: false,
            error: '',
            success: false,
            putAction: this.putAction,
            postAction: this.postAction,
            timeout: this.timeout,
            // ...file,
            dataList: file.dataList,
            uploadType: file.uploadType,
            privacy: 1,
            response: {},

            progress: 0,          // 只读
            speed: 0,                  // 只读
            // xhr: false,                // 只读
            // iframe: false,             // 只读
          }

          file.data = {
            ...this.data,
            ...file.data ? file.data : {},
          }

          file.headers = {
            ...this.headers,
            ...file.headers ? file.headers : {},
          }
        }

        // 必须包含 id
        if (!file.id) {
          file.id = Math.random().toString(36).substr(2)
        }

        if (this.emitFilter(file, undefined)) {
          continue
        }

        // 最大数量限制
        if (this.maximum > 1 && (addFiles.length + this.files.length) >= this.maximum) {
          break
        }

        addFiles.push(file)

        // 最大数量限制
        if (this.maximum === 1) {
          break
        }
      }

      // 没有文件
      if (!addFiles.length) {
        return false
      }

      // 如果是 1 清空
      if (this.maximum === 1) {
        this.clear()
      }


      // 添加进去 files
      let newFiles
      if (index === true || index === 0) {
        newFiles = addFiles.concat(this.files)
      } else if (index) {
        newFiles = this.files.concat([])
        newFiles.splice(index, 0, ...addFiles)
      } else {
        newFiles = this.files.concat(addFiles)
      }

      this.files = newFiles

      // 定位
      for (let i = 0; i < addFiles.length; i++) {
        let file = addFiles[i]
        this.maps[file.id] = file
      }

      // 事件
      this.emitInput()
      for (let i = 0; i < addFiles.length; i++) {
        this.emitFile(addFiles[i], undefined)
      }

      return isArray ? addFiles : addFiles[0]
    },



    // 添加表单文件
    addInputFile(el) {
      let files = []
      if (el.files) {
        for (let i = 0; i < el.files.length; i++) {
          let file = el.files[i]
          let path = file.webkitRelativePath || file.relativePath || file.name
          let name = path.substring(path.indexOf('/') + 1)
          files.push({
            size: file.size,
            name: name,
            type: file.type,
            file,
          })
        }
      } else {
        var names = el.value.replace(/\\/g, '/').split('/')
        delete el.__vuex__
        files.push({
          name: names[names.length - 1],
          el,
        })
      }
      return this.add(files)
    },


    // 添加 DataTransfer
    addDataTransfer(dataTransfer) {
      let files = []
      if (dataTransfer.items && dataTransfer.items.length) {
        let items = []
        for (let i = 0; i < dataTransfer.items.length; i++) {
          let item = dataTransfer.items[i]
          if (item.getAsEntry) {
            item = item.getAsEntry() || item.getAsFile()
          } else if (item.webkitGetAsEntry) {
            item = item.webkitGetAsEntry() || item.getAsFile()
          } else {
            item = item.getAsFile()
          }
          if (item) {
            items.push(item)
          }
        }

        return new Promise((resolve, reject) => {
          let forEach = (i) => {
            let item = items[i]
            // 结束 文件数量大于 最大数量
            if (!item || (this.maximum > 0 && files.length >= this.maximum)) {
              return resolve(this.add(files))
            }
            this.getEntry(item).then(function (results) {
              files.push(...results)
              forEach(i + 1)
            })
          }
          forEach(0)
        })
      }

      if (dataTransfer.files.length) {
        for (let i = 0; i < dataTransfer.files.length; i++) {
          files.push(dataTransfer.files[i])
          if (this.maximum > 0 && files.length >= this.maximum) {
            break
          }
        }
        return Promise.resolve(this.add(files))
      }

      return Promise.resolve([])
    },


    // 获得 entry
    getEntry(entry, path = '') {
      return new Promise((resolve, reject) => {
        if (entry.isFile) {
          entry.file(function (file) {
            resolve([
              {
                size: file.size,
                name: path + file.name,
                type: file.type,
                file,
              }
            ])
          })
        } else if (entry.isDirectory && this.dropDirectory) {
          let files = []
          let dirReader = entry.createReader()
          let readEntries = () => {
            dirReader.readEntries((entries) => {
              let forEach = (i) => {
                if ((!entries[i] && i === 0) || (this.maximum > 0 && files.length >= this.maximum)) {
                  return resolve(files)
                }
                if (!entries[i]) {
                  return readEntries()
                }
                this.getEntry(entries[i], path + entry.name + '/').then((results) => {
                  files.push(...results)
                  forEach(i + 1)
                })
              }
              forEach(0)
            })
          }
          readEntries()
        } else {
          resolve([])
        }
      })
    },


    replace(id1, id2) {
      let file1 = this.get(id1)
      let file2 = this.get(id2)
      if (!file1 || !file2 || file1 === file2) {
        return false
      }
      let files = this.files.concat([])
      let index1 = files.indexOf(file1)
      let index2 = files.indexOf(file2)
      if (index1 === -1 || index2 === -1) {
        return false
      }
      files[index1] = file2
      files[index2] = file1
      this.files = files
      this.emitInput()
      return true
    },

    // 移除
    remove(id) {
      let file = this.get(id)
      if (file) {
        if (this.emitFilter(undefined, file)) {
          return false
        }
        let files = this.files.concat([])
        let index = files.indexOf(file)
        if (index === -1) {
          console.error('remove', file)
          return false
        }
        files.splice(index, 1)
        this.files = files

        // 定位
        delete this.maps[file.id]

        // 事件
        this.emitInput()
        this.emitFile(undefined, file)
      }
      return file
    },

    // 更新
    update(id, data) {
      let file = this.get(id)
      if (file) {
        let newFile = {
          ...file,
          ...data
        }
        // 停用必须加上错误
        if (file.fileObject && file.active && !newFile.active && !newFile.error && !newFile.success) {
          newFile.error = 'abort'
        }

        if (this.emitFilter(newFile, file)) {
          return false
        }

        let files = this.files.concat([])
        let index = files.indexOf(file)
        if (index === -1) {
          console.error('update', file)
          return false
        }
        files.splice(index, 1, newFile)
        this.files = files

        // 删除  旧定位 写入 新定位 （已便支持修改id)
        delete this.maps[file.id]
        this.maps[newFile.id] = newFile

        // 事件
        this.emitInput()
        this.emitFile(newFile, file)
        return newFile
      }
      return false
    },



    // 预处理 事件 过滤器
    emitFilter(newFile, oldFile) {
      let isPrevent = false
      this.$emit('input-filter', newFile, oldFile, function () {
        isPrevent = true
        return isPrevent
      })
      return isPrevent
    },

    // 处理后 事件 分发
    emitFile(newFile, oldFile) {
      this.$emit('input-file', newFile, oldFile)
      if (newFile && newFile.fileObject && newFile.active && (!oldFile || !oldFile.active)) {
        this.uploading++
        // 激活
        this.$nextTick(function () {
          setTimeout(() => {
            this.upload(newFile).then(() => {
              // eslint-disable-next-line
              newFile = this.get(newFile)
              if (newFile && newFile.fileObject) {
                this.update(newFile, {
                  active: false,
                  success: !newFile.error
                })
              }
            }).catch((e) => {
              this.update(newFile, {
                active: false,
                success: false,
                error: e.code || e.error || e.message || e
              })
            })
          }, parseInt(Math.random() * 50 + 50, 10))
        })
      } else if ((!newFile || !newFile.fileObject || !newFile.active) && oldFile && oldFile.fileObject && oldFile.active) {
        // 停止
        this.uploading--
      }

      // 自动延续激活
      if (this.active && (Boolean(newFile) !== Boolean(oldFile) || newFile.active !== oldFile.active)) {
        this.watchActive(true)
      }
    },

    emitInput() {
      this.$emit('input', this.files)
    },


    // 上传
    upload(id) {
      let file = this.get(id)

      // 被删除
      if (!file) {
        return Promise.reject('not_exists')
      }

      // 不是文件对象
      if (!file.fileObject) {
        return Promise.reject('file_object')
      }

      // 有错误直接响应
      if (file.error) {
        return Promise.reject(file.error)
      }

      // 已完成直接响应
      if (file.success) {
        return Promise.resolve(file)
      }

      // 后缀
      let extensions = this.extensions
      if (extensions && (extensions.length || typeof extensions.length === 'undefined')) {
        if (typeof extensions !== 'object' || !(extensions instanceof RegExp)) {
          if (typeof extensions === 'string') {
            extensions = extensions.split(',').map(value => value.trim()).filter(value => value)
          }
          extensions = new RegExp('\\.(' + extensions.join('|').replace(/\./g, '\\.') + ')$', 'i')
        }
        if (file.name.search(extensions) === -1) {
          return Promise.reject('extension')
        }
      }

      // 大小
      if (this.size > 0 && file.size >= 0 && file.size > this.size) {
        return Promise.reject('size')
      }

      if (this.customAction) {
        return this.customAction(file, this)
      }

      if (this.features.html5) {
        // if (this.shouldUseChunkUpload(file)) {
        //   return this.uploadChunk(file)
        // }
        // if (file.putAction) {
        //   return this.uploadPut(file)
        // }
        if (file.postAction) {
          return this.uploadHtml5(file)
        }
      }
      if (file.postAction) {
        // return this.uploadHtml4(file)
      }
      return Promise.reject('No action configured')
    },

    /**
     * Whether this file should be uploaded using chunk upload or not
     *
     * @param Object file
     */
    shouldUseChunkUpload (file) {
      return this.chunkEnabled &&
        !!this.chunkOptions.handler &&
        file.size > this.chunkOptions.minSize
    },

    /**
     * Upload a file using Chunk method
     *
     * @param File file
     */
    uploadChunk (file) {
      const HandlerClass = this.chunkOptions.handler
      file.chunk = new HandlerClass(file, this.chunkOptions)

      return file.chunk.upload()
    },

    uploadPut(file) {
      let querys = []
      let value
      for (let key in file.data) {
        value = file.data[key]
        if (value !== null && value !== undefined) {
          querys.push(encodeURIComponent(key) + '=' + encodeURIComponent(value))
        }
      }
      let queryString = querys.length ? (file.putAction.indexOf('?') === -1 ? '?' : '&') + querys.join('&') : ''
      let xhr = new XMLHttpRequest()
      xhr.open('PUT', file.putAction + queryString)
      return this.uploadXhr(xhr, file, file.file)
    },

    uploadHtml5(file) {
      let fd = new window.FormData()
      let postAction = file.postAction

      if (file.uploadType == 'tga') {
          let start = getNameNum(getFileName(file.dataList[0].file.name));
          let stop = getNameNum(getFileName(file.dataList[file.dataList.length - 1].file.name));
          fd.append('uri_name', file.name + '.tga?_fcc=imsq&start='+ start +'&stop=' + stop);
          for (let j = 0; j < file.dataList.length; j++) {
            fd.append('file_' + j, file.dataList[j].file);
            let title = getNameNum(getFileName(file.dataList[j].file.name));
            fd.append('file_' + j + '_rename', file.name + '_' + title + '.tga');
          }
      } else if (file.uploadType == 'p2') {
          fd.append('uri_name', file.name + '?_fcc=p2dk');
          let name = '';
          for (let j = 0; j < file.dataList.length; j++) {
            fd.append('file_' + j, file.dataList[j].file);
            let path = file.dataList[j].file.webkitRelativePath;

            if (name.length == 0) {
              name = path.substring(0, path.indexOf('/'));
            }

            let relativePath = '';
            let shortname = path.substring(0, path.indexOf('/'));
            if (shortname == 'CONTENTS') {
              relativePath = path;
            } else {
              relativePath = path.substring(path.indexOf('/') + 1);
            }

            let rename = relativePath;
            fd.append('file_' + j + '_rename', rename);
          }

          fd.append('empty_folders', 'CONTENTS/AVCLIP;CONTENTS/PROXY;CONTENTS/VOICE');
          fd.append('empty_files', 'LASTCLIP.TXT');

          postAction = file.postAction + '&type=multi';
      } else {
        if (file.dataList.length > 1) {
          let ext = getFileExt(file.dataList[1].name);

          fd.append('uri_name', file.dataList[0].name + '?_fcc=saud&audio=' + ext);
          for (var j = 0; j < file.dataList.length; j++) {
            fd.append('file_' + j, file.dataList[j].file);
            fd.append('file_' + j + '_rename', file.dataList[j].name);
          }
        } else {
          fd.append('uri_name', file.dataList[0].name);
          fd.append('file_0', file.dataList[0].file);
        }
      }

      let xhr = new XMLHttpRequest()
      xhr.open('POST', postAction)
      return this.uploadXhr(xhr, file, fd)
    },

    uploadXhr(xhr, _file, body) {
      let file = _file
      let speedTime = 0
      let speedLoaded = 0

      // 进度条
      xhr.upload.onprogress = (e) => {
        // 还未开始上传 已删除 未激活
        file = this.get(file)
        if (!e.lengthComputable || !file || !file.fileObject || !file.active) {
          return
        }

        // 进度 速度 每秒更新一次
        let speedTime2 = Math.round(Date.now() / 1000)
        if (speedTime2 === speedTime) {
          return
        }
        speedTime = speedTime2

        file = this.update(file, {
          progress: parseInt(e.loaded / e.total * 100),
          speed: e.loaded - speedLoaded,
        })
        speedLoaded = e.loaded
      }

      // 检查激活状态
      let interval = setInterval(() => {
        file = this.get(file)
        if (file && file.fileObject && !file.success && !file.error && file.active) {
          return
        }

        if (interval) {
          clearInterval(interval)
          interval = false
        }

        try {
          xhr.abort()
          xhr.timeout = 1
        } catch (e) {
        }
      }, 100)

      return new Promise((resolve, reject) => {
        let complete
        let fn = (e) => {
          // 已经处理过了
          if (complete) {
            return
          }
          complete = true
          if (interval) {
            clearInterval(interval)
            interval = false
          }

          file = this.get(file)

          // 不存在直接响应
          if (!file) {
            return reject('not_exists')
          }

          // 不是文件对象
          if (!file.fileObject) {
            return reject('file_object')
          }

          // 有错误自动响应
          if (file.error) {
            return reject(file.error)
          }

          // 未激活
          if (!file.active) {
            return reject('abort')
          }


          // 已完成 直接相应
          if (file.success) {
            return resolve(file)
          }

          let data = {}

          switch (e.type) {
            case 'timeout':
            case 'abort':
              data.error = e.type
              break
            case 'error':
              if (!xhr.status) {
                data.error = 'network'
              } else if (xhr.status >= 500) {
                data.error = 'server'
              } else if (xhr.status >= 400) {
                data.error = 'denied'
              }
              break
            default:
              if (xhr.status >= 500) {
                data.error = 'server'
              } else if (xhr.status >= 400) {
                data.error = 'denied'
              } else {
                data.progress = 100
              }
          }

          if (xhr.responseText) {
            let contentType = xhr.getResponseHeader('Content-Type')
            if (contentType && contentType.indexOf('/json') !== -1) {
              data.response = JSON.parse(xhr.responseText)
            } else {
              data.response = xhr.responseText
            }
          }

          // 更新
          file = this.update(file, data)

          // 相应错误
          if (file.error) {
            return reject(file.error)
          }

          // 响应
          return resolve(file)
        }

        // 事件
        xhr.onload = fn
        xhr.onerror = fn
        xhr.onabort = fn
        xhr.ontimeout = fn

        // 超时
        if (file.timeout) {
          xhr.timeout = file.timeout
        }

        // headers
        for (let key in file.headers) {
          xhr.setRequestHeader(key, file.headers[key])
        }

        // 更新 xhr
        file = this.update(file, { xhr })

        // 开始上传
        xhr.send(body)
      })
    },




    uploadHtml4(_file) {
      let file = _file
      let onKeydown = function (e) {
        if (e.keyCode === 27) {
          e.preventDefault()
        }
      }

      let iframe = document.createElement('iframe')
      iframe.id = 'upload-iframe-' + file.id
      iframe.name = 'upload-iframe-' + file.id
      iframe.src = 'about:blank'
      iframe.setAttribute('style', 'width:1px;height:1px;top:-999em;position:absolute; margin-top:-999em;')


      let form = document.createElement('form')

      form.action = file.postAction

      form.name = 'upload-form-' + file.id


      form.setAttribute('method', 'POST')
      form.setAttribute('target', 'upload-iframe-' + file.id)
      form.setAttribute('enctype', 'multipart/form-data')

      let value
      let input
      for (let key in file.data) {
        value = file.data[key]
        if (value && typeof value === 'object' && typeof value.toString !== 'function') {
          value = JSON.stringify(value)
        }
        if (value !== null && value !== undefined) {
          input = document.createElement('input')
          input.type = 'hidden'
          input.name = key
          input.value = value
          form.appendChild(input)
        }
      }
      form.appendChild(file.el)

      document.body.appendChild(iframe).appendChild(form)




      let getResponseData = function () {
        let doc
        try {
          if (iframe.contentWindow) {
            doc = iframe.contentWindow.document
          }
        } catch (err) {
        }
        if (!doc) {
          try {
            doc = iframe.contentDocument ? iframe.contentDocument : iframe.document
          } catch (err) {
            doc = iframe.document
          }
        }
        if (doc && doc.body) {
          return doc.body.innerHTML
        }
        return null
      }


      return new Promise((resolve, reject) => {
        setTimeout(() => {
          file = this.update(file, { iframe })

          // 不存在
          if (!file) {
            return reject('not_exists')
          }

          // 定时检查
          let interval = setInterval(() => {
            file = this.get(file)
            if (file && file.fileObject && !file.success && !file.error && file.active) {
              return
            }

            if (interval) {
              clearInterval(interval)
              interval = false
            }

            iframe.onabort({ type: file ? 'abort' : 'not_exists' })
          }, 100)


          let complete
          let fn = (e) => {
            // 已经处理过了
            if (complete) {
              return
            }
            complete = true


            if (interval) {
              clearInterval(interval)
              interval = false
            }

            // 关闭 esc 事件
            document.body.removeEventListener('keydown', onKeydown)

            file = this.get(file)

            // 不存在直接响应
            if (!file) {
              return reject('not_exists')
            }

            // 不是文件对象
            if (!file.fileObject) {
              return reject('file_object')
            }

            // 有错误自动响应
            if (file.error) {
              return reject(file.error)
            }

            // 未激活
            if (!file.active) {
              return reject('abort')
            }

            // 已完成 直接相应
            if (file.success) {
              return resolve(file)
            }

            let response = getResponseData()
            let data = {}
            switch (e.type) {
              case 'abort':
                data.error = 'abort'
                break
              case 'error':
                if (file.error) {
                  data.error = file.error
                } else if (response === null) {
                  data.error = 'network'
                } else {
                  data.error = 'denied'
                }
                break
              default:
                if (file.error) {
                  data.error = file.error
                } else if (data === null) {
                  data.error = 'network'
                } else {
                  data.progress = 100
                }
            }

            if (response !== null) {
              if (response && response.substr(0, 1) === '{' && response.substr(response.length - 1, 1) === '}') {
                try {
                  response = JSON.parse(response)
                } catch (err) {
                }
              }
              data.response = response
            }

            // 更新
            file = this.update(file, data)

            if (file.error) {
              return reject(file.error)
            }

            // 响应
            return resolve(file)
          }


          // 添加事件
          iframe.onload = fn
          iframe.onerror = fn
          iframe.onabort = fn


          // 禁止 esc 键
          document.body.addEventListener('keydown', onKeydown)

          // 提交
          form.submit()
        }, 50)
      }).then(function (res) {
        iframe.parentNode && iframe.parentNode.removeChild(iframe)
        return res
      }).catch(function (res) {
        iframe.parentNode && iframe.parentNode.removeChild(iframe)
        return res
      })
    },



    watchActive(active) {
      let file
      let index = 0
      while ((file = this.files[index])) {
        index++
        if (!file.fileObject) {
          // 不是文件对象
        } else if (active && !this.destroy) {
          if (this.uploading >= this.thread || (this.uploading && !this.features.html5)) {
            break
          }
          if (!file.active && !file.error && !file.success) {
            this.update(file, { active: true })
          }
        } else {
          if (file.active) {
            this.update(file, { active: false })
          }
        }
      }
      if (this.uploading === 0) {
        this.active = false
      }
    },


    watchDrop(_el) {
      let el = _el
      if (!this.features.drop) {
        return
      }

      // 移除挂载
      if (this.dropElement) {
        try {
          document.removeEventListener('dragenter', this.onDragenter, false)
          document.removeEventListener('dragleave', this.onDragleave, false)
          document.removeEventListener('drop', this.onDocumentDrop, false)
          this.dropElement.removeEventListener('dragover', this.onDragover, false)
          this.dropElement.removeEventListener('drop', this.onDrop, false)
        } catch (e) {
        }
      }

      if (!el) {
        el = false
      } else if (typeof el === 'string') {
        el = document.querySelector(el) || this.$root.$el.querySelector(el)
      } else if (el === true) {
        el = this.$parent.$el
      }

      this.dropElement = el

      if (this.dropElement) {
        document.addEventListener('dragenter', this.onDragenter, false)
        document.addEventListener('dragleave', this.onDragleave, false)
        document.addEventListener('drop', this.onDocumentDrop, false)
        this.dropElement.addEventListener('dragover', this.onDragover, false)
        this.dropElement.addEventListener('drop', this.onDrop, false)
      }
    },


    onDragenter(e) {
      e.preventDefault()
      if (this.dropActive) {
        return
      }
      if (!e.dataTransfer) {
        return
      }
      let dt = e.dataTransfer
      if (dt.files && dt.files.length) {
        this.dropActive = true
      } else if (!dt.types) {
        this.dropActive = true
      } else if (dt.types.indexOf && dt.types.indexOf('Files') !== -1) {
        this.dropActive = true
      } else if (dt.types.contains && dt.types.contains('Files')) {
        this.dropActive = true
      }
    },

    onDragleave(e) {
      e.preventDefault()
      if (!this.dropActive) {
        return
      }
      if (e.target.nodeName === 'HTML' || e.target === e.explicitOriginalTarget || (!e.fromElement && (e.clientX <= 0 || e.clientY <= 0 || e.clientX >= window.innerWidth || e.clientY >= window.innerHeight))) {
        this.dropActive = false
      }
    },

    onDragover(e) {
      e.preventDefault()
    },

    onDocumentDrop() {
      this.dropActive = false
    },

    onDrop(e) {
      e.preventDefault()
      this.addDataTransfer(e.dataTransfer)
    },
  }
}
</script>
