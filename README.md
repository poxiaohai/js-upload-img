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

如果使用canvas对图片进行压缩，具有一定的兼容性问题。由于在ios系统中如果处理的图片超过2M则会自动中断，简单查可一下大神介绍说可以分段上传。这里不详细介绍，感兴趣的同学可以参考这个[https://github.com/stomita/ios-imagefile-megapixel](https://github.com/stomita/ios-imagefile-megapixel)。

### 基于canvas的上传方法

参考链接：

http://www.cnblogs.com/stoneniqiu/p/5957356.html

https://www.zhihu.com/question/30692677#answer-15055715



