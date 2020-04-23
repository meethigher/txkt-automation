---
title: 腾讯课堂以及各种文库下载实现自动化
date: 2020-04-17 18:52:04
tags: javascript

---

今天上线代课，老师抽查了一次签到，我没签上，应该按旷课处理了。

很恼火，这老师自己不好好教，天天念ppt，我干嘛还花时间听他的课？

ok，那就自动化！

<!--more-->

本着开源分享的精神，我把代码发到了b站上和博客上。

这个解决问题的想法并非我原创，网上有好多教程，比我早。而且这个一开始也是跟我自己的思路有点出入的。只能算是我对其的一个扩展。供大家学习使用。

# 腾讯课堂

[腾讯课堂实现自动化](https://b23.tv/BV1Lz41187RR
)

打开`chrome`浏览器（其他的也可以，chrome最好啦）。

打开`开发者工具`，直接将代码复制到`console`栏，按`enter`键即可。如果还不会用，就点击上面的那个链接。

## 1 开启自动任务

### 自动送花

开启一个`3秒`送花的定时器：

```javascript
let flower=setInterval(function (){
    document.getElementsByClassName("toolbar-icon")[2].click();
    console.log("送花");
},3000);
```

### 自动签到

开启一个每隔`10秒`检测一次是否有签到按钮的定时器，有的话就点击：

```javascript
let btns;
let attend=setInterval(function (){
    btns=document.getElementsByClassName("s-btn s-btn--primary s-btn--m");
    if(btns.length>0){
        console.log(new Date().toLocaleTimeString()+"--完成-->"+btns[0].innerText);
        btns[0].click();
    }
},10000);
```

### 自动刷屏

每隔`3秒`发送`886`

```javascript
let say = setInterval(function() {
    document.getElementsByClassName("ql-editor")[0].firstElementChild.innerText = "886";
}, 3000);
let say2 = setInterval(function() {
    document.getElementsByClassName("text-editor-btn")[0].click();
}, 3000)
```

### 下课自动发886

设置好下课的时间，比如我的线代下课时间是`16:50`，自动发送`886`。

