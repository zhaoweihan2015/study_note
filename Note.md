# 问题总结
### Q:如何预览图片
#### A:使用 `createObjectURL` (注意兼容性)

### Q:如果多图片多input上传
#### A:所谓的多图片上传不是单纯的使用`multiple`进行设置，而是多个input标签以同一字段进行上传，而使用同一name会被最后一个空值input覆盖，而且input并不存在name[]的使用方法（？需要查查）,所以需要用js进行数组填充拼接为`FilesList`后通过`FormData`进行上传
#### 如果后台必须需要这个字段有值，因为js的数据类型并不存在file(文件)类型，所以需要另外一个值进行判断是否需要接收

### Q:img标签能否像background属性一样拥有cover这种全显功能
#### A:有，使用 `object-fit` 属性，`object-fit:cover`
#### 虽然很好用很有效但是ie浏览器全线不支持，ie手机浏览器也不支持，据说安卓版微信也不兼容（也许是老版本）

### Q:vue-cli 使用 less 问题，安装后报 Cannot find module 'less' 的错误
#### A:vue-cli本身可以使用`less-loader`，安装后重新运行即可。如果报错Cannot find module 'less'的错误，这个锅似乎要cnpm背。是因为cnpm在安装时少了一个`less`的入口文件，需要单独安装
#### `npm i --save-dev less` 安装即可
