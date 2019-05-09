---
title: Phonegap jQueryMobile 在线应用拍照、相册选择上传+预览
date: 2015-07-07 10:53:25
tags:
- WebApp
---
拍照上传以及从相册中选择图片上传是大多数手机应用的典型功能，这几天在做一个移动端的项目，其中就用到该功能。
  
  项目技术选择的是Phonegap+jQueryMobile，因为jQuery自己比较熟悉，用jQueryMobile没有什么大的难点。
  
  界面用jQueryMobile搭的很快，一开始打算用Phonegap将应用做为离线版本，但后来发现离线版本升级的话还是和普通的app一样，都需要重新安装，这就没有了WebApp的优势和亮点，故最终选择了在线版本，即将项目作为一个web应用，用Tomcat发布在服务器上，Phonegap作为一个app外壳行使浏览器功能在其中请求服务器上的页面等资源。
  
  做该功能的时候，由于对于Phonegap接触不久，花了我整整两天的时间，这两天没有做其他的，一直在研究如何实现图片预览，网上、官网有很多成功的例子，但没有一个在我这边成功预览的，几乎看遍了网上所有的帖子，点坏了鼠标！心情坏到了极点，因为本以为很简单的东西，而后网上说的也都很简单，很多帖子都是一样的内容，但是我却死活跑不通，崩溃！
  
  到最后，才发现问题原来出在我的“在线”应用上，无语，其实做J2EE的我深知浏览器端是无权限操作本地文件的，但是由于对Phonegap的不了解，由于知道Phonegap可以直接操作本地资源，误认为用了Phonegap就无敌了，你以为呢！于是我就把WebApp整成离线的试下喽，果然，可以显示了！
  
  但项目已经做到这个地步了，再把它重构成离线的？不可取。于是我就开始分析了。。。
  
  Phonegap不是可以操作本地文件吗？html中的img标签不能根据本地图片路径渲染图片，但它可以不用文件路径啊！于是呵呵了。
  
  最终解决方案是用Phonegap的FileReader来根据本地图片路径以base64的形式读取其内容，base64是可以直接在img标签上渲染的，所以问题就解决了。
  
  下面截取拍照上传以及预览的代码如下：
  
  图片处理代码：
