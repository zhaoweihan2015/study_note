# 问题总结 
## Q:如何预览图片
#### A:使用 `createObjectURL` (注意兼容性) 
#### PS:最新问题，chorme在2018年7月取消`createObjectURL`对流的转换，所有媒体流改用`HTMLMediaElement.srcObject`解决
```javascript
	video.src = URL.createObjectURL(stream)
	// 改用此 (stream)为之前getUserMedia获取的媒体流
	video.srcObject = stream
```
 
## Q:如果多图片多input上传
#### A:所谓的多图片上传不是单纯的使用`multiple`进行设置，而是多个input标签以同一字段进行上传，而使用同一name会被最后一个空值input覆盖，而且input并不存在name[]的使用方法（？需要查查）,所以需要用js进行数组填充拼接为`FilesList`后通过`FormData`进行上传
#### 如果后台必须需要这个字段有值，因为js的数据类型并不存在file(文件)类型，所以需要另外一个值进行判断是否需要接收

## Q:img标签能否像background属性一样拥有cover这种全显功能
#### A:有，使用 `object-fit` 属性 
```css  
object-fit:cover
```
#### 虽然很好用很有效但是ie浏览器全线不支持，ie手机浏览器也不支持，据说安卓版微信也不兼容（也许是老版本）

## Q:vue-cli 使用 less 问题，安装后报 Cannot find module 'less' 的错误
#### A:vue-cli本身可以使用`less-loader`，安装后重新运行即可。如果报错Cannot find module 'less'的错误，这个锅似乎要cnpm背。是因为cnpm在安装时少了一个`less`的入口文件，需要单独安装
#### `npm i --save-dev less` 安装即可

## Q:如何在不同的行中使不定数量的图片显示适当
#### A:使用css3新伪类`last-nth-child(n)`搭配兄弟选择器`~`使用
```css
li img:last-nth-child(2),img:last-nth-child(2)~img{ 
  #code 
}
li img:last-nth-child(3),img:last-nth-child(3)~img{ 
  #code 
}
```

## Q:验证对象中某属性是否存在
#### A:别再验证它是否为undefined了，改用`hasOwnProprety`
#### PS:然而在jquery源码中使用的是===进行判断
```javascript
if(obj.a === null ||　obj.a === undefined){
  // 判断
}
```
## Q:背景图在加载完成后显示
#### A:很多时候为了`cover`显示放弃img改用背景属性（虽然可以用`object-fit`，但是IE不兼容）,但加载时会出现渐变闪烁逐层出现的问题,可以采取比较曲线的方法
#### 我的做法是先生成一个Image对象，在Image加载成功后添加背景，这样读取的是缓存中的图片，加载就会快速显示
```javascript
var src = 'bg.png',
    img = new Image()
img.onload = function(){
  el.style.backgroundImage = src 
}
img.src = src // 一定要在onload之后设置，不然IE中会出现错误
img = null // 清空对象
```
#### PS:canvas的图片填充同理，如果需要图片加载填充完成后进行操作，也需要用到`onload`（不过小心回调地狱）
#### PS的PS:如果图片过大（测试时为4000X5000的照片，手机高清拍摄）在加载时候还是会逐层加载，但是也会比直接添加属性快一些

## Q:IOS手机在上传照片方向不对的问题
#### A:首先安卓手机拍摄后的照片本身会被处理一遍，按照拍摄方向进行转向，IOS虽然不会处理但是会在图片的原始数据包含照片信息（包括地理位置，手机型号，拍照方向，曝光度，光圈选择等问题），除了IOS，部分单反拍摄的照片的原始数据也会包含信息
#### 处理方法为使用`exif.js`检测图片的元数据，然后根据`Orientation`属性判断照片方向
```javascript
EXIF.getData(IMG, function(){
  EXIF.getTag(this, 'Orientation');
});
``` 

## Q:pc端浏览器，安卓手机，IOS手机在input files表现的差异（目前发现两点）
#### A:作为一个前端，window电脑，Mac，安卓手机，IOS手机各配一个还是有些必要，比如这里我完全不知道IOS下还会有exif问题...
#### 1.IOS手机拍摄的照片同单反一样，附带exif原始数据（包含拍摄时设备方向，地理位置，设备型号等），但是拍摄后照片是相对于设备方向的。安卓手机拍摄的照片不附带exif原始数据，但是拍摄后的照片会经过系统处理变为相对于地理位置（重力）方向的图片。而经过处理的照片（ps等工具）会丢失exif原始数据，以处理后的方向为准。
####   所以在这里出现的问题就是ios和单反拍摄的竖向照片（非机器正方向照片），在web读取的时候全部“倒下”（以原设备正方向），处理方法见上一个问题
#### 2.PC浏览器和安卓手机在选择一次图片后第二次选择时选择取消也会触发change事件，并且input标签的files属性会被清空（取消所有选择），而IOS会保持上一次的状态（取消这次选择），且不会触发change事件

