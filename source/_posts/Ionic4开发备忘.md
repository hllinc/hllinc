---
title: Ionic4开发备忘
date: 2019-12-20 15:26:10
tags:
categories:
---
Ionic是一款跨平台开发App的框架，详细介绍见官网：[https://ionicframework.com](https://ionicframework.com/docs/cli)

1. 环境搭建

全局安装NodeJS和Ionic最新稳定版，安装命令如下：

```
npm i -g cordova ionic
```
2. 新建项目

初始化项目命令如下：

```
ionic start myIonicApp tabs
```
其中的tabs为初始化项目模板，可选模板有blank,tabs,sidemenu,tutorial等，可以运行ionic start --list命令查看所有可用模板。
3. 项目常用命令

详细命令参考官网：[https://ionicframework.com/docs/cli](https://ionicframework.com/docs/cli)

  1. 启动项目

ng serve -o

  2. 编译项目
```
ionic cordova prepare ios
ionic cordova prepare android
ionic cordova run android --prod --release
ionic cordova run ios --prod --release
ionic cordova build android --prod --release
ionic cordova build ios --prod --release
```
  3. 管理平台

添加平台：

```
ionic cordova platform add android
```
删除平台：
```
cordova platform rm ios
```
根据配置文件安装平台：
```
cordova prepare
```
  4. 管理插件

添加插件：

```
ionic cordova plugin add cordova-sqlite-storage
```
删除插件：
```
ionic cordova plugin rm cordova-sqlite-storage
```
根据配置文件安装插件：
cordova prepare

  5. 自动创建代码命令
```
ionic generate page pages/my-page
```
4. 路由传参

路由传参只能传递字符串参数，不能为复杂对象，当然也可以将复杂对象转成json再通过路由进行传参，但这种方式不被推荐，某些情况下转换的对象json中存在特殊字符的话会直接影响路由功能。

前端有localStorage这个比较强大的前端缓存工具，因为路由传参不适合传递复杂对象，我们还有一种方式就是将复杂对象存储在localStorage中，它的key放在路由参数中进行传递，然后到详情页面中通过这个key从localStorage中查询。显然，某些场景下也会出现下面的问题，但这的确是一个方法。关键前端缓存这块后面弄个专题说一下。

在大部分场景中，比如查看订单列表中某一条数据的详细信息，通过路由传递一个id字段到详情页面就足够了，虽然前端可以缓存订单列表中已有的订单字段，但为了数据的准确性，这里还是推荐在详情页面根据id重新获取最新的详情数据。当然，得结合实际业务功能决定到底用何种方式。

路由配置代码：

```
const routes: Routes = [
	  {path: 'order-detail/:id', loadChildren: './pages/order-detail/order-detail.module#OrderDetailPageModule'}
];
```
传递参数代码：
```
<ion-item routerLink="/order-detail/3"></ion-item>
```
接收参数代码：
```
constructor(private route: ActivatedRoute) {
}
ngOnInit() {
 const id = this.route.snapshot.paramMap.get('id');
}
```
5. 组件传参

这里的组件传参就可以直接传递复杂参数，当然它同样不适合上面我们所说的查看详情等场景，但在以下场景中它将为我们带来极大的方便。

涉及到流程表单的场景：用户在前端需要操作一个流程表单，分好几步，第一步干嘛，第二步干嘛，每一步都会在前端录入一些必要数据，而且这些数据可能在下一步的表单操作中要使用到，此时，组件传参就发挥了它的威力。

在使用该component的module中需要配置entryComponents，代码如下：

```
@NgModule({
 declarations: [ReceiveOrderPage],
 entryComponents: [ReceiveOrderPage]
})
```
在传递参数的component中代码如下：
```
constructor(private modalCtrl: ModalController) {
}
async receiveOrder() {
 const modal = await this.modalCtrl.create({
   component: ReceiveOrderPage,
   componentProps: {
     params: {id: 1, name: 'hllinc'}
   },
   animated: true
 });
 return await modal.present();
}
```
接收参数代码：
```
export class ReceiveOrderPage implements OnInit {
 @Input() params;
}
<ion-content padding>
 <p>{{params.name}}</p>
</ion-content>
```
6. 禁用菜单

当我们使用menu功能时，一般会在全局添加menu左或右滑动唤出的功能，但在某些场景下，我们只希望在主页面中启用该功能，在子页面中禁用该功能，因为左右滑动在子页面可能会实现其他必要功能，此时我们需要在子页面中添加如下代码以禁用菜单唤出功能，代码如下：

```
constructor(private menuCtrl: MenuController) {
}
ionViewWillEnter() {
 this.menuCtrl.enable(false);
}
```
7. Flex布局

某些情况下，采用Flex可以为布局带来更好的效果，css代码如下：

```
.dayPlat {
display: flex;
flex-wrap: wrap;
flex-direction: row;
align-content: flex-start;
.day {
 font-size: 14px;
 width: 60px;
 height: 60px;
 text-align: center;
 line-height: 60px;
 margin: 10px;
 border: 1px solid #dddddd;
 color: #555;
 -webkit-border-radius: 5px;
 -moz-border-radius: 5px;
 border-radius: 5px;
} 
```
效果如下图所示：
![图片](https://uploader.shimo.im/f/a1oAw02RFIw8UdD6.png!thumbnail)

8. Modal窗口

对于一些页面的打开方式我们可能不需要像普通页面那样具有返回功能，在打开的页面中提供一个关闭按钮，并且打开效果也不像普通页面那样从右向左滑入，而是具有弹出效果的打开，以达到提醒用户当前打开页面中的内容很重要的目的。比如，协议页面、登录/注册页面等。

打开Modal窗口，代码如下：

```
constructor(private modalCtrl: ModalController) {
}
async openProtocol() {
 const modal = await this.modalCtrl.create({
   component: ProtocolPage
 });
 return await modal.present();
}
```
关闭Modal窗口需要在打开的component中添加如下代码：
```
<ion-button color="primary" (click)="closeModal()">
 <ion-icon name="close" color="light"></ion-icon>
</ion-button>
constructor(private modalCtrl: ModalController) { }
closeModal() {
 this.modalCtrl.dismiss();
}
```
当然，这个modal中的component也是可以进行传参的，方式同组件传参。
9. 拍照

脚本代码如下：

```
import {Component, OnInit} from '@angular/core';
import {Camera, CameraOptions} from '@ionic-native/camera/ngx';
@Component({
 selector: 'app-camera',
 templateUrl: './camera.page.html',
 styleUrls: ['./camera.page.scss'],
})
export class CameraPage implements OnInit {
 imageData;
 constructor(private camera: Camera) {
 }
 ngOnInit() {
 }
 takePhone() {
   const options: CameraOptions = {
     quality: 100,
     destinationType: this.camera.DestinationType.DATA_URL,
     encodingType: this.camera.EncodingType.JPEG,
     mediaType: this.camera.MediaType.PICTURE
   };
   this.camera.getPicture(options).then((imageData) => {
     // imageData is either a base64 encoded string or a file URI
     // If it's base64 (DATA_URL)
     let base64Image = 'data:image/jpeg;base64,' + imageData;
     this.imageData = base64Image;
   }, (err) => {
     // Handle error
   });
 }
}
```
视图代码如下：
```
<ion-header>
 <ion-toolbar>
   <ion-buttons slot="start">
     <ion-back-button [text]="'返回'"></ion-back-button>
   </ion-buttons>
   <ion-title>camera</ion-title>
 </ion-toolbar>
</ion-header>
<ion-content padding>
 <ion-button (click)="takePhone()">拍照</ion-button>
 <ion-img [src]="imageData"></ion-img>
</ion-content>
```
10. 生命周期

Ionic4中的生命周期函数和angualr7基本是一样的，下面我们看看Ionic4中的生命周期函数，以及生命周期函数的用法。  

**Ionic4中内置的生命周期函数：**

**ionViewWillEnter **—当进入一个页面时触发(如果它从堆栈返回)

**ionViewDidEnter **—进入后触发

**ionViewWillLeave **—如果页面将离开触发

**ionViewDidLeave **— 在页面离开后触发

**ionViewWillUnload **— 在Angular中没有触发，因为这里你必须使用ngOnDestroy

**Ionic4中使用Angular生命周期函数：**

1、Ionic4中的生命周期函数**ngOnChanges **当被绑定的输入属性的值发生变化时调用(父子组件传值的时候会触发)

2、Ionic4中的生命周期函数**ngOnInit **请求数据一般放在这个里面  （重要*）

3、Ionic4中的生命周期函数**ngDoCheck** 检测，并在发生 Angular 无法或不愿意自己检测的变化时作出反应

4、Ionic4中的生命周期函数 **ngAfterContentInit **当把内容投影进组件之后调用

5、Ionic4中的生命周期函数 **ngAfterContentChecked **每次完成被投影组件内容的变更检测之后调用

6、Ionic4中的生命周期函数 **ngAfterViewInit  **初始化完组件视图及其子视图之后调用（dom操作放在这个里面） （重要）

7、Ionic4中的生命周期函数 **ngAfterViewInit**  每次做完组件视图和子视图的变更检测之后调用

8、Ionic4中的生命周期函数  **ngOnDestroy  **组件销毁后执行 （重要）

```
constructor() { 
  console.log('00构造函数执行了---除了使用简单的值对局部变量进行初始化之外，什么都不应该做')
}
ngOnChanges() {
  console.log('01ngOnChages执行了---当被绑定的输入属性的值发生变化时调用(父子组件传值的时候会触发)'); 
}
ngOnInit() {
    console.log('02ngOnInit执行了--- 请求数据一般放在这个里面');     
}
ngDoCheck() {
    //写一些自定义的操作
    console.log('03ngDoCheck执行了---检测，并在发生 Angular 无法或不愿意自己检测的变化时作出反应');
    if(this.userinfo!==this.oldUserinfo){
        console.log(`你从${this.oldUserinfo}改成${this.userinfo}`);
        this.oldUserinfo = this.userinfo;
    }else{            
        console.log("数据没有变化");          
    }
}
ngAfterContentInit() {
    console.log('04ngAfterContentInit执行了---当把内容投影进组件之后调用');
}
ngAfterContentChecked() {
    console.log('05ngAfterContentChecked执行了---每次完成被投影组件内容的变更检测之后调用');
}
ngAfterViewInit(): void {     
    console.log('06 ngAfterViewInit执行了----初始化完组件视图及其子视图之后调用（dom操作放在这个里面）');
}
ngAfterViewChecked() {
    console.log('07ngAfterViewChecked执行了----每次做完组件视图和子视图的变更检测之后调用');
}
ngOnDestroy() {
    console.log('08ngOnDestroy执行了····');
}
```
Ionic4内置生命周期函数使用demo
```
export class Page01 implements OnInit {
  constructor() { 
  }
  ngOnInit() {
  }
  ionViewWillEnter(){
    console.log('ionViewWillEnter');
  }
  ionViewDidEnter(){
    console.log('ionViewDidEnter');
  }
}
```
注：本章节内容转载自网络：[http://www.ionic.wang/article-index-id-169.html](http://www.ionic.wang/article-index-id-169.html)
11. 地图

集成高德地图。

12. 修改应用图标
  1. 命令
```
ionic cordova resources
ionic cordova resources --icon
ionic cordova resources --splash
```
  2. 准备
* 准备一张大小为1024*1024的图片文件作为APP的图标（命名：icon）
* 准备一张大小为2732 * 2732(ionic2为2208 * 2208)的图片作为APP的启动界面（命名：splash）
* 两张图片的格式为png、psd或ai都可
  3. 修改方法

1.将准备好的两张图片（注意分辨率和图片格式）替换到web-app目录下的resources文件夹下。

2.执行ionic cordova resources命令，或分别执行ionic cordova resources --icon和ionic cordova resources --splash。

  4. 说明

一般执行第一条命令（ionic cordova resources）即可，如失败（一般都是网络问题，有的是图片问题），网络问题：执行后第二、三条命令（ionic cordova resources --icon和ionic cordova resources --splash），图片问题：修改或更换图片。

如果在启动界面不显示“小圆圈”（进度圈），修改config.xml文件<preference name="ShowSplashScreenSpinner" value="false" />的value为false，为true为显示。如下图：

![图片](https://uploader.shimo.im/f/km1Id4yU07spcNeL.png!thumbnail)

  5. 其他

config.xml配置说明：

```
<preference name="webviewbounce" value="false" />
<preference name="UIWebViewBounce" value="false" />
<preference name="DisallowOverscroll" value="true" />
<preference name="android-minSdkVersion" value="16" />
<preference name="BackupWebStorage" value="none" />
<preference name="ShowSplashScreen" value="true" />
<preference name="AutoHideSplashScreen" value="false" />
<preference name="ShowSplashScreenSpinner" value="false" />
<preference name="SplashMaintainAspectRatio" value="true" />
<preference name="FadeSplashScreenDuration" value="300" />
<preference name="SplashShowOnlyFirstTime" value="false" />
<preference name="SplashScreen" value="screen" />
<preference name="SplashScreenDelay" value="1500" />
<preference name="Fullscreen" value="true" />
<preference name="Orientation" value="landscape" />
```
<preference name="android-minSdkVersion" value="16" />
 android最小sdk版本

 <preference name="BackupWebStorage" value="none" />
 网络备份存储

 <preference name="ShowSplashScreen" value="true" />
 显示启动界面

 <preference name="AutoHideSplashScreen" value="false" />
 自动隐藏启动界面

 <preference name="ShowSplashScreenSpinner" value="false" />
 显示启动界面的进度圈

 <preference name="SplashMaintainAspectRatio" value="true" />
 屏幕保持长宽比

 <preference name="FadeSplashScreenDuration" value="300" />
 启动界面消失延迟

 <preference name="SplashShowOnlyFirstTime" value="false" />
 启动界面只在第一次显示

 <preference name="SplashScreen" value="screen" />
 启动界面

 <preference name="SplashScreenDelay" value="1500" />
 启动界面延迟

 <preference name="Fullscreen" value="false" />
 全屏显示

 <preference name="Orientation" value="landscape" />
 屏幕方向。Orientation设置可以让你锁定应用程序屏幕方向以阻止屏幕自动翻转。可选的值有：default，landscape(横屏)，portrait(竖屏)。

