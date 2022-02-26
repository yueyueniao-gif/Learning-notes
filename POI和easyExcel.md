---
title: POI和easyExcel
author: Wang Yue Niao
top: false
toc: true
date: 2020-08-25 19:56:53
tags: excel
categories: excel
---

### POI

POI是Apache软件基金会的，POI为“Poor Obfuscation Implementation”的首字母缩写，意为“简洁版的模糊实现”。
所以**POI的主要功能是可以用Java操作Microsoft Office的相关文件**，这里我们主要讲Excel

##### 导入依赖

```java
 <dependency>
     <groupId>org.apache.poi</groupId>
     <artifactId>poi</artifactId>
     <version>3.17</version>
</dependency>
     //下面是07(xlsx)版本的，上面是03(xls)
<dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
      <version>3.17</version>
</dependency>
```

##### 写操作

```
Workbook wordkbook =new ****Workbook();//创建一个Workbook对象
 wordkbook.createSheet();//创建表名，如果不写参数，会有默认值
 Row row1=sheet.createRow(0);//根据里面的数字拿到对应的行，0默认为第一行
 Cell cell = row1.createCell(0);//根据行对象创建单元格，这里0为第一个
 cell.setCellValue("");//可以给单元格赋值
```



HSSF：Excel97-2003版本，扩展名为.xls。一个sheet最大行数65536，最大列数256，扩展名为.xls

*XSSF：*Excel2007版本开始，扩展名为.xlsx。一个sheet最大行数1048576，最大列数16384，扩展名为.xlsx。

​            虽然写入数量变多了，但是速度变慢了，虽有有了他的优化版，SXSSF。

SXSSF：是在XSSF基础上，POI3.8版本开始提供的支持低内存占用的操作方式，扩展名为.xlsx。



03版本的写

```java
public class ExcelWriteTest  { 
    String path="D:\\IDEA代码\\POI && easyExcel\\wang-POI\\";
    public void testWrite03() throws Exception {
        //1.创建一个工作簿
        Workbook workbook=new HSSFWorkbook();
        //2.创建一个工作表
        Sheet sheet=workbook.createSheet("niaoniao统计表");
        //3.创建一个行
        Row row1=sheet.createRow(0);
        //4.创建一个单元格(1,1)
        Cell cell01=row1.createCell(0);
        cell01.setCellValue("今日观众第一个格子");
        //(1,2)
        Cell cell02=row1.createCell(1);
        cell02.setCellValue("666第二个格子");

        //第二行(2,1)
        Row row2=sheet.createRow(1);
        Cell cell021=row2.createCell(0);
        cell021.setCellValue("统计时间");
        //(2,2)
        Cell cell022=row2.createCell(1);
        String time = new DateTime().toString("yyyy-MM-dd HH:mm:ss");
        cell022.setCellValue(time);

        //生成一张表(io流) 03版本就是使用xls结尾！
        FileOutputStream fileOutputStream = new FileOutputStream(path + "wangyueniao统计表.xls");
        //输出
        workbook.write(fileOutputStream);

        ///关闭流
        fileOutputStream.close();
        System.out.println("输出完毕");
    }
}
```

07版本的写

```java
public class ExcelWriteTest {
     String path="D:\\IDEA代码\\POI && easyExcel\\wang-POI\\";
    public void testWrite07() throws Exception {
        //1.创建一个工作簿
        Workbook workbook=new XSSFWorkbook();
        //2.创建一个工作表
        Sheet sheet=workbook.createSheet("niaoniao统计表");
        //3.创建一个行
        Row row1=sheet.createRow(0);
        //4.创建一个单元格(1,1)
        Cell cell01=row1.createCell(0);
        cell01.setCellValue("今日观众第一个格子");
        //(1,2)
        Cell cell02=row1.createCell(1);
        cell02.setCellValue("666第二个格子");

        //第二行(2,1)
        Row row2=sheet.createRow(1);
        Cell cell021=row2.createCell(0);
        cell021.setCellValue("统计时间");
        //(2,2)
        Cell cell022=row2.createCell(1);
        String time = new DateTime().toString("yyyy-MM-dd HH:mm:ss");
        cell022.setCellValue(time);

        //生成一张表(io流) 07版本就是使用xlsX结尾！
        FileOutputStream fileOutputStream = new FileOutputStream(path + "wangyueniao统计表.xlsX");
        //输出
        workbook.write(fileOutputStream);

        ///关闭流
        fileOutputStream.close();
        System.out.println("输出完毕");
    }
}
```

07版本的优化