## Q:图片上传检测问题
#### A:对于一个图片上传总会有人上传奇奇怪怪的东西。哪怕没有人上传测试会去传各种奇怪的东西去测试
#### 1.上传内容不为图片
#### 解决方法是使用`this.files[0].name.split('.')[1]`取得后缀名来进行后缀名判断（对于一个文件名字多.的，请使用length-1解决）
```javascript
 var filesName = this.files[0].name.split('.'),
     filesTypeArr = ['jpg', 'png', 'jpeg', 'JPG', 'PNG', 'JPEG']
      filesName = filesName[filesName.length - 1]
      if (filesTypeArr.indexOf(filesName) == -1) {
            // 不是图片格式
      } 
```
#### 2.上传内容为损坏图片或者其他文件强行改为图片后缀 
#### 一般估计还真有人会这么干（比如我），因为原项目中有图片预览，取得的图片地址因为无法进行读取所以无法进行后续处理，解决方法为使用img.onerror
```javascript
  var img = new Image()
  img.src = imgSrc
  img.onload = function () {
    // 图片加载成功之后
  }
  img.onerror = function () {
    // 图片加载失败之后
  }
```
#### PS:`img.onerror`本身有兼容性问题，在火狐中会无法使用，原因比较简单，就是火狐把src的属性设为空不视作error错误

## Q:手机端后退Input标签预填充
#### A:在PC，安卓，IOS上都各有差异，pc安卓在后退后会带缓存进行“假”刷新，而ios是恢复到条状前的状态，差距还是不小的
#### 如果希望后退后input中不带数据，请使用`form.reset()`进行清除数据，并在input标签上设置`autocomplete="off"`

## Q:如何让伪数组使用数组方法
#### A:这个解决方法还是在刷阿里前端面试题的时候发现的。如果我希望循环一个伪数组（比如dom节点集合或者arguments），但是我们无法使用forEach,因为forEach是内置对象Array的方法，而伪数组并不是Array，只是一个长得比较像Array但是没有Array方法的Object（虽然Array也是Object上附带的内置方法和属性）。但是我们依旧可以使用其他方法使伪数组使用数组方法
```javascript
function check (){
  var res = []
  for (var i = 0 ; i < arguments.length ; i ++) {
    if(arguments[i] > 2){
      res.push(arguments[i])
    }
  }
  return res
}

function check () {
  return Array.prototype.forEach.call(arguments,function(e){
    return e > 2
  })
}
```
#### 伪数组具有数字的“型”，但是没有数组的方法，通过call()和prototype将数组的特有方法绑定上下文为伪数组，可以使伪数组使用数组方法。这种方法比for循环更加优雅且高效。

## Q:IOS手机无法自动播放音乐
#### IOS手机通过某些设置禁止了web页面audio的自动播放，只能通过事件触发
```javascript 
  function autoplay2IOS () {
    var music = document.getElementsByTagName('audio')[0],
        play = function () {
          music.play()
        }
    // 安卓手机和pc端电脑（直接播放）
    play()
    // ios微信端（需要二次开发）
    document.addEventListener("WeixinJSBridgeReady",function(){
      play()
    })
    // ios safari （增加手势事件，在事件触发时播放）
    document.addEventListener("touchstart",function(){
      play()
    })
 }
```
#### 对于在手机端使用自动播放还是放弃吧，IOS这问题它从根上挡住了，压根没法解决
#### ps:然而我在测试的时候IOS手机哪怕是通过事件触发play()也无法播放声音，也许是IOS的微信缓存太大了

## Q:vue解决文字闪烁问题
#### 在文字未加载之前会先显示{{}}再显示内容，会有闪烁问题。采用v-cloak属性可以解决
#### HTML
```html
<div id="app" v-cloak>
	<!-- code -->
</div>
```
#### CSS
```css
[v-cloak]{
	display:none;
}
```
#### PS:v-cloak不用每一条都加，在总标签上添加既即可
#### PS的PS:v-cloak需要和display:none配合使用，否则无效

## Q:webpack报错`Cannot find module 'webpack-cli/bin/config-yargs'`
#### 原因是`webpack`与`webpack-dev-server`版本不对应，如果直接选择最高版本，两个模块内版本是不对应的，所以要退级使用
#### 选择使用`webpack@2.6.1`与`webpack-dev-server@2.11.2`(查了mpvue-cli中使用的是这两个版本)
```cmd
npm install webpack@2.6.1 --save-dev
npm install webpack-dev-server@2.11.2 --save-dev
```
## Q:webpack报错`chunk.sortModules is not a function`
#### 原因是`extract-test-webpack-plugin`版本不对，采用2.0.0版本
```cmd
npm install extract-test-webpack-plugin@2.0.0 --save-dev
```
### Q:在IOS中Date相关方法（例如：getHours()）返回NaN
#### 在正常使用new Date()是正常的，而使用字符串传参时候，比如new Date('2018-05-19 12:00:00')这种形式，在IOS下调用getHours()等方法返回NaN
```javascript
	// IOS
new Date('2018-05-19 12:00:00').getHours() // NaN
```
#### 解决方法是改用 2018/05/09 12:00:00 这种形式
```javascript
var now - new Date('2018/05/19 12:00:00')
```
#### 如果所使用的时间字符串由其他来源获取，可以采用正则表达式和String的replace方法进行替换
```javascript
new Date("2018-05-19".replace(/-/g,'/'))
```