<template>
  <div class="simple-crop">
    <div class="bottom">
      <img ref="cropImg" :src="imgSrc" style="display:block;max-width:50%" @load="begin">
      <div class="btn">
        <div @click="end">
          <slot name="end"></slot>
        </div>
        <div @click="cancel">
          <slot name="cancel"></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script type='text/ecmascript-6'>
import Cropper from 'cropperjs'
import lrz from 'lrz';
import 'cropperjs/dist/cropper.css';

export default {
  // props: ['imgSrc', 'config','filename','quality'],
  props:{
    imgSrc:{
      type:String
    },
    config:{
      type:Object
    },
    filename:{
      type:String
    },
    quality:{
      type:Number,
      default:0.6
    }
  },
  data () {
    return {
      isBegin: false,
      cropper: null,
    }
  },
  methods: {
    begin () {
      this.isBegin = true;
      var img = this.$refs.cropImg;
      var bodyH = document.body.offsetHeight;
      var bodyW = document.body.offsetWidth;
      console.dir(img);
      var defaultOption = {
        aspectRatio: 17 / 10,
        minContainerWidth: bodyW,
        minContainerHeight: bodyH,
        // viewMode:1,
        preview: '.showImg',
        crop (event) {
          console.log(event);
          console.log(event.detail.y);
          console.log(event.detail.width);
          console.log(event.detail.height);
          console.log(event.detail.rotate);
          console.log(event.detail.scaleX);
          console.log(event.detail.scaleY);
        },
      }
      var options = Object.assign(defaultOption, this.config.options);
      console.log(options);
      this.cropper = new Cropper(img, options)
    },
    end () {
      console.log('end');
      this.cropper.getData();
      console.dir(this.cropper.getCroppedCanvas())
      this.cropper.getCroppedCanvas().toBlob((blob) => {
        console.log(blob);
        const formData = new FormData();
        formData.append('croppedImage', blob);
        console.log(formData.get('croppedImage'))
        if (this.config.Compression.isTrue) {
          this.beforeCompression(formData.get('croppedImage'));     //如果开启压缩
        } else {
          this.$emit('post', formData.get('croppedImage'))      //如果不开启压缩
        }
      })
    },
    beforeCompression (file) {
      var maxWidth = 300;
      // var file = formData.get('croppedImage');
      var reader = new FileReader(file);

      console.log(file);
      console.log(reader);
      //检测是不是文件
      if (file.type.split('/')[0] !== 'image') {
        alert('you should choose an image file');
        return;
      }

      //在onload中进行显示,压缩,上传
      reader.onload = () => {
        const ONE_MB = 1024 * 1024;
        const img_data = reader.result;
        // var imgEle = img_data; //this.$refs.imgElement;
        var image = new Image();		  // 创建一个image对象，供canvas绘图使用
        image.src = reader.result;		  // this.result即base64的数据
        image.onload = () => {

          if (img_data.length > 0) {
            /*
           * 这里的img_data就是图片地址，经过base64编码的可以直接传到服务器，
           *  也可以直接在img标签的src属性里去展示
           *
           */
            if (file.size > ONE_MB) {
              //图片过大时可设置透明，不可操作
            }
            // //图片的预览
            // this.header_img = img_data;
            // // this.fileUpload = file;
            // this.$nextTick(() => {
            //   this.option.img = img_data;
            //   this.isShowCropper = true
            //   console.log(this.option.img)
            // })

            this.handleCompress(image.src);
          }
        };
      }
      reader.readAsDataURL(file);
    },
    handleCompress(img){
      var self = this;
      lrz(img,{quality: self.quality}).then((res)=>{
        console.log(res)
        self.$emit('post',res)
      })
    },
    // handleCompress (img, maxWidth, mimeType, rotateDeg) {
    //   //创建画布
    //   var cvs = document.createElement('canvas');
    //   var width = img.naturalWidth,
    //     height = img.naturalHeight,
    //     imgRatio = width / height;
    //   //
    //   if (width > maxWidth) {
    //     width = maxWidth;
    //     height = width / imgRatio;
    //   }
    //   cvs.width = width;
    //   cvs.height = height;
    //   var ctx = cvs.getContext("2d");
    //   var destX = 0,
    //     destY = 0;
    //   if (rotateDeg) {
    //     ctx.translate(cvs.width / 2, cvs.height / 2);
    //     ctx.rotate(rotateDeg);
    //     destX = -width / 2;
    //     destY = -height / 2;
    //   }
    //   ctx.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight, destX, destY, width, height);
    //   //图片质量进行适当压缩
    //   var quality = width >= 1200 ? 0.6 :
    //     width > 600 ? 0.8 : 1;
    //   //导出图片为base64
    //   var newImageData = cvs.toDataURL(mimeType, quality);
    //   var newFile = this.dataURLtoFile(newImageData,this.filename)
    //   // var resultImg = new Image();
    //   console.log(newFile);
    //   this.$emit('post', newFile.get('croppedImage'));
    //   // this.new_img = newImageData;
    //   // resultImg.src = newImageData;
    //   // this.isClick = false
    //   // return resultImg;
    //   // console.log(resultImg);
    // },
    dataURLtoFile (dataurl, filename) { //将base64转换为文件

      var arr = dataurl.split(','),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n);

      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      const formData = new FormData();
      formData.append('croppedImage', new File([u8arr], filename, {
        type: mime
      }));
      return formData;

    },
    convertBase64UrlToBlob (urlData) {
      let bytes = window.atob(urlData.split(',')[1]);//去掉url的头，并转换为byte
      //处理异常,将ascii码小于0的转换为大于0
      let ab = new ArrayBuffer(bytes.length);
      let ia = new Uint8Array(ab);
      for (var i = 0; i < bytes.length; i++) {
        ia[i] = bytes.charCodeAt(i);
      }
      const formData = new FormData();
      formData.append('croppedImage', new Blob([ab], { type: 'image/jpeg' }));
      return formData;
    },
    cancel () {
      this.$emit('handleCancel')
    }
  },
}
</script>

<style scoped>
/* .simple-crop {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 3000;
} */
#cropEndBtn {
  position: absolute;
  bottom: 0;
  z-index: 3001;
}
.bottom {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
/deep/ .cropper-container {
  height: 100%;
  width: 100%;
}
.showImg {
  width: 160px;
  height: 90px;
  overflow: hidden;
}
.btn {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-top: 18px;
}
</style>