```java
public class ExcelWriteTest {
    String path="D:\\IDEA代码\\POI && easyExcel\\wang-POI\\";
       public  void testWrite07BigDataSuper() throws Exception {
        //开始时间
        long begin = System.currentTimeMillis();
        //创建一个工作簿
        Workbook workbook=new SXSSFWorkbook();
        //创建表
        Sheet sheet = workbook.createSheet();
        //写入数据
        for(int rowNum=0;rowNum<65536;rowNum++){
            Row row = sheet.createRow(rowNum);
            for(int cellNum=0;cellNum<10;cellNum++){
                Cell cell = row.createCell(cellNum);
                cell.setCellValue(cellNum);
            }
        }
        System.out.println("结束");
        FileOutputStream fileOutputStream = new FileOutputStream(path + "testWrite0BigDataS.xlsX");
        workbook.write(fileOutputStream);
        //清除临时文件
        ((SXSSFWorkbook) workbook).dispose();
        //结束时间
        long end = System.currentTimeMillis();

        System.out.println("共用时"+(double)(end-begin));
    }
}
```

##### 读操作

03版本的读

```java
public class ExcelReadTest {
    String path="D:\\IDEA代码\\POI && easyExcel\\wang-POI\\";
    public void testRead03() throws Exception {

        //获取文件流
        FileInputStream fileInputStream = new FileInputStream(path + "wangyueniao统计表.xls");

        //1.创建一个工作簿，使用excel能操作的这边都能操作
        Workbook workbook=new HSSFWorkbook(fileInputStream);
        //得到表
        Sheet sheetAt = workbook.getSheetAt(0);
        Row row = sheetAt.getRow(0);
        Cell cell = row.getCell(0);
        //读取值的时候，一定需要注意类型
        String stringCellValue = cell.getStringCellValue();
        System.out.println(stringCellValue);

    }
}
```

07版本的读

```java
public class ExcelReadTest {
    String path="D:\\IDEA代码\\POI && easyExcel\\wang-POI\\";
     public void testRead07() throws Exception {

        //获取文件流
        FileInputStream fileInputStream = new FileInputStream(path + "wang-POItestWrite0BigDataS.xlsX");

        //1.创建一个工作簿，使用excel能操作的这边都能操作
        Workbook workbook=new XSSFWorkbook(fileInputStream);
        //得到表
        Sheet sheetAt = workbook.getSheetAt(0);
        Row row = sheetAt.getRow(0);
        Cell cell = row.getCell(0);
        //读取值的时候，一定需要注意类型
        double numericCellValue = cell.getNumericCellValue();
        System.out.println(numericCellValue);

    }
}
```

###### 读取表中所有的数据

```java
public class ExcelReadTest {
    String path="D:\\IDEA代码\\POI && easyExcel\\wang-POI\\";
    public void testRead() throws Exception {
        //获取文件流
        FileInputStream fileInputStream = new FileInputStream(path + "wangyueniao统计表.xls");
        //1.创建一个工作簿，使用excel能操作的这边都能操作
        Workbook workbook=new HSSFWorkbook(fileInputStream);
        //得到表
        Sheet sheetAt = workbook.getSheetAt(0);

        //获取表中的内容
        //获取总行数
        int rowcounts = sheetAt.getPhysicalNumberOfRows();
        System.out.println("总行数："+rowcounts);
        for(int rowNum=1;rowNum<rowcounts;rowNum++){
            Row rowData= sheetAt.getRow(rowNum);
            if(rowData!=null){
                //读取列
                int cellCount = rowData.getPhysicalNumberOfCells();
                System.out.println("总列数"+cellCount);
                for(int cellNum=0;cellNum<cellCount;cellNum++){
                    Cell cell = rowData.getCell(cellNum);
                    //匹配数据类型
                    if(cell!=null){
                        int cellType = cell.getCellType();
                        String cellValue="";

                        switch(cellType){
                            case HSSFCell.CELL_TYPE_STRING: //字符串
                                System.out.println("【String】");
                                cellValue = cell.getStringCellValue();
                                break;
                            case HSSFCell.CELL_TYPE_BOOLEAN: //布尔
                                System.out.println("【BOOLEAN】");
                                cellValue = String.valueOf(cell.getStringCellValue());
                                break;
                            case HSSFCell.CELL_TYPE_BLANK: //空
                                System.out.println("【BLANK】");
                                cellValue = String.valueOf(cell.getStringCellValue());
                                break;
                            case HSSFCell.CELL_TYPE_NUMERIC: //空
                                System.out.println("【NUMERIC】");
                                if(HSSFDateUtil.isCellDateFormatted(cell)){
                                    System.out.println("日期");
                                    Date date=cell.getDateCellValue();
                                    cellValue = new DateTime(date).toString();
                                }else{
                                    System.out.println("【转换为字符串输出】");
                                    cell.setCellType(HSSFCell.CELL_TYPE_STRING);
                                    cellValue=cell.toString();
                                }
                                break;
                            case HSSFCell.CELL_TYPE_ERROR:
                                System.out.println("【数据类型错误】");
                                break;
                        }
                        System.out.println(cellValue);
                    }
                }
            }
        }
    }
}
```

### easyExcel

##### 导入依赖

```xml
  <dependency>   
            <groupId>com.alibaba</groupId>
            <artifactId>easyexcel</artifactId>
            <version>2.2.0-beta2</version>
        </dependency>
```

具体[传送门](https://www.yuque.com/easyexcel/doc/)