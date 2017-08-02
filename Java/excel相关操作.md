# excel相关操作

excel的操作基本都用apache的poi工具包来支持。

## excel下载

### XSSFWorkbook
10w左右的数据量，可以使用XSSFWorkbook，可以提供丰富的cell控制能力。但是因为整个文件都在内存中，数据量过大容易导致OOM，同时过多数据需要同步下载等待时间也过长，体验不好。

让宽度自适应写法：
```
for(int i = 0; i < headers.length; i++) {
    sh.autoSizeColumn(i, true);
}
```

### SXSSFWorkbook
10w+的数据，最好用SXSSFWorkbook，可以提供基础的cell控制能力。因为此时生成excel的模式是流模式，生成的时候，内存中的数据量是有限的，超出的部分数据会写入本地临时文件。因此生成excel的时候，不会导致OOM。
```
SXSSFWorkbook wb = new SXSSFWorkbook(100);
SXSSFSheet sh = wb.createSheet("休息休息");

Row firstRow = sh.createRow(0);
String[] headers = new String[]{"sssss,"sssss","ssss"};

for(int i = 0; i < headers.length; i++) {
    Cell cell = firstRow.createCell(i);
    cell.setCellValue(headers[i]);
}
```

SXSSFWorkbook让所有列宽度自适应支持：
```
sh.trackAllColumnsForAutoSizing();

......
创建row
......

for(int i = 0; i < headers.length; i++) {
    sh.autoSizeColumn(i, true);
}
```
让所有的column保持合适width，需要消耗大量的内存与cpu，建议将需要处理的column数目保持在最小。如：
```
sh.trackColumnForAutoSizing(3);
......
for() {
    创建row
    sh.autoSizeColumn(3);
}
......
```

关于下载，最好改进交互方式，由同步下载改为同步接受请求命令，异步下载好后，通知用户去下载。