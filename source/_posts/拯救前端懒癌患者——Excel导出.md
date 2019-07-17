---
title: 拯救前端懒癌患者——Excel导出
date: 2019-07-13 20:10:07
urlname: frontend-excel-export
categories: ["技术"]
tags: ["JS"]
toc: true
---

### 前言

几乎所有的前端只要是做过后台管理系统的都会碰到一个需求，那就是导出`Excel`。

导出表格或许特别简单也或许特别复杂，如果后端提供了`Excel`对应的服务器下载地址那对于前端来讲再好不过了。但若需要前端自己对数据进行处理并且实现导出文件看起来就有点棘手。为了解决前端导出`Excel`的问题，一些库应运而生，不过很多都需要对表格数据进行组装，光是看那一长串的`API`就头痛，那有没有一个库用法既简单又功能强大呢？必须有，我们今天就来介绍这样一款`Excel`库。

### [js-xlsx](https://github.com/SheetJS/js-xlsx)

`js-xlsx`是`SheetJS`推出的一个用于处理`Excel`相关问题的前端工具库，支持几乎所有框架和平台且非常好上手。

![js-xlsx](https://s2.ax1x.com/2019/07/14/ZIByW9.png)

#### 安装

`js-xlsx`支持直接引入资源文件以及`npm`、`bower`安装:

``` javascript
// 可以从官方github上下载
<script lang="javascript" src="dist/xlsx.full.min.js"></script>
// 也可以引入CDN地址
<script src="https://cdn.bootcss.com/xlsx/0.14.3/xlsx.full.min.js"></script>
```

npm:

``` shell
$ npm install xlsx
```

bower:

``` shell
$ bower install js-xlsx
```

#### 使用

这里以`Angular`为例:

第一步准备好数据:

``` javascript
$scope.data = [
  { Name: "Bill Clinton", Index: 42 },
  { Name: "GeorgeW Bush", Index: 43 },
  { Name: "Barack Obama", Index: 44 },
  { Name: "Donald Trump", Index: 45 }
];
```

第二步将数据渲染到`table`上:

``` html
<table id="sjs-table">
  <tr><th>Name</th><th>Index</th></tr>
  <tr ng-repeat="row in data">
    <td>{{row.Name}}</td>
    <td>{{row.Index}}</td>
  </tr>
</table>
```

最后，导出:

```javascript
var wb = XLSX.utils.table_to_book(document.getElementById('sjs-table'));
XLSX.writeFile(wb, "export.xlsx");
```

简简单单三步，大功告成。其实这里`js-xlsx`就是帮助我们将页面上的表格`dom`直接处理成Excel文件，不过当然生成`dom`并不是唯一的用法，你也可以将数据直接注入到表格中生成文件:

``` javascript
var data = [
  { name: "Barack Obama", pres: 44 },
  { name: "Donald Trump", pres: 45 }
];
var ws = XLSX.utils.json_to_sheet(data);
var wb = XLSX.utils.book_new();
XLSX.utils.book_append_sheet(wb, ws, "Presidents");
XLSX.writeFile(wb, "sheetjs.xlsx");
```

#### 多表格

`js-xlsx`还支持将多张表格一次性导出到同一个文件中:

``` javascript
var workbook = XLSX.utils.book_new();
var ws1 = XLSX.utils.table_to_sheet(document.getElementById('table1'));
XLSX.utils.book_append_sheet(workbook, ws1, "Sheet1");
var ws2 = XLSX.utils.table_to_sheet(document.getElementById('table2'));
XLSX.utils.book_append_sheet(workbook, ws2, "Sheet2");
```

#### 控制百分比显示

表格中如果有诸如`41%`这样的百分比数据在导出的`Excel`中会显示成`0.41`，只需要一个配置就可以避免这样的自动转换:

``` javascript
var wb = XLSX.utils.table_to_book(document.getElementById('sjs-table'), {raw: true});
```

`raw`这个参数用来配置是否保持原有字符串。

### 结语

今天只是对`jx-xlsx`做了一个最常用功能的介绍，其他框架的用法`GitHub`上也都有详尽介绍。更多丰富的功能我也还在发掘中。