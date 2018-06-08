### 缘起
项目本来采用的`elementUI`，但是项目需要兼容到ie9，`elementUI`里面有些组件是不支持ie9的，其中就包括上传组件，所以找来`webuploader`，改装成一个vue组件在项目中使用。

### 使用
需要在index.html先引入jquery库
- 引入组件，加载
```
import uploader from '@/components/uploader/uploader.vue'

export default {
  components: {
    uploader
  }
}
``` 


### uploader参数文档

- 可传参数

  `button` 上传按钮的文字。类型：`String`，默认值： `点击选择`

  `fileList` 上传文件列表。类型：`Array`，默认值： `[]`
  
  `drag` 是否支持拖拽上传。类型：`Boolean`，默认值： `false`

  `dragTip` 拖拽框提示语。 类型：`String`，默认值： `'或者把文件拖到这里，单次不要超过300个'`

  `action` 上传服务器地址。 类型：`String`，默认值： `''`

  `autoUpload` 是否自动上传。 类型：`Boolean`，默认值： `true`

  `multiple` 是否可以同时选择多个文件。 类型：`Boolean`，默认值： `true`
  
  `accept` 指定文件类型。 类型：`Object`，默认值： `null`。使用参考`webuploader`

  `thumb` 生成缩略图。 类型：`Object`，默认值： `null`。使用参考`webuploader`

  `compress` 压缩图片。 类型：`Object`，默认值： `null`。使用参考`webuploader`
  
  `prepareNextFile` 提前准备下一个文件。 类型：`Boolean`，默认值： `false`。使用参考`webuploader`

  `chunked` 分片上传。 类型：`Boolean`，默认值： `false`。使用参考`webuploader`

  `limit` 限制文件数量。 类型：`Number`，默认值： `undefined`。使用参考`webuploader`

  `fileSizeLimit` 限制所有文件总大小。 类型：`Number`，默认值： `undefined`。使用参考`webuploader`

  `fileSingleSizeLimit` 限制单个文件大小。 类型：`Number`，默认值： `undefined`。使用参考`webuploader`

  `duplicate` 取消去重。 类型：`Boolean`，默认值： `false`。使用参考`webuploader`
  
  `formData` 文件上传请求的参数表。 类型：`Object`，默认值： `{}`。使用参考`webuploader`


- 方法，也以 `v-bind`绑定

  - `onPrview(files)` 文件加入队列中。  
    
    `files`,文件数组  类型: 数组
    
  - `onBeforeSend(object, data, headers)` 发送文件之前，可以对headers做调整。  
      
  - `onProgress(file)` 文件上传中的钩子函数。  
    
    `file`,文件数组  类型: 数组
  
  - `onSuccess(file, res)` 上传成功。  
    
    `file`, 文件  类型: 文件对象
    
    `res`, 后端返回参数  类型: 文件对象
    
  - `onError(file, err)` 上传失败。  
    
    `file`, 文件  类型: 文件对象
    
    `err`, 错误码  类型: 字符串，具体含义参考`webuploader`

  - `onAlways(file, err)` 上传成功和失败的回调。
  
    `file`, 文件  类型: 文件对象
  
  - `onRemove(file)` 移除文件回调。
  
    `file`, 文件  类型: 文件对象
  


### ie9下报错406  

检查发现IE上传时的请求头中，Accept: text/*

而Chrome的请求头中，Accept: */*

此时，需要后端配合设置 `ignoreAcceptHeader` 为`true`, 就可以正常上传了
   