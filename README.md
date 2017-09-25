原生js实现图片上传、预览、压缩
=========================

由于项目需求，需要在web端实现上传用户的头像和相册。那么问题来了。是使用原始的js来实现，还是使用第三方的JS插件。搜索了一下发现这类插件大小在 10K-100+K 不等，我就实现一个图片的上传没必要还引入一个插件来增加页面请求，所以选择使用原生js上传图片。

## 首先考虑的是form表单提交图片

这里由于还需要对图片进行上传前的预览和压缩操作，选择使用 `new FormData()` 方式拼装表单数据，并通过xhr对象上传至后台。

```html
	
<!-- 单张上传 -->
<input type="file" id="uploadImg">

<!-- 多张上传 -->
<input type="file" multiple id="uploadImg">

```

[HTMLInputElement.multiple 参见MDN文档](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/multiple)

> 获取input文件，验证是否为图片，生成预览图片并拼装在FormData中

```js

// h5 file reader
function getBase64( thisFiles ) {
	var reader = new FileReader();
	reader.readAsDataURL( thisFiles );
	reader.onload = function(e){
		console.log('预览图片的base64编码为：' + this.result);
	}
};

// send form data
function sendData( file ) {
	var formData = new FormData();
	formData.append('file', file);
	
	var request = new XMLHttpRequest();
	request.onload = function() {
	    console.log("上传成功！");
	}
	request.open("POST", "http://foo.com/submitform.php");
	request.send(formData);
};

var upload = document.getElementById('upload');

upload.addEventListener('change', function(){
	var file = upload.files[0];
	
	if( !upload.value.match(/.jpg|.jepg|.gif|.png|.bmp/i) ){
		console.log('上传图片的格式不正确，请重新选择~');
	} else {
		getBase64( file );
		sendData( file );
	}
});

```

[FormData 对象的使用 参见MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects)

**这里重点说一下，formdata 对象中append进去的数据是私有化的，直接输出是取不到的，需要使用form.get('参数名')的方式获取。**

当然，如果后台需要传base64编码的文件那就不需要拼装在FormData中。直接取到预览的base64编码字符串，就可以直接发送了。POST限定的发送字符串大小一般都在8M以上，后台可以控制接受字符串的大小。

## 对图片进行压缩

由于在ios系统中如果`canvas`处理的图片超过2M则会自动中断，大神介绍说可以分段上传。这里不详细介绍，感兴趣的同学可以参考这个[https://github.com/stomita/ios-imagefile-megapixel](https://github.com/stomita/ios-imagefile-megapixel)。

### 基于canvas的压缩方法

压缩js主逻辑代码

```js

var thisImg = document.querySelector('#img');

//  压缩图片需要的一些元素
var reader = new FileReader(), img = new Image();

// 选择的文件对象
var file = null;

// 缩放图片需要的canvas
var canvas = document.createElement('canvas');
var context = canvas.getContext('2d');

// base64地址图片加载完毕后
img.onload = function() {
  // 图片原始尺寸
  var originWidth = this.width;
  var originHeight = this.height;

  // 最大尺寸限制
  var maxWidth = 400, maxHeight = 400;

  // 目标尺寸
  var targetWidth = originWidth, targetHeight = originHeight;

  // 图片尺寸超过400*400的限制
  if (originWidth > maxWidth || originHeight > maxHeight){
      if (originWidth / originHeight > maxWidth / maxHeight) {

          // 更宽，按照宽度限定尺寸
          targetWidth = maxWidth;
          targetHeight = Math.round(maxWidth * (originHeight / originWidth));
      } else {
          targetHeight = maxHeight;
          targetWidth = Math.round(maxHeight * (originWidth / originHeight));
      }
  }

  // canvas对图片进行缩放
  canvas.width = targetWidth;
  canvas.height = targetHeight;

  // 清除画布
  context.clearRect(0, 0, targetWidth, targetHeight);

  // 图片压缩
  context.drawImage(img, 0, 0, targetWidth, targetHeight);

  // canvas转为blob并上传
  canvas.toBlob(function (blob) {

      // 图片ajax上传
      var xhr = new XMLHttpRequest();

      // 文件上传成功
      xhr.onreadystatechange = function() {
          if ( xhr.status === 200 ) {
              // xhr.responseText 就是返回的数据
          }
      };

      // 开始上传
      xhr.open('POST', 'upload.php', true);
      xhr.send( blob );
  }, file.type || 'image/png');
};

// 文件base64化，以便获取图片原尺寸
reader.onload = function(){
  img.src = this.result;
};
thisImg.addEventListener('change', function (event){
  file = event.target.files[0];

  // 选择的文件是图片
  if (file.type.indexOf('image') === 0){
      reader.readAsDataURL( file );
  }
});

```

涉及到的几个api：

* [HTMLCanvasElement.toBlob() MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toBlob)
* [CanvasRenderingContext2D.clearRect() MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/clearRect)
* [CanvasRenderingContext2D.drawImage()  MDN文档](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage)
* [HTMLCanvasElement.getContext() MDN文档](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/getContext)

疑问： 为什么缩小尺寸的图片转blob对象的size会比原图尺寸转blob对象的size大很多为什么？但上传的压缩图片的大小确实会比原图的大小小很多。

参考链接：

http://www.cnblogs.com/stoneniqiu/p/5957356.html

https://www.zhihu.com/question/30692677#answer-15055715

http://www.zhangxinxu.com/study/201707/js-compress-image-before-upload.html



