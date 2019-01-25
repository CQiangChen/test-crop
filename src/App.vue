<template>
  <div class="test">
    <input type="file" @change="uploadImg">
    <button @click="reCrop">重新编辑</button>
    <div class="cropper" v-if="isClick">
      <div class="cnt">
        <test-crop
          :imgSrc="imgSrc"
          :filename="filename"
          :config="config"
          @post="postFile"
          @handleCancel="handleCancel"
        >
          <template slot="end">
            <span class="btn-end">Cropper</span>
          </template>
          <template slot="cancel">
            <span class="btn-cancel">Cancel</span>
          </template>
        </test-crop>
      </div>
    </div>
    <img :src="new_img" alt>
  </div>
</template>

<script type='text/ecmascript-6'>
import TestCrop from './test-crop'
// import './node_modules/test-crop/test-crop.css'

export default {
  components: {
    TestCrop,
  },
  data () {
    return {
      imgSrc: '',
      new_img: "",
      isClick: false,
      filename:'',
      config: {
        Compression: {
          isTrue: true,  //开启压缩
        },
        options: {
          // viewMode: 1,
          minContainerWidth: 680,
          minContainerHeight: 360,
        }
      },
    }
  },
  methods: {
    uploadImg (e) {
      console.dir(e.target);
      var file = e.target.files[0];
      this.filename = file.name;
      console.log(file)
      var reader = new FileReader(file);
      //检测是不是文件
      if (file.type.split('/')[0] !== 'image') {
        alert('you should choose an image file');
        return;
      }
      //在onload中进行裁剪，显示
      reader.onload = () => {
        const img_data = reader.result;
        if (img_data.length > 0) {
          this.imgSrc = img_data;
          this.isClick = true;
        }
      };
      reader.readAsDataURL(file);
    },
    getFileUrl (inpt) {
      var url;
      if (navigator.userAgent.indexOf("MSIE") >= 1) { // IE 
        url = inpt.value;
      } else if (navigator.userAgent.indexOf("Firefox") > 0) { // Firefox 
        url = window.URL.createObjectURL(inpt.files.item(0));
      } else if (navigator.userAgent.indexOf("Chrome") > 0) { // Chrome 
        url = window.URL.createObjectURL(inpt.files.item(0));
      }
      return url;
    },
    reCrop () {
      this.isClick = true;
    },
    postFile (data) {
      console.log('---------返回结果-------')
      console.log(data)
      this.isClick = false;
      this.new_img = window.URL.createObjectURL(data.file);

      const url = '';
      const formFile = new FormData();
      formFile.append('file', data.file);

      const xhr = new XMLHttpRequest();
      xhr.open('post', url, true)//post方式，url为服务器请求地址，true 该参数规定请求是否异步处理
      xhr.onload = this.uploadComplete; //请求完成
      xhr.onerror = this.uploadFailed; //请求失败

      // xhr.upload.onprogress = progressFunction;//【上传进度调用方法实现】
      xhr.upload.onloadstart = function () {//上传开始执行方法
        // ot = new Date().getTime();   //设置上传开始时间
        // oloaded = 0;//设置上传开始时，以上传的文件大小为0
      };

      xhr.send(formFile); //开始上传，发送form数据
      // this.new_img = data;
    },
     uploadComplete (evt) {
      var data = JSON.parse(evt.target.responseText);
      console.log(data)
    },
    uploadFailed (evt) {
      alert("上传失败！");
    },
    handleCancel () {
      this.isClick = false;
      this.imgSrc = null;
    }

  }
}
</script>

<style scoped>
.cropper {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.3);
}
.cnt {
  width: 698px;
  height: 514px;
  background: white;
  overflow: hidden;
}
.btn-end,
.btn-cancel {
  display: inline-block;
  width: 140px;
  height: 50px;
  background: rgba(0, 211, 238, 1);
  border-radius: 25px;
  text-align: center;
  line-height: 50px;
  color: white;
  font-size: 18px;
}
.btn-end {
  margin-right: 40px;
}
</style>
