# Javascript
[TOC]

## js 
```html
<!DOCTYPE html>
<html>
<head>
    <title>javascript</title>
    <!-- 外部调用 -->
    <script type="text/javascript" src="ext.js"></script>
    <!-- 内部调用  -->
    <script type="text/javascript">
        alert("Hello world");
    </script>
</head>
<body>
  <h1>H1</h1>
</body>
</html>
```

## js 
```html

```

## with
```js
with(document) {
    bgColor = "blue";
    fgColor = "#ff0000";
    linkColor = "yellow"
    alinkColv = "black";
    vlinkColor = "white";
}

with(document) {
    writeln("<h1>This is Head 1</h1>");
    writeln("<p>frontground</p>" + fgColor);
    writeln("<p>background</p>" + bgColor);
    writeln("<a href='www.baidu.com'>baidu</a>");
    writeln("<p>Last modified</p>" + lastModified);
    writeln("<p>URL</p>" + URL);
    writeln("<p>location</p>" + location);
}
```



## Document 对象
```js
// prompt 输入框
var name = prompt("Input your name:");

// 浏览器打开新窗口
win = window.open("newpage.html");
if(null != win && !win.closed) {
    // document.open([MIMEType])
    // MIMEType 有 text/html, text/plain, image/jpeg等等
    win.document.open("text/plain");
    win.document.writeln("hello," + name +"!");
    win.document.write("welcome!");
    win.document.close();
}
```


## 函数
```js
function a(x, y) {
    return x + y;
}

var b = function(x, y) {
    alert(arguments.length);
    return x + y;
}

var c = Function("x", "y", "return x + y");

document.writeln(a(1, 2) + "<br>");
document.writeln(b(3, 4) + "<br>");
document.writeln(c(5, 6) + "<br>");
```

> 有3种函数的定义


## 类
```js
function member(name, gender) {
    this.name = name;
    this.gender = gender;
    this.display = display;
}

function display() {
    var str = this.name + ":" + this.gender;
    document.writeln(str + "</br>");
}

function showProperty(obj, objString) {
    var str ='';
    for(var i in obj) {
        str += objString + "." + i + "=" + obj[i] + "<br>";
    }
    return str;
}

var obj = new member("Tom", "male");
obj.display();
document.writeln(showProperty(obj, "person"));
```










