# DCloud踩坑实录
因为公司要使用mui进行混开，这个曾经没踏开的坑也要重新踏了，所以在此会记录自己遇到的坑和解决方法
### 1.预加载的webview无法打开
#### 在经过preLoad预加载的webview直接使用show方法会直接报错（undefined）。因为预加载的webview模板也是需要时间进行加载的，如果直接对变量进行操作，变量的值仍为undefined，所以在使用show时需要使用setTimeout进行加载时间模拟
```javascript
	var ws = mui.preLoad({
		url:'list.html',
		id:'list'
	})
	
	setTimeout(function(){
		ws.show()
	},150)
```
### 2.手机后退键和后退事件会错误的关闭webview
#### 需要手动重写mui的back事件
```javascript
	mui.back = function() {
		plus.webview.close(plus.webview.getTopWebview(), 'auto', 300)
	}
```
### 3.在已经加载的webview在此使用show()不会有动画效果
#### 需要先将webview使用hide方法隐藏或者使用close方法销毁，再次使用show后才会有动画效果
#### 所以需要在动画结束之后对其他的webview进行hide
```javascript
	ws.addEventListener('show', function() {
		var hideWebViewList = ['index', 'HBuilder', viewName]
		plus.webview.all().forEach(function(e) {
			if(hideWebViewList.indexOf(e.id) == -1) {
				e.hide()
			}
		})
	})
```
#### PS:对webview的show方法进行监听，会在show动画结束后触发（查了好久文档，当初还以为是onAniamtionFinshed）
### 4.webview切换时动画卡顿
#### webview在使用openWindow进行切换时，动画和创建渲染是同步进行的，所以会有卡顿的问题。
#### 解决方法就是监听webview在状态为rendered时进行show动画
#### loading->loaded->rendering->rendered
```javascript
	ws.addEventListener('rendered', function() {
		ws.show('slide-in-right', 300)
	})
```
#### PS:个人在使用时效果还是不错的，但是无法达到原生那种顺滑，再努力优化优化看看。
### 5.plus检测
#### 找到的官方方法
```javascript
if(!navigator.userAgent.match(/Html5Plus/i)) {
	//非5+引擎环境，直接return;					
	return;									
}												
```