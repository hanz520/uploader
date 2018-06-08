<template>
  <div id="uploader" class="webuploader"> 
    <div v-if="this.drag" ref="myDrager" class="webuploader-drag">
      <div class="webuploader-drag-icon">
        <i class="uploader uploader-cloud-upload"></i>
      </div>
      <div class="webuploader-drag-btn" ref="myPicker">{{button}}</div>
      <div class="webuploader-drag-tip">{{dragTip}}</div>
    </div>
    <div v-else ref="myPicker" class="webuploader-btn">
      {{button}}
    </div>
    <div v-if="$slots.tip" class="webuploader-tip">
      <slot name="tip" />
    </div>
    <div class="webuploader-list">
      
      <ul>
        <li v-for="item in fileList" :key="item.id">
          <span class="title"><i class="uploader uploader-file"></i>{{item.name}}</span>
          <span v-if="item.progress" class="loading"><i :style="`width: ${getProgress(item)};`"></i></span>
          <span class="operate operate-1">
            <i v-if="item.upStatus == '1'">{{getProgress(item)}}</i>
            <i v-if="item.upStatus == '2'" class="success uploader uploader-success"></i>
            <i v-if="item.upStatus == '3'" class="error uploader uploader-warn"></i>
          </span>
          <span class="operate operate-2">
            <i class="delete uploader uploader-error" @click="deleteFile(item)"></i>
          </span>
        </li>
      </ul>
    </div>
</div>
</template>

<script>
/* eslint-disable no-undef  */
import './webuploader.js'
import UploaderSwf from './Uploader.swf'
export default {
  props: {
    // 上传按钮文字
    button: {
      type: String,
      default: '点击选择'
    },
    // 上传文件列表
    fileList: {
      type: Array,
      default: function() {
        return []
      }
    },
    // 是否支持拖拽上传
    drag: {
      type: Boolean,
      default: false
    },
    dragTip: {
      type: String,
      default: '或者把文件拖到这里，单次不要超过300个'
    },
    // 服务器地址
    action: {
      type: String,
      default: ''
    },
    // 是否自动上传
    autoUpload: {
      type: Boolean,
      default: true
    },
    // 是否可以同时选择多个文件
    multiple: {
      type: Boolean,
      default: true
    },
    // 指定文件类型
    accept: {
      type: Object,
      default: null
    },
    // 生成缩略图
    thumb: {
      type: Object,
      default: null
    },
    // 压缩图片
    compress: {
      type: Object,
      default: null
    },
    // 提前把下一个文件准备好
    prepareNextFile: {
      type: Boolean,
      default: false
    },
    // 分片上传
    chunked: {
      type: Boolean,
      default: false
    },
    // 文件上传请求的参数表，每次发送都会发送此对象中的参数
    formData: {
      type: Object,
      default: function() {
        return {}
      }
    },
    // 限制文件数量
    fileNumLimit: {
      type: Number,
      default: undefined
    },
    // 限制所有文件总大小
    fileSizeLimit: {
      type: Number,
      default: undefined
    },
    // 限制单个文件大小
    fileSingleSizeLimit: {
      type: Number,
      default: undefined
    },
    // 取消去重
    duplicate: {
      type: Boolean,
      default: false
    },
    // 文件加入队列
    onPrview: {
      type: Function,
      default: files => {}
    },
    // 发送请求前, 可以对请求头做修改
    onBeforeSend: {
      type: Function,
      default: (object, data, headers) => {}
    },
    // 发送进度
    onProgress: {
      type: Function,
      default: (file, percentage) => {}
    },
    // 上传成功
    onSuccess: {
      type: Function,
      default: (file, res) => {}
    },
    // 上传失败
    onError: {
      type: Function,
      default: (file, err) => {}
    },
    // 上传成功和失败
    onAlways: {
      type: Function,
      default: file => {}
    },
    // 删除文件
    onRemove: {
      type: Function,
      default: (file) => {}
    }
  },
  mounted() {
    this.init()
  },
  data() {
    return {
      uploader: {}
    }
  },
  methods: {
    init() {
      // 初始化实例
      this.uploader = WebUploader.create({
        dnd: this.$refs.myDrager, // 拖拽区域
        disableGlobalDnd: this.$refs.myDrager,
        swf: UploaderSwf,
        server: this.action, // 上传服务器接口
        pick: {
          id: this.$refs.myPicker, // 上传按钮
          multiple: this.multiple //  多选
        },
        accept: this.accept, // 接收文件类型
        thumb: this.thumb, // 缩略图
        compress: this.compress, // 压缩图
        formData: this.formData,
        auto: this.autoUpload, // 自动上传
        prepareNextFile: this.prepareNextFile, // 提前准备下一个文件
        chunked: this.chunked, // 分片上传
        fileNumLimit: this.fileNumLimit,
        fileSizeLimit: this.fileSizeLimit,
        fileSingleSizeLimit: this.fileSingleSizeLimit,
        duplicate: this.duplicate
      })

      // 文件加入队列
      this.uploader.on('filesQueued', files => {
        files.map(v => {
          v.upStatus = '0'    // upStatus 0 加入队列 1 上传中 2 上传成功 3 上传失败
          this.fileList.push(v)
        })
        this.onPrview(files)
      })

      // 发送前
      this.uploader.on('uploadBeforeSend', this.onBeforeSend)

      // 上传进度
      this.uploader.on('uploadProgress', (file, percentage) => {
        // 把当前进入放到list的文件对象中，供视图显示进度, 用set方法可以令试图发生变化，不然没效果
        file.progress = percentage
        file.upStatus = '1'    // upStatus 0 加入队列 1 上传中 2 上传成功 3 上传失败
        this.changeArrItem(file, this.fileList)
        this.onProgress(file, percentage)
      })

      // 上传成功
      this.uploader.on('uploadSuccess', (file, res) => {
        file.upStatus = '2'    // upStatus 0 加入队列 1 上传中 2 上传成功 3 上传失败
        this.changeArrItem(file, this.fileList)
        this.onSuccess(file, res)
      })

      // 上传失败
      this.uploader.on('uploadError', (file, err) => {
        file.upStatus = '3'    // upStatus 0 加入队列 1 上传中 2 上传成功 3 上传失败
        this.changeArrItem(file, this.fileList)
        this.onError(file, err)
      })

      // 上传成功或失败
      this.uploader.on('uploadComplete', (file) => {
        file.progress = undefined
        this.changeArrItem(file, this.fileList)
        this.onAlways(file)
      })
    },
    // 手动上传
    submit() {
      this.uploader.upload()
    },
    // 移除列表中的文件
    deleteFile(file) {
      this.uploader.removeFile(file, true)
      // this.uploader.cancelFile(file)
      // 移除成功
      this.uploader.on('fileDequeued', file => {
        // 移除视图的
        this.fileList.splice(this.fileList.indexOf(file), 1)
        this.onRemove(file)
      })
    },
    // 获取进度
    getProgress(file) {
      if(!file.progress) {
        return '0%'
      }
      return Math.floor(file.progress * 100) + '%'
    },
    // 修改数组元素
    changeArrItem(item, arr) {
      arr.map((v, k) => {
        if (v.id === item.id) {
          this.$set(arr, k, item)
        }
      })
    }
  },
  watch: {
    fileList(val) {
      if(val.length === 0) {
        this.uploader.reset()
      }
    }
  }
}
</script>

<style lang="less">
@import url('./webuploader.css');
@import url('./iconfont.css');
@import url('./uploader.less');
</style>
