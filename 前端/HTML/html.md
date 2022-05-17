# 常用标签

#### link

常用来链接外部样式表文件

```html
<link href="style.css" ref="stylesheet">
```

#### style

业内样式表定义，放在<head>标签中

```html
<style type="text/css">
	.font-red {
		color: red;
	}
</style>
```

#### 链接

```html
<a href="http://www.magedu.com" target="_blank" onclick="alert('anchor')">这是链接</a>
```

href属性，指定链接或锚点

- href="#test"，这是定义页内的锚点
- href="http://www.magedu.com"，指定域名的超链接
- href='test'，使用当前URL的资源路径，拼接上test
- href="/test"，使用当前URL的站点根路径，拼接test

target属性，指定是否在本窗口中打开

- _blank 就是在新窗口打开链接
- _self 当前页面打开链接

onclick是点击事件，等号后面通常写一个js函数或脚本执行。常见事件属性有onclick、ondblclick、onkeydown、onkeyup、onkeypress、onmousedown、onmousemove、onmouseover、onsubmit、onchange等。后面写的函数称作事件相应函数或事件回调函数

默认情况下，超链接点击后会发起一个HTTP GET请求

#### 标题

\<h1>~\<h6>，标题标签，默认h1字体最大，h6字体最小

#### 图片

```html
<img src="http://www.magedu.com/wpcontent/uploads/2018/12/2018122312035677.png" alt="magedu">
```

图片标签，src指定图片路径。注意，图片会发起一个HTTP GET请求；alt图片不能显示时，看到这属性的文本

#### 图层

```html
<div id="logo" class="logo"></div>
```

id属性，非常重要，标签的唯一标识

class属性，非常重要，样式表定位并附加样式

\<div>标签，目前改标签加上CSS，被广泛用于网页布局中

#### 列表

无序列表

```html
<ul>
    <li>coffee</li>
    <li>milk</li>
</ul>
```

有序列表

```html
<ol>
    <li>coffee</li>
    <li>milk</li>
</ol>
```

#### 表格

HTML表格的基本结构

- \<tabble>...\</table>：定义表格
- \<tr>...\</tr>：定义表格的行
- \<th>...\</th>：定义表格的标题列（文字加粗）
- \<td>...\</td>：定义表格的列
- colspan：合并单元格

th用的少，表格表头是否字体加粗，可以用CSS控制

```html
<table border="1">
    <tr>
        <th>1,1</th>
        <th>1,2</th>
    </tr>
    <tr>
        <td>2,1</td>
        <td>2,2</td>
    </tr>
    <tr>
        <td colspan="2">占2列</td>
    </tr>
</table>
```

#### 表单

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>magedu html</title>
    <script src="jquery.min.js"></script>
    <style>
    	td { padding: 5px}
    </style>
</head>
<body>
    <form action="" method="POST" enctype="application/x-www-formurlencoded">
        <table border="1" style="border-collapse: collapse">
            <tr>
                <td colspan="2">用户注册</td>
            </tr>
            <tr>
                <td>用户名</td>
                <td><input type="text" placeholder="用户名"></td>
            </tr>
            <tr>
                <td>密码</td>
                <td><input type="password" name="password" id="password"></td>
            </tr>
            <tr>
                <td>性别</td>
                <td>
                    <input type="radio" name="gender" id="gender" checked value="M">男
                    <input type="radio" name="gender" id="gender" value="F">女
                </td>
            </tr>
            <tr>
                <td>爱好</td>
                <td>
                    <input type="checkbox" name="interest" id="interest" value="music">音乐
                    <input type="checkbox" name="interest" id="interest" checked value="movie">电影
                </td>
            </tr>
            <tr>
                <td colspan="2">
                    <input type="submit" value="提交">
                    <input type="reset" value="重置">
                </td>
            </tr>
        </table>
    </form>
</body>
</html>
```

**特别注意：**表单控件如果要提交数据，必须使用name属性，否则不能提交到服务端

form标签的重要属性

- action：表单数据submit提交到哪里
- method：提交的方法，常用POST
- enctype：对提交的数据编码
  - application/x-www-form-urlencoded，在发送前编码所有字符（默认）
    - 空格转换为 "+" 加号，特殊符号转换为 ASCII HEX 值
  - multipart/form-data，不对字符编码。在使用包含文件上传控件的表单时，必须使用该值
  - text//plain，空格转换为“+”加号，但不对特殊字符编码