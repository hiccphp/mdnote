# 如何使用DataTable实现后端分页

## 官网案例

JS代码

```js
$(document).ready(function() {
    $('#example').DataTable( {
        "processing": true, // 是否显示‘处理中....’的过渡提示
        "serverSide": true, // 是否开启后端分页
        "ajax": "../server_side/scripts/server_processing.php"
    } );
} );
```

Html代码

```html
<table id="example" class="display" style="width:100%">
        <thead>
            <tr>
                <th>First name</th>
                <th>Last name</th>
                <th>Position</th>
                <th>Office</th>
                <th>Start date</th>
                <th>Salary</th>
            </tr>
        </thead>
        <tfoot>
            <tr>
                <th>First name</th>
                <th>Last name</th>
                <th>Position</th>
                <th>Office</th>
                <th>Start date</th>
                <th>Salary</th>
            </tr>
        </tfoot>
    </table>
```

异步请求的数据格式

```json
{
  "draw": 1,
  "recordsTotal": 57,
  "recordsFiltered": 57,
  "data": [
    [
      "Airi",
      "Satou",
      "Accountant",
      "Tokyo",
      "28th Nov 08",
      "$162,700"
    ],
    ...
    [
      "Cedric",
      "Kelly",
      "Senior Javascript Developer",
      "Edinburgh",
      "29th Mar 12",
      "$433,060"
    ]
  ]
}
```



## 配置一览

| **名称**   | **类型** | **描述**                                                                                       |
| ---------- | -------- | ---------------------------------------------------------------------------------------------- |
| aaSorting  | array    | 默认第几个排序                                                                                 |
| bStateSave | boolean  | 是否保存状态                                                                                   |
| serverSide | boolean  | 是否开启后端分页（默认关闭）                                                                   |
| processing | boolean  | 是否显示处理状态(排序的时候，数据很多耗费时间长的话，也会显示这个)                             |
| paging     | boolean  | 表格允许分页                                                                                   |
| ajax       | mixed    | 数据来源（包括处理分页，排序，过滤） ，即url，action，接口，等等。可以通过配置异步请求调整参数 |
| scroller   | object   | 滚动显示，动态加载的功能。                                                                     |
| columnDefs | object   | 定义列。                                                                                       |
| columns    | object   | 定义后端分页后，用此项来配置每列的数据显示。                                                   |
| scroller   | object   | 滚动显示，动态加载的功能。                                                                     |


## 异步回调
```js
"ajax": function(data, callback, settings) {
    var pagesize = data.length;//页面显示记录条数，在页面显示每页显示多少项的时候,页大小
    var page = (data.start) / data.length+1; // 获取分页条数
    $.ajax({
        type:'GET',
        url: '/account/list',
        data: {page:page,pagesize:pagesize},
        dataType:'json',
        success: function(data) {
            var arr = "";
            if ('object' == typeof data) {
                arr = data;
            } else {
                arr = $.parseJSON(data);//将json字符串转化为了一个Object对象
            }
            var returnData = {};
            returnData.recordsTotal = arr.data.totalCount;//totalCount指的是总记录数
            returnData.recordsFiltered = arr.data.totalCount;//后台不实现过滤功能,全部的记录数都需输出到前端，记录数为总数
            returnData.data = arr.data.data;//返回汽车列表
            callback(returnData);// 必须
        }
    })
},
```

## 定制每列
```js
"aoColumnDefs": [
  {"bVisible": false, "aTargets": [ 3 ]} //控制列的隐藏显示
  {"orderable":false,"aTargets":[0,1,2,3,5,6,10,11,12]},// 制定列不参与排序
  {'className':'text-c', 'aTargets':[0,1,2,3,5,6,10,11,12]}, // 某列的CSS的样式
  {'width':'6%','aTargets':[1, 12]}, // 定制目标属性
  {'width':'8%','aTargets':[4]},
  {'width':'8%','aTargets':[10,11]}
],
```

## 处理数据
```js
'columns': [
    { 
        data: 'id',
        render: function(data, type, row) {
            return '<input type="checkbox" class="editor-active">';
        }
    },
    { data: 'account_id' },
    { data: 'remain_num' },
    { data: 'status' },
    { data: 'update_time' },
	{
        data: 'id',
        render: function(data, type, row) {
            return '<a title="编辑" href="javascript:;" onclick="member_edit(\'编辑\',\'/account/form?id='+data+'\',\'4\',\'\',\'560\')" class="ml-5" style="text-decoration:none"><i class="Hui-iconfont">&#xe6df;</i></a> &nbsp;&nbsp;'
        }
    }
],
```
