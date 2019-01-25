
# 简单版截图，暂时制作截图+压缩（自用，还未完成)

**我是工具搬运工，搬这搬那，合在一起方便自己用！**

>因为看到有人下载使用，所以先声明下：如果有人需要使用先到一个新的项目中引用试一下，因为暂时为自己的一个项目简单写的，所以很多没有考虑到，不建议使用，如果尝试使用首先建议使用vue，其次可以参考下面的方法。

**使用的插件：**
截图工具：[cropperjs](https://github.com/fengyuanchen/cropperjs)
压缩工具：[lrz](https://www.npmjs.com/package/lrz)

**该插件是以vue-cli3.x的快速原型开发进行开发，如想自己编辑开发，下载后，操作如下：**

```
npm install -g @vue/cli-service-global
//进入src下
vue serve App.vue
```

## 下载：
```
npm install --save test-crop
```

## 使用：
注：需要配合input使用，传入imgSrc,具体使用可以参考src目录下App.vue的引用

**以下例子以Vue为主**

**1、引入：**
```
//局部引入
import TestCrop from 'test-crop'
import 'test-crop/test-crop.css'
```

**2、创建input,注册change事件，加入该方法**
```
<input type="file" @change="uploadImg">
```
注：imgSrc：传入截图工具的图片地址，isShow：手动控制截图工具的显示隐藏（默认false）
```
uploadImg (e) {
  //...其他操作
  var file = e.target.files[0];
  var reader = new FileReader(file);
  //检测是不是文件
  if (file.type.split('/')[0] !== 'image') {
    alert('you should choose an image file');
    return;
  }
  //
  reader.onload = () => {
    const img_data = reader.result;
    if (img_data.length > 0) {
      this.imgSrc = img_data; //图片地址
      this.isShow = true; //控制是否显示
    }
  };
  reader.readAsDataURL(file);
  //...其他操作
}
```

**3、页面引入组件：**
```
 <test-crop
    :imgSrc="imgSrc"
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
```
imgSrc：如2所说，传入图片的数据base64

config:
```
config: {
  Compression: {
    isTrue: true,  //是否开启压缩
  },
  options: {    
    //该配置是cropperjs的配置项，支持大部分的内容，可以参考cropperjs
    // viewMode: 1,
    minContainerWidth: 680, //此处按照需求改变，手机端可以采用获取全屏的大小
    minContainerHeight: 360,
  }
},
```
post：返回事件，返回值为base64,blob,file，进行上传操作

handleCancel：返回事件，返回是否点击取消处理，手动对截图工具控制隐藏

其他：
按钮样式自己设置，整个test-crop可以套上div对其限制位置显示，注意必要时候加上overflow:hidden

