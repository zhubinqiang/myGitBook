# jQuery
[TOC]

jQuery [CDN](http://www.bootcdn.cn/jquery/)
中文 jQuery [API](http://hemin.cn/jq/) 速查
3.2.1: https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js

## 引入 jQuery
```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
    <script type="text/javascript">
        $().ready(function(){
            // your code
        });

    </script>
</head>
```

## Dom 与 jQuery 相互转换
标准JavaScript处理
```js
var p = document.getElementById("tab");
p.innerHTML = "hello world";
p.style.color = "red";
```

jQuery 处理
```js
var $p = $("#tab");
$p.html("hello world").css("color", "red");
```


```html
<div>item1</div>
<div>item2</div>
<div>item3</div>
```

jQuery --> DOM 
```js
var $div = $("div");     // jQuery对象
var div = $div.get(0);   // 通过get方法，转化为dom对象
div.style.color = "red"; // 通过dom对象操作
```

DOM ---> jQuery
```js
var div = document.getElementsByTagName("div");  // dom
var $div = $(div);           // jQuery
var $first = $div.first();   // 找到第一个元素
$first.css("color", "red");  // 给第一个对象设置元素
```

## [选择器](https://www.w3cschool.cn/jquery/jquery-ref-selectors.html)
|语法|描述|
|:--------|:--------|
|`$("*")`|选取所有元素|
|`$(this)`|选取当前 HTML 元素|
|`$("p.intro")`|选取 class 为 intro 的 `<p>` 元素|
|`$("p:first")`|选取第一个 `<p>` 元素|
|`$("ul li:first")`|选取第一个 `<ul>` 元素的第一个 `<li>` 元素|
|`$("ul li:first-child")`|选取每个 `<ul>` 元素的第一个 `<li>` 元素|
|`$("[href]")`|选取带有 href 属性的元素|
|`$("a[target='_blank']")`|选取所有 target 属性值等于 "_blank" 的 `<a>` 元素|
|`$("a[target!='_blank']")`|选取所有 target 属性值不等于 "_blank" 的 `<a>` 元素|
|`$(":button")`|选取所有 type="button" 的 `<input>` 元素 和 `<button>` 元素|
|`$("tr:even")`|选取偶数位置的 `<tr>` 元素|
|`$("tr:odd")`|选取奇数位置的 `<tr>` 元素|

## 方法
### 获取
`text()`: 设置或返回所选元素的 **文本内容**
`html()`: 设置或返回所选元素的内容（包括 HTML 标记）
`val()`:  设置或返回 **表单字段的值**
`attr()`: 方法用于获取属性值

```html
<body>
<p id="test1">这是一个段落。</p>
<p id="test2">这是另外一个段落。</p>
<p>输入框: <input type="text" id="test3" value="W3Cschool教程"></p>
<button id="btn1">设置文本</button>
<button id="btn2">设置 HTML</button>
<button id="btn3">设置值</button>
<p><a href="http://www.w3cschool.cn" id="runoob">W3Cschool教程</a></p>
</body>
```

```js
$("#test1").text("Hello world!");
$("#test2").html("<b>Hello world!</b>");
$("#test3").val("Dolly Duck");


$("#runoob").attr("href","//www.w3cschool.cn/jquery");
$("#runoob").attr({
    "href" : "//www.w3cschool.cn/jquery",
    "title" : "jQuery 教程"
});
```

### 添加元素
`append()`: 在被选元素 **内部的结尾** 插入指定内容
`prepend()`: 在被选元素 **内部的开头** 插入指定内容

```html
<body>
    <p>这是一个段落。</p>
    <p>这是另外一个段落。</p>
    <ol>
        <li>List item 1</li>
        <li>List item 2</li>
        <li>List item 3</li>
    </ol>
    <button id="btn1">添加文本</button>
    <button id="btn2">添加列表项</button>
</body>
```

```js
$("p").append("Some appended text.");
$("p").prepend("Some prepended text.");
```

`after()`: 在被选元素之后插入内容。
`before()`: 在被选元素之前插入内容。

```js
var txt1="<b>I </b>";                    // 使用 HTML 创建元素  
var txt2=$("<i></i>").text("love ");     // 使用 jQuery 创建元素
var txt3=document.createElement("big");  // 使用 DOM 创建元素
txt3.innerHTML="jQuery!";
$("img").after(txt1,txt2,txt3);          // 在图片后添加文本
```

### 删除元素
`remove()`: 删除被选元素（及其子元素
`empty()`: 从被选元素中删除子元素 **就剩下它本身**

```html
<div id="div1" style="height:100px;width:300px;border:1px solid black;background-color:yellow;">

    这是 div 中的一些文本。
    <p>这是在 div 中的一个段落。</p>
    <p>这是在 div 中的另外一个段落。</p>

</div>
<br>
<button>移除div元素</button>
```

```js
$("#div1").remove();
$("#div1").empty();

$("p").remove(".italic"); // 删除 class="italic" 的所有 <p> 元素
```


### CSS
`addClass()`: 向被选元素添加一个或多个类
`removeClass()`: 从被选元素删除一个或多个类
`toggleClass()`: 对被选元素进行添加/删除类的切换操作
`css()`: 设置或返回样式属性

```html
<h1>标题 1</h1>
<h2>标题 2</h2>
<p>这是一个段落。</p>
<p>这是另外一个段落。</p>
<div>这是一些重要的文本!</div>
<br>
<button>为元素添加 class</button>
```

```css
.important{
    font-weight:bold;
    font-size:xx-large;
}
.blue{
    color:blue;
}
```

```js
$(document).ready(function(){
    $("button").click(function(){
        $("h1,h2,p").addClass("blue");
        $("div").addClass("important");

        $("p").css("background-color","yellow");
        $("p").css({"background-color":"yellow","font-size":"200%"});
    });
});
```

### Ajax
```js
$.get(URL,callback);
$.post(URL,data,callback);
```


## 判断元素是否存在
```js
var val = $("#abcd").length;
if(val>0) {
    console.log("Yes");
    return true;
}else {
    console.log("No");
    return false;
}
```

## 跳转页面
dom 操作
```js
// 利用http的重定向来跳转
window.location.replace("https://www.baidu.com/");

// 使用href来跳转
window.location.href = "https://www.baidu.com/";
```

jQuery 操作
```js
$(location).attr('href', 'http://www.jb51.net');
$(window).attr('location','http://www.jb51.net');
$(location).prop('href', 'http://www.jb51.net');
```