```javascript
var picUrl = "" ;
function capturePhotoUrl() {
 
     navigator.camera.getPicture(onCaptureSuccess, onUrlFail, {
           quality : 80,
//         allowEdit : true,//在Android中此配置忽略
           destinationType : Camera.DestinationType.FILE_URI,
           sourceType:Camera.PictureSourceType.CAMERA,
           targetWidth : 800, // 生成的图片大小 单位像素
           targetHeight : 640
     });
 
}
function onCaptureSuccess(imageURI) {
     picUrl = imageURI;
     //这是关键部分
     window.resolveLocalFileSystemURI(imageURI, function(fileEntry){
           fileEntry.file( function(file){
                 var reader = new FileReader();
                    reader.onloadend = function(evt) {
                     $( "#uploadImgDom").append("<img path='"+imageURI+"' class='uploadImg' src='"+evt.target.result+"' />");
                     $( "#report-page").trigger("create" );
                    };
                    reader.readAsDataURL(file);
           }, readFileFail);
     }, readFileFail);
//   $("#urlinfo").text("图片的原始路径" + imageURI);
}
 
function readFileFail(evt){
     navigator.notification.alert( "文件读取失败，原因：" + evt.code, null, "警告" );
}
 
function onUrlFail(message) {
     navigator.notification.alert( "失败，原因：" + message, null, "警告" );
}
 
function loadImageLocal() {
     navigator.camera.getPicture(onLoadSuccess, onUrlFail, {
           quality : 80,
           sourceType : Camera.PictureSourceType.PHOTOLIBRARY,
           destinationType : Camera.DestinationType.FILE_URI,
//         encodingType : Camera.EncodingType.JPEG,
//         mediaType : Camera.MediaType.PICTURE,
           targetWidth : 800, // 生成的图片大小 单位像素，选择图片的时候一定要制定这个值，否则
           targetHeight : 640
     });
 
}
 
function onLoadSuccess(imageURI) {
     imageURI = imageURI + ".jpg";
     picUrl = imageURI;
     window.resolveLocalFileSystemURI(imageURI, function(fileEntry){
           fileEntry.file( function(file){
                 var reader = new FileReader();
                    reader.onloadend = function(evt) {
                     $( "#uploadImgDom").append("<img path='"+imageURI+"' class='uploadImg' src='"+evt.target.result+"' />");
                     $( "#report-page").trigger("create" );
                    };
                    reader.readAsDataURL(file);
           }, readFileFail);
     }, readFileFail);
//   $("#urlinfo").text("图片的原始路径" + picUrl);
}
 
function uploadPhoto() {
     $("#report-btn").attr({ "disabled": "disabled"});
     var options = new FileUploadOptions();
     options.fileKey = "file";
     options.fileName = picUrl.substr(picUrl.lastIndexOf( '/') + 1);
     options.mimeType = "image/jpeg";
     
     var ft = new FileTransfer(); //文件上传类
    ft.onprogress = function (progressEvt) { //显示上传进度条
        if (progressEvt.lengthComputable) {
            navigator.notification.progressValue(Math.round(( progressEvt.loaded / progressEvt.total ) * 100));
        }
    }
    navigator.notification.progressStart( "提醒", "当前上传进度" );
     
     var params = new Object();
     params.title = $( "#textinput-2").val();
     params.info = $( "#textarea-2").val();
     params.longitude = $( "#longitude").text();
     params.latitude = $( "#latitude").text();
     
     options.params = params;
     
     
     ft.upload(picUrl, basePath+ "/report", win,
                fail, options);
}
 
function win(r) {
     navigator.notification.progressStop(); //停止进度条
     
     $("#returnpic").attr( "src",
                basePath+ "/files/" + r.response);
 
//   $("#returninfo").html(
//              "上传成功\n：反馈的信息:r.responseCode:" + r.responseCode + "\nr.response:"
//                         + r.response + "\nr.bytesSent:" + r.bytesSent);
     $("#report-btn").removeAttr( "disabled");
     navigator.notification.alert( "上传成功！" , function(){
           $( "#reportPageBackBtn").click();
     }, "提醒");
}
 
function fail(error) {
 
     /*
      * FileTransferError.FILE_NOT_FOUND_ERR：1 文件未找到错误。
      * •FileTransferError.INVALID_URL_ERR：2 无效的URL错误。
      * •FileTransferError.CONNECTION_ERR：3 连接错误。 FileTransferError.ABORT_ERR =
      * 4; 程序异常
      */
     var errorcode = error.code;
     var errstr = "";
     switch (errorcode) {
     case 1: {
           errstr = "错误代码1：源文件路径异常，请重新选择或者拍照上传！" ;
            break;
     }
     case 2: {
           errstr = "错误代码2:目标地址无效,请重试！" ;
            break;
     }
     case 3: {
           errstr = "您手机或者后台服务器网络异常,请重新上传！" ;
            break;
     }
     default: {
           errstr = "程序出错";
            break;
     }
 
     }
     $("#returninfo").text(
                 "上传失败,错误代码:" + errstr + "上传源文件:" + error.source + "目标地址:"
                           + error.target + "请重新上传！" );
    
     $("#report-btn").removeAttr( "disabled");
}
```
前台显示代码：
```html
<div role= "main" class = "ui-content jqm-content" data-iscroll ="content" >
  <div class = "ui-grid-a">
    <div class = "ui-block-a">
        <a href = "#" onclick ="capturePhoto()" class = "ui-shadow ui-btn ui-btn-b ui-corner-all ui-btn-icon-left ui-icon-camera"> 拍照</ a>
    </div>
    <div class = "ui-block-b">
         <a href = "#" onclick ="loadImageLocal()" class= "ui-shadow ui-btn ui-btn-b ui-corner-all ui-btn-icon-left ui-icon-heart"> 相册</a>
    </div>
</div>
<div id = "returninfo"></div>
<div id = "uploadImgDom"></div>
```
  后台上传的代码就不贴了，就一个Servlet。
  
  谨记我这两天的精彩编程生活。
  
  <font color=red>最后感谢所有网友的帮助与分享！</font>