title: 纯前端json数据生成excel下载的实现
date: 2017-06-09 22:51:52
categories:
- 学习
- 技术
tags:
- 总结
- 工作
- 思考
---
本需求是将页面已经获取到的json数据按照产品给的逻辑导出为excel文件，在查阅资料的过程中放弃了使用jQuery的实现方法，决定利用现在浏览器广泛支持的Blob对象和FileSaver插件来实现。
<!-- more -->
### 引入工具

本项目是使用vue2.0的后端项目, 用的ES6的模块化语法。
[file-saver](https://github.com/eligrey/FileSaver.js)
```
npm install file-saver --save-dev

// 在文件中引入并使用并触发下载的model。
import { saveAs } from 'file-saver';

saveAs(new Blob([],{}), name); 
```

### 另存为CSV格式的文件
本质流程是将json数据转换为对应csv格式 在excel表格里面展现出来。关键点如下：
- csv的分隔符规定: 逗号是单元格之间的分隔 `\n`是表格换行。
- 本质是文本文件输出，注意saveAs中对输出的Blob对象的相关参数配置。
- 生成的excel如果细心，会发现时间列会出现丢失0的情况，如何使其按照文本输出？

```javascript
methods:{
    generateJson() { //生成所需json数据逻辑
        let liveStreamId = '';
        let listLength = this.list.length;
        let arr = [];
        let typeMap = {
            'MATCH': '赛事',
            'PROGRAM': '自制节目',
            'OTHER': '其他节目'
        }
        for (let i = 0; i < listLength; i++) {
            var {type, cid, name, startTime, endTime} = this.list[i];
            // 加\t是制表符  从而不让时间中的0丢失 解决了上面提出的问题
            startTime = startTime ? "\t" + moment(startTime, 'YYYYMMDDHHmmss').format('YYYY-MM-DD HH:mm:ss') : " ";
            endTime = endTime ? "\t" + moment(endTime, 'YYYYMMDDHHmmss').format('YYYY-MM-DD HH:mm:ss') : " ";
            type = typeMap[type];
            let streamLength = this.list[i].liveStreams.length;
            if (streamLength > 1) {
                for (let j = 0; j < streamLength; j++) {
                    liveStreamId = this.list[i].liveStreams[j].id;
                    let obj = {type, cid, name, startTime, endTime, liveStreamId};
                    arr.push(obj);
                }
            } else {
                liveStreamId = this.list[i].liveStreams.length ? this.list[i].liveStreams[0].id : '';
                let obj = {type, cid, name, startTime, endTime, liveStreamId};
                arr.push(obj);
            }
        }
        return arr;
    },
    dataToCSV(data) {
        let dataArr = [];
        let titleArr = ['节目类型', '节目名称', '开始时间', '结束时间', '直播流id']; //表头
        let dataKeys = ['type', 'name', 'startTime', 'endTime', 'liveStreamId'];
        dataArr.push(titleArr.join(','));
        for (let j = 0; j < data.length; j++) {
            let itemArr = [];
            for (let i = 0; i < dataKeys.length; i++) {
                itemArr.push(data[j][dataKeys[i]]);
            }
            dataArr.push(itemArr.join(','));
        }
        return dataArr.join('\n');
    },
    function getCSV() {
        let dataJson = this.generateJson();
        let dataJson = [];
        let csvContent = dataToCSV(dataJson);
        let today = moment().format('YYYY-MM-DD');
        let excelName = "直播.csv";
        // "\ufeff"是用来放置乱码加上的
        saveAs(
            new Blob(["\ufeff" + csvContent], {type: "text/plain;charset=utf8"}),
            excelName
        );
    }
}
// 在vue中 点击按钮触发getCSV函数即可进行下载
```
缺点: 不能控制excel的格式，打开后都是默认单元格的大小，很不合理

*参考链接奉上：http://www.cnblogs.com/dojo-lzz/p/4837041.html*


### 另存为xls格式的文件
为了解决excel的样式问题，这是最后采用的方案，原理是excel是xml类型的数据，关键点如下：
- 这个可以通过对table设置样式，从而易用性得到了大大的增强，流程基本没有变化
- 本质是通过拼接xml字符串来写入文件中，并设置文件后缀名位xml。
- 时间列会出现丢失0的情况，这次不能用制表符避免，经查阅发现`&nbsp`可以解决。

```javascript
methods: {
    generateJson() {
        // ...
        startTime = startTime ? moment(startTime, 'YYYYMMDDHHmmss').format('YYYY-MM-DD HH:mm:ss') : " ";
        endTime = endTime ? moment(endTime, 'YYYYMMDDHHmmss').format('YYYY-MM-DD HH:mm:ss') : " ";
        // ...
    },
    dataToExcel(data) {
        let style = "color:Black;background-color:White;border-color:#CCCCCC;border-width:1px;border-style:None;width:100%;border-collapse:collapse;font-size:12pt;text-align:center;";
        let excel = '<table cellspacing="0" rules="rows" border="1" style=' + style + '>';
        let titleArr = ['节目类型', '节目名称', '开始时间', '结束时间', '直播流id'];
        let dataKeys = ['type', 'name', 'startTime', 'endTime', 'liveStreamId'];
        let row = '<tr style="font-size:15pt;font-weight:700;text-align:center;">';
        for (let i = 0; i < titleArr.length; i++) {
            row += "<td>" + titleArr[i] + "</td>";
        }
        excel += row + "</tr>";
        for (let j = 0; j < data.length; j++) {
            let row = "<tr>";
            for (let i = 0; i < dataKeys.length; i++) {
                // 处理时间丢失0的问题
                if (dataKeys[i] === 'startTime' || dataKeys[i] === 'endTime') {
                    row += "<td>&nbsp;" + data[j][dataKeys[i]] + "</td>";
                } else {
                    row += "<td>" + data[j][dataKeys[i]] + "</td>";
                }
            }
            excel += row + "</tr>";
        }
        excel += "</table>";
        return excel;
    },
    generateExcel(excel) {
        var excelFile = "<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:x='urn:schemas-microsoft-com:office:excel' xmlns='http://www.w3.org/TR/REC-html40'>";
        excelFile += '<meta http-equiv="content-type" content="application/vnd.ms-excel; charset=UTF-8">';
        excelFile += '<meta http-equiv="content-type" content="application/vnd.ms-excel';
        excelFile += '; charset=UTF-8">';
        excelFile += "<head>";
        excelFile += "<!--[if gte mso 9]>";
        excelFile += "<xml>";
        excelFile += "<x:ExcelWorkbook>";
        excelFile += "<x:ExcelWorksheets>";
        excelFile += "<x:ExcelWorksheet>";
        excelFile += "<x:Name>";
        excelFile += "{worksheet}";
        excelFile += "</x:Name>";
        excelFile += "<x:WorksheetOptions>";
        excelFile += "<x:DisplayGridlines/>";
        excelFile += "</x:WorksheetOptions>";
        excelFile += "</x:ExcelWorksheet>";
        excelFile += "</x:ExcelWorksheets>";
        excelFile += "</x:ExcelWorkbook>";
        excelFile += "</xml>";
        excelFile += "<![endif]-->";
        excelFile += "</head>";
        excelFile += "<body>";
        excelFile += excel;
        excelFile += "</body>";
        excelFile += "</html>";
        return excelFile;
    }
    getExcel() {
        let dataJson = this.generateJson();
        let excelContent = this.dataToExcel(dataJson);
        let today = moment().format('YYYY-MM-DD');
        let excelName = today + "直播.xls";
        let excel = this.generateExcel(excelContent);
        saveAs(
            new Blob([excel], {type: "application/vnd.ms-excel;charset=utf-8"}),
            excelName
        )
    }
    // 在vue中 点击按钮触发getExcel函数即可进行下载
}
```

*参考链接奉上：http://blog.csdn.net/educast/article/details/52775559*

使用node做后端来返回excel文件 参考此库[json2xls](https://github.com/rikkertkoppes/json2xls)。

心得: 这个过程有人已经封装为jquery插件，不过我是没有采用jquery依赖实现的,从[github-tableExport](https://github.com/kayalshri/tableExport.jquery.plugin)中获得不少启发。 另外还有[npm-xlsx](https://www.npmjs.com/package/xlsx)基本上要把Excel玩坏了  各种来回读取，有时间可以看看。文件的本质就是各种数据流的操作，现在浏览器真心是越来越强大，期待HTML5的文件API被广为支持的一天。