这个执行了之后会自动关闭，如果在执行之前想关闭，请跳转到[这里关闭下课提示](#关闭下课提示)

```javascript
let targetHour = "16";
let targetMin = "50";
let date;
let inputed = false;
let timing = setInterval(function() {
    date = new Date();
    if (date.getHours() == parseInt(targetHour) && date.getMinutes() == parseInt(targetMin)) {
        if (!inputed) {
            document.getElementsByClassName("ql-editor")[0].firstElementChild.innerText = "886"
            inputed = true;
        } else {
            document.getElementsByClassName("text-editor-btn")[0].click();
            window.clearInterval(timing);
            console.log("下课咯！");
        }
    }
}, 1000);
```

### 显示/隐藏对话框

有小伙伴要这个功能，然后就加了一下

```javascript
let $switch=document.createElement("span");
let $body=document.querySelector(".web");
let show=true;
$switch.innerText="显示/隐藏";
$switch.style="position:absolute;top:20px;right:520px;color:#cccccc;border-radius:10px;cursor:pointer;z-index:9999999";
$switch.onclick=function (){
	if(show){
		document.querySelector(".study-body.mr").style="right:0;z-index:999";
		show=false;
	}else{
		document.querySelector(".study-body.mr").style="right:300px;z-index:0";
		show=true;
	}
}
$body.appendChild($switch);
```

效果如图：

{% asset_img 4.png 显示/隐藏 %}

## 2 关闭自动任务

### 关闭送花

```javascript
if(flower){
    window.clearInterval(flower);
    console.log("已关闭送花");
}
```

### 关闭签到

```javascript
if(attend){
    window.clearInterval(attend);
    console.log("已关闭签到");
}
```

### 关闭刷屏

```javascript
if(say){
    window.clearInterval(say);
    window.clearInterval(say2);
    console.log("已关闭刷屏");
}
```

### 关闭下课提示

```javascript
if(timing){
    window.clearInterval(timing);
    console.log("提前关闭下课提示");
}
```

## 3 今天上课的小插曲

一开始刚测试送花的时候，我开的是1秒送一个，估计是太活跃了，直接让马原老师，叫起来提问问题了。

我一脸懵逼啊，还好有大佬江湖救急。

{% asset_img 1.png 插曲 %}

还有一个小插曲，就是马原老师问了一个问题，“越南是什么党派？”。

然后，下面一堆回复共产党。老师说，他直接被腾讯警告了。

哈哈，太难了也！

# 文库下载

[超简单！学生如何免费下载文档，一看就会](https://www.bilibili.com/video/BV1gV411Z7YS/)

刚才在做英语练习题的时候，做完了，想对一下答案，就百度了搜了一下，进去一个文库。

下载文档要12块钱，我就看了一下，上面预览的都是图片，所以就想着，js批量把图片下载下来，然后合成pdf，就正常看了，关键是没花钱啊，要不用浪费多长时间。

具体的使用我放到youtube的[文库免费下载小技巧](https://youtu.be/UXI-_SBcw7o)，本来想往b站传的，但是被驳回锁定了。下面放上代码吧。

{% asset_img 3.png 只不过是程序代替了手动 %}

## 1 示例淘豆网

淘豆网就是直接把图片放出来了，咱们直接下手就行。

```javascript
function download(url, fileName) {
    let xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);//true表示异步
    xhr.responseType = 'blob';
    xhr.onload = () => {
        if (xhr.status === 200) {
           downloadByA(xhr.response,fileName);
        }
    };
    xhr.send();
}
function downloadByA(data,fileName){
	let urlObject = window.URL || window.webkitURL || window;
	let export_blob=new Blob([data]);
	let a=document.createElement("a");
	a.href=urlObject.createObjectURL(export_blob);
	a.download=fileName;
	a.click();
}
//下面这块代码需要按自己需求，进行稍微地修改，上面两块代码可以不用动
document.querySelectorAll(".pageBox img").forEach(function(ele, i) {
    download(ele.src,i+".jpg");
});
```

然后将下载出的图片合成pdf，就ok了

点击进入[淘豆网示例网址](https://www.taodocs.com/p-134597690.html)

## 2 示例道客巴巴

道客巴巴就有点小心眼了，他把所有的预览图片，都转成了canvas形式的，那也没事，咱们同样用代码给他转回来。

```javascript
function download(url, fileName) {
    let xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);//true表示异步
    xhr.responseType = 'blob';
    xhr.onload = () => {
        if (xhr.status === 200) {
           downloadByA(xhr.response,fileName);
        }
    };
    xhr.send();
}
function downloadByA(data,fileName){
	let urlObject = window.URL || window.webkitURL || window;
	let export_blob=new Blob([data]);
	let a=document.createElement("a");
	a.href=urlObject.createObjectURL(export_blob);
	a.download=fileName;
	a.click();
}
//下面这块代码需要按自己需求，进行稍微地修改，上面两块代码可以不用动
document.querySelectorAll(".outer_page .inner_page").forEach(function(ele, i) {
    download(ele.toDataURL("image/jpeg"),i+".jpg");
});
```

点击进入[道客巴巴示例网址](https://www.doc88.com/p-7337496651450.html)

## 3 示例百度文库

百度文库有文字显示的，也有图片显示的，这里主要针对图片显示的，文字的我也看了太麻烦了。

但是在处理图片的过程中也出了问题，涉及到跨域的问题

{% asset_img 2.png 百度文库 %}

```javascript
document.querySelectorAll(".mod.reader-page.complex.hidden-doc-banner .inner .bd .reader-pic-item").forEach(function(ele, i) {
    download(ele.style.backgroundImage.match(/[^url("].*[^")]/)[0],i+".jpg");
});
```

点击进入[百度文库示例网址](https://wenku.baidu.com/view/3dce4d3e02d8ce2f0066f5335a8102d276a261d2)，网址放到这里，留着以后有时间再研究。准备上数学课了。

## 4 关于合成pdf

现在工具很多，有免费在线版，也有免费的客户端，给大家推荐两款吧。

1. [免费在线转pdf](https://smallpdf.com/)
2. [adobe acrobat](https://acrobat.adobe.com/cn/zh-Hans/acrobat.html)

我是用的acrobat这个软件，这个比较好用，还可以去水印之类的。在我之前去水印的文章里，有提到过。
