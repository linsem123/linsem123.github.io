---
title: 你不知道的js
date: 2019-10-24 10:13:10
categories:
- 编程
tags:
- js
---

# 《你不知道的js》
### 1. window.location.hash
[window.location.hash 使用](https://www.cnblogs.com/canger/p/7595641.html)
```
http://www.example.com/index.html#print
```
就代表网页index.html的print位置。浏览器读取这个URL后，会自动将print位置滚动至可视区域。
单单改变#后的部分，浏览器只会滚动到相应位置，不会重新加载网页。
改变#会改变浏览器的访问历史.
每一次改变#后的部分，都会在浏览器的访问历史中增加一个记录，使用"后退"按钮，就可以回到上一个位置。
**onhashchange事件**: hash发生变化时触发
```
window.onhashchange = func;
```

### 2. npm i -save -save-dev 区别
- npm i XXX: 安装项目到项目目录下，不会将模块依赖写入到devDependencies或dependencies
- npm i -g XXX: -g 的意思是将模块安装的全局，具体安装到磁盘的哪一个位置，依赖于npm config prefix的位置
- npm i -save XXX: -save 的意思是将模块安装到项目目录下面，并在package文件的dependencies节点写入依赖
- npm i -save-dev XXX: -save-dev 的意思是将模块安装的项目目录下，并在package文件的devDependencies节点写入依赖。

### 3. 获取根结点字体大小
``` 
getComputedStyle(window.document.documentElement)['font-size']
Number(getComputedStyle(window.document.documentElement)['font-size'].replace(/px/gi, '')) 
```

### 4. javascript正则表达式/g与/i及/gi的意义
例如
```
regularexpression=/pattern/[switch] // 这个switch就有三种值
```
- g: 全局匹配 
- i: 忽略大小写 
- gi: 全局匹配 + 忽略大小写


