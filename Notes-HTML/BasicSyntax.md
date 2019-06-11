# HTML 标题

|  内容  |         语法         |
| :----: | :------------------: |
|  标题  |         <h1>         |
| 水平线 |         <hr>         |
|  注释  | <!-- 这是一个注释--> |
|  换行  |         <br>         |

# HTML段落

| 内容 |                语法                 |
| :--: | :---------------------------------: |
| 段落 | <p>这是一个段落，<br>想换一个行</p> |

# HTML文本格式化

| 内容 |         语法         |
| :--: | :------------------: |
| 粗体 |  ```<b>bold</b>```   |
| 斜体 | ``` <i>italic</i>``` |

# HTML链接

~~~html
<!---target属性，定义链接文档在何处显示 -->
<a href="http://www.baidu.com" target="_blank">链接文本</a>
<!---id属性： -->
<p>
<a href="http://www.runoob.com/html/html-links.html#tips">
访问有用的提示部分</a>
<a href="#C4">查看章节 4</a>
</p>
<h2>章节 1</h2>
<p>这边显示该章节的内容……</p>

<h2>章节 2</h2>
<p>这边显示该章节的内容……</p>

<h2>章节 3</h2>
<p>这边显示该章节的内容……</p>

<h2><a id="C4">章节 4</a></h2>
<p>这边显示该章节的内容……</p>
~~~

# HTML头部

## head元素

包含了所有的头部标签元素。在 <head>元素中你可以插入脚本（scripts）, 样式文件（CSS），及各种meta信息。

可以添加在头部区域的元素标签为:
~~~html
<title>, <style>, <meta>, <link>, <script>, <noscript>, and <base>.
~~~

|   标签   |             描述             |
| :------: | :--------------------------: |
|  <head>  |          文档的信息          |
| <title>  |          文档的标题          |
|  <base>  |  页面链接标签的默认链接地址  |
|  <link>  | 一个文档和外部资源之间的关系 |
|  <meta>  |      HTML文档中的元数据      |
| <script> |       客户端的脚本文件       |
| <style>  |      HTML文档的样式文件      |

# HTML 样式-CSS

_CSS (Cascading Style Sheets)_ 用于渲染HTML元素标签的样式

## 内联样式

~~~html
<p style="color:blue;margin-left:20px;">
    this is a paragraph.
</p>
~~~

## 外部引用

~~~html
<head>
    <link rel="stylesheet" type="text/css" href="mystyle.css">
</head>
~~~

## 内部样式表

在HTML文档头部 <head> 区域使用<style> 元素 来包含CSS

~~~html
<head>
    <style type="text/css">
        body {background-color:yellow;}
        p {color:blue;}
    </style>
</head>
~~~

## 样式实例

### 背景颜色

~~~html
<body style="background-color:yellow;">
    <h2 style="background-color:red;">这是一个标题</h2>
    <p style="background-color:green;">这是一个段落。</p>
</body>
~~~

### 字体样式、颜色、大小

```html
<h1 style="font-family:verdana;">一个标题</h1>
<p style="font-family:arial;color:red;font-size:20px;">一个段落。</p>
```

### 文本对齐方式

```html
<h1 style="text-align:center;">居中对齐的标题</h1>
<p>这是一个段落。</p>
```

# HTML图像

~~~html
<img src="url" alt="some_txt"  width="304" height="228">
~~~

# HTML表格

表格由 <table> 标签来定义。每个表格均有若干行（由 <tr> 标签定义），每行被分割为若干单元格（由 <td> 标签定义）。字母 td 指表格数据（table data），即数据单元格的内容。数据单元格可以包含文本、图片、列表、段落、表单、水平线、表格等等。

~~~html

<table border="1">
  <caption>Monthly savings</caption>
  <tr>
    <th>Month</th>
    <th>Savings</th>
  </tr>
  <tr>
    <td>January</td>
    <td>$100</td>
  </tr>
  <tr>
    <td>February</td>
    <td>$50</td>
  </tr>
</table>


~~~

|    描述    |    标签    |
| :--------: | :--------: |
|    表格    |  *table*   |
| 表格的表头 |    *th*    |
|  表格的行  |    *tr*    |
|  表格单元  |    *td*    |
| 表格列的组 | *colgroup* |

# HTML列表

