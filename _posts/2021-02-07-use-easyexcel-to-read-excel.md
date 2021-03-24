---
layout:       post
title:        "Java | 使用EasyExcel读取excel"
subtitle:     "自定义监听、转换器等，完成常用的excel读取"
date:         2021-02-07
updated:      2021-02-18
author:       "权芹乐"
header-img:   "img/home-bg.webp"
catalog:      true
tags:
    - Java
    - Excel
---

[toc]

# EasyExcel ？

以前我都是用`Apache poi`解析Excel，但我不喜欢自己处理模型转换、数值类型转换，而且poi处理大excel时内存高。后来听说了阿里开源的EasyExcel，决定试用下。

官方如此自我介绍：
> 快速、简单避免OOM的Java处理Excel工具
> 
> EasyExcel是一个基于Java的简单、省内存的读写Excel的开源项目。在尽可能节约内存的情况下支持读写百M的Excel。 github地址：https://github.com/alibaba/easyexcel

# 引入依赖
我的maven pom文件如下
```xml
<properties>
    <jdk.version>11</jdk.version>
</properties>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>2.2.7</version>
</dependency>

<!-- 以下非必须，视自己的实际情况决定是否引入 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.10</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```

# 极简读Excel示例

## 1. 数据类`DemoBizExcelRow.java`，存储excel一行的数据
```java
@Data // 使用lombok，或自己实现get/set
public class DemoBizExcelRow {
    private String string;
    private Date date;
    private Double doubleData;
}
```

## 2. 自定义数据行监听类`DemoDataListener.java`，逐行解析数据

如果想要获得解析结果集的话，可以像下面这样通过构造函数传参来接收。如果要在监听类里完成DB存储操作，可以参考官方示例`DemoDataListener.java`。
```java
public class DemoDataListener extends AnalysisEventListener<DemoBizExcelRow> {

    List<DemoBizExcelRow> list = new ArrayList<DemoBizExcelRow>();

    public DemoDataListener() { }

    /**
     * 如果使用了spring，请使用这个构造方法。每次创建Listener的时候需要把spring管理的类传进来
     */
    public DemoDataListener(List<DemoBizExcelRow> list) {
        this.list = list;
    }

    /**
     * 这个每一条数据解析都会来调用
     *
     * @param data
     *            one row value. Is is same as {@link AnalysisContext#readRowHolder()}
     * @param context
     */
    @Override
    public void invoke(DemoBizExcelRow data, AnalysisContext context) {
        list.add(data);
    }

    /**
     * 所有数据解析完成了 都会来调用
     *
     * @param context
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) { }
}
```

## 3. 使用
```java
String file = "D:\\Demo.xlsx";
List<DemoExcelRow> list = new ArrayList<DemoExcelRow>();

try {
    EasyExcel.read(file, DemoExcelRow.class, new DemoDataListener<>(list))
            .sheet(0)
            .headRowNumber(1)
            .doRead();
} catch (Exception e) {
    LOGGER.error("e={}", e);
}
```

使用起来不复杂，自己只要做两件事：1.定义数据类 2.自定义监听（用于获取excel内容，或者完成dao操作等等）

# 自定义读取Excel

可是，在我自己的实际使用中，我还需要以下的功能：
+ 表头有多行，每类业务表excel还互不相同
+ 获取表头数据，至少包括排序、列名
+ 校验部分表头名，以确保excel符合要求，如不符，直接退出
+ 解析时数据校验，比如，必填项不能为空，特定列的值不能超出某个范围，特定列值通过enum转换
+ 获取每行中所有错误列集合，然而，官方默认行为是，遇到某个单元格有误的时候忽略剩余单元格，跳到下一行

## 1. 返回结果类

上面的例子只返回了有效数据集合`List<DemoExcelRow>`，而现在想要a）有效数据 b）错误信息 c）表头信息，设计一个结果类处理起来更方便。
```java
@Data
public class ReadExcelResult<T> {

    /**
     * 表头map。
     * 行index->表头整行
     */
    Map<Integer, Map<Integer, CellData>> rowIdx2HeadMap;

    /**
     * 有效数据map。
     * 行index->数据整行
     */
    Map<Integer, T> rowIdx2RowDataMap;

    /**
     * 错误行列map。
     * 行index->错误列index
     */
    Map<Integer, Set<Integer>> rowIdx2ErrColIdxMap;

    /**
     * 初始化变量，使之都不为空
     */
    public ReadExcelResult() {
        this.rowIdx2HeadMap = new TreeMap<>();
        this.rowIdx2RowDataMap = new TreeMap<>();
        this.rowIdx2ErrColIdxMap = new TreeMap<>();
    }
}
```

## 2. 数据类`DemoExcelRow.java`
```java
@Data // 使用lombok，或自己实现get/set
public class DemoExcelRow {

    /**
     * 表头所在行
     * 行号从1开始
     */
    @ExcelIgnore
    private final static int HEAD_ROW_NUMBER = 5;
    /**
     * 列数
     * index从0开始，last_index==列数-1
     */
    @ExcelIgnore
    private final static int COLUMN_LAST_NUMBER = 37;
    /**
     * 期望的表头
     * <p>主要用于表格合法性校验。这里可以只校验必要的字段，即，配置实际excel的表头字段的子集。</p>
     * <p>当为null时，不校验表头</p>
     * <p>key是表头排序，即columnIndex，从0开始；</p>
     * <p>value是表头名，可以忽略前后空格，但必须包含中间空格和换行</p>
     */
    @ExcelIgnore
    private final static Map<Integer, String> HEAD_CHECK_MAP = new HashMap<>() {
        {
            put( 0, "索引号");
            put( 1, "选取样本特征");
            put( 2, "发函单位（客户）*");
        }
    };

    /* ---- 以下是表格一一对应的字段 ---- */


    @ExcelProperty({"索引号"})
    private String string;
    @ExcelProperty(value = "会计科目", converter = SubjectConverter.class)
    private String subject;
    @ExcelProperty("存款日期")
    private Date date;
    @ExcelProperty(index = 5)
    @NumberFormat("#.##")
    private Double doubleData;

    public static int getHeadRowNumber() {
        return HEAD_ROW_NUMBER;
    }

    public static int getColumnLastNumber() {
        return COLUMN_LAST_NUMBER;
    }

    public static Map<Integer, String> getHeadCheckMap() {
        return HEAD_CHECK_MAP;
    }
}
```
解读：
+ 变量`subject`一个转换器，下面会说
+ 变量`doubleData`通过注解`@NumberFormat("#.##")`指定两位小数，通过`@ExcelProperty(index = 5)`设定它在第6列
+ 默认所有字段都会和excel匹配，通过`@ExcelIgnore`说明某变量不是excel中的字段
+ `HEAD_ROW_NUMBER`真正的表头所在行
+ `HEAD_CHECK_MAP`用于表头校验的map

## 3. 转换器
转换器`SubjectConverter.java`将excel中的文本转成enum中的1、2、3，如本例中，读到"应收账款"转存为16。

下面的代码只有`convertToJavaData()`被修改了，其他都是继承来的，IDE会自动生成，所以这部分不写了。
```java
public class SubjectConverter implements Converter<String> {

// 省略了一些代码

    /**
     * 往来明细表 往来账项列示  会计科目
     */
    public final static Map<String, Integer> CONTACT_SUBJECT_MAP = new HashMap<>() {
        {
            put("应收账款",16);
            put("预收款项",17);
            put("应付账款",18);
            put("预付款项",19);
        }
    };

    /**
     * Convert excel objects to Java objects
     *
     * @param cellData            Excel cell data.NotNull.
     * @param contentProperty     Content property.Nullable.
     * @param globalConfiguration Global configuration.NotNull.
     * @return Data to put into a Java object
     * @throws Exception Exception.
     */
    @Override
    public String convertToJavaData(CellData cellData, ExcelContentProperty contentProperty, GlobalConfiguration globalConfiguration) throws Exception {
        String key = cellData.getStringValue();
        Integer val = CONTACT_SUBJECT_MAP.get(key);
        if (val == null) {
            throw new ParseException("fail to convert 会计科目: " + key, -1);
        }
        return val.toString();
    }

}
```

## 4. 自定义单元格解析的监听

`ReadAllCellDataThrowExceptionLastListener.java`类照搬了官方`ModelBuildEventListener.java`，只是将其中遇到错误格中止当前行剩余内容，改成了遇到错误不中止继续读完整行。

下面只写出被修改的函数，相同部分不写了，直接在`ModelBuildEventListener.java`里抄了。
```java
public class ReadAllCellDataThrowExceptionLastListener extends
        AbstractIgnoreExceptionReadListener<Map<Integer, CellData>> {

// 省略了一些代码

    private Object buildUserModel(Map<Integer, CellData> cellDataMap, ReadHolder currentReadHolder,
                                  AnalysisContext context) {
        ExcelReadHeadProperty excelReadHeadProperty = currentReadHolder.excelReadHeadProperty();
        Object resultModel;
        try {
            resultModel = excelReadHeadProperty.getHeadClazz().getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            throw new ExcelDataConvertException(context.readRowHolder().getRowIndex(), 0,
                    new CellData(CellDataTypeEnum.EMPTY), null,
                    "Can not instance class: " + excelReadHeadProperty.getHeadClazz().getName(), e);
        }
        Map<Integer, Head> headMap = excelReadHeadProperty.getHeadMap();
        Map<String, Object> map = new HashMap<String, Object>(headMap.size() * 4 / 3 + 1);
        Map<Integer, ExcelContentProperty> contentPropertyMap = excelReadHeadProperty.getContentPropertyMap();
        LinkedList<ExcelDataConvertException> exceptionLinkedList = new LinkedList<>();
        for (Map.Entry<Integer, Head> entry : headMap.entrySet()) {
            Integer index = entry.getKey();
            if (!cellDataMap.containsKey(index)) {
                continue;
            }
            CellData cellData = cellDataMap.get(index);
            if (cellData.getType() == CellDataTypeEnum.EMPTY) {
                continue;
            }
            ExcelContentProperty excelContentProperty = contentPropertyMap.get(index);
            try {
                Object value = ConverterUtils.convertToJavaObject(cellData, excelContentProperty.getField(),
                        excelContentProperty, currentReadHolder.converterMap(), currentReadHolder.globalConfiguration(),
                        context.readRowHolder().getRowIndex(), index);
                if (value != null) {
                    map.put(excelContentProperty.getField().getName(), value);
                }
            } catch (ExcelDataConvertException e) {
                exceptionLinkedList.add(e);
            }
        }
        
        if (CollectionUtils.isEmpty(exceptionLinkedList)) {
            //没有异常，则转换为需要的map
            BeanMap.create(resultModel).putAll(map);
            return resultModel;
        } else {
            //存在异常，挨个抛出，最后一个异常往外抛结束运行
            for (int i = 0; i < exceptionLinkedList.size(); i++) {
                ExcelDataConvertException exception = exceptionLinkedList.get(i);
                if (i == exceptionLinkedList.size() - 1) {
                    // 最后
                    throw exception;
                } else {
                    handleException(context, exception);
                }
            }
            return null;
        }

    }

    private void handleException(AnalysisContext analysisContext, Exception e) {
        for (ReadListener readListenerException : analysisContext.currentReadHolder().readListenerList()) {
            try {
                readListenerException.onException(e, analysisContext);
            } catch (RuntimeException re) {
                throw re;
            } catch (Exception e1) {
                throw new ExcelAnalysisException(e1.getMessage(), e1);
            }
        }
    }
}
```

## 5. 自定义数据行解析的监听

```java
public class MyBizDemoListener<T> extends AnalysisEventListener<T> {

    /**
     * 期望的表头
     * <p>用于表格合法性校验。这里可以只校验必要的字段，即，配置实际excel的表头字段的子集。</p>
     * <p>当为null时，不校验表头</p>
     * <p>key是表头排序，即columnIndex，从0开始；</p>
     * <p>value是表头名，可以忽略前后空格，但必须包含中间空格和换行</p>
     */
    protected Map<Integer, String> headCheckMap;
    /**
     * 读取excel后，存入该对象
     */
    protected ReadExcelResult<T> readExcelResult;

    /**
     * 表头行/列
     */
    protected Map<Integer, Map<Integer, CellData>> rowIdx2HeadMap = new TreeMap<>();
    /**
     * 有效行/列信息
     */
    protected Map<Integer, T> rowIdx2RowDataMap = new TreeMap<>();
    /**
     * 错误行/列信息
     */
    protected Map<Integer, Set<Integer>> rowIdx2ErrColIdxMap = new TreeMap<>();
    /**
     * 当前行的错误列index集合
     */
    protected Set<Integer> currentRowErrorColumnIndexSet;

    public MyBizDemoListener() {
        this(new ReadExcelResult<>(), null);
    }

    /**
     * @param readExcelResult 读取excel后，存入该对象
     * @param headCheckMap    表格头校验map。A map contains columnIndex<->表头名. If null, do not check head
     */
    public MyBizDemoListener(ReadExcelResult<T> readExcelResult, Map<Integer, String> headCheckMap) {
        this.readExcelResult = readExcelResult;
        this.headCheckMap = headCheckMap;
    }

    /**
     * 处理表头
     * 一行一行调用该函数
     * @param headMap -
     * @param context -
     */
    @Override
    public void invokeHead(Map<Integer, CellData> headMap, AnalysisContext context) {
        super.invokeHead(headMap, context);

        ReadRowHolder readRowHolder = context.readRowHolder();
        int rowIndex = readRowHolder.getRowIndex();

        rowIdx2HeadMap.put(rowIndex, headMap);

        /*
         * 表头合法性校验
         */
        if (headCheckMap != null && !headCheckMap.isEmpty()) {
            int headRowNumber = context.readSheetHolder().getHeadRowNumber();
            if (headRowNumber == rowIndex + 1) {
                for (Integer key : headCheckMap.keySet()) {
                    String expect = headCheckMap.get(key).trim();
                    CellData cell = headMap.get(key);
                    if (null == cell) {
                        //模板不符！退出
                        throw new ExcelHeadException("表头与预期不符。未找到表头：" + expect);
                    }

                    String real = cell.getStringValue();
                    real = (real==null? null : real.trim());
                    if (!expect.equalsIgnoreCase(real)) {
                        //模板不符！退出
                        throw new ExcelHeadException("表头与预期不符。期望：" + expect + " <--> 实际：" + real);
                    }
                }
            }
        }

    }

    /**
     * When analysis one row trigger invoke function.
     * 自动跳过空行，即，空行不会进入这个函数
     *
     * @param data    one row value. Is is same as {@link AnalysisContext#readRowHolder()}
     * @param context -
     */
    @Override
    public void invoke(T data, AnalysisContext context) {

        currentRowErrorColumnIndexSet = new TreeSet<>();

        ReadRowHolder readRowHolder = context.readRowHolder();
        int rowIndex = readRowHolder.getRowIndex();

        /**
         * 非空校验方法。业务强相关，略去源码。可以通过特定的注解判断必填
         */
        ReadExcelUtil.checkNotEmpty(data, context, currentRowErrorColumnIndexSet);

        /**
         * 值有效性校验。业务强相关，略去源码
         */
        ReadExcelUtil.checkFieldValueInStringMap(data, "sendType", ReadExcelConstant.SEND_TYPE_MAP, context, currentRowErrorColumnIndexSet);

        if (currentRowErrorColumnIndexSet.isEmpty()) {
            rowIdx2RowDataMap.put(rowIndex, data);
        } else {
            Set<Integer> errColIdxMap = rowIdx2ErrColIdxMap.get(rowIndex);
            if (errColIdxMap != null) {
                currentRowErrorColumnIndexSet.addAll(errColIdxMap);
            }
            rowIdx2ErrColIdxMap.put(rowIndex, currentRowErrorColumnIndexSet);
        }

    }

    /**
     * if have something to do after all analysis
     *
     * @param context -
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        readExcelResult.setRowIdx2HeadMap(rowIdx2HeadMap);
        readExcelResult.setRowIdx2ErrColIdxMap(rowIdx2ErrColIdxMap);
        readExcelResult.setRowIdx2RowDataMap(rowIdx2RowDataMap);
    }


    /**
     * The current method is called when extra information is returned
     *
     * @param extra   extra information
     * @param context -
     */
    @Override
    public void extra(CellExtra extra, AnalysisContext context) { }

    /**
     * All listeners receive this method when any one Listener does an error report. If an exception is thrown here, the
     * entire read will terminate.
     * 获取其他异常下会调用本接口。抛出异常则停止读取。如果这里不抛出异常则 继续读取下一行。
     *
     * @param exception -
     * @param context -
     */
    @Override
    public void onException(Exception exception, AnalysisContext context) throws Exception {

        // 如果是某一个单元格的转换异常，能获取到具体行号
        // 如果要获取头的信息，配合invokeHeadMap使用
        if (exception instanceof ExcelDataConvertException) {
            ExcelDataConvertException excelDataConvertException = (ExcelDataConvertException)exception;
            Integer cellRowIndex = excelDataConvertException.getRowIndex();
            Integer cellColumnIndex = excelDataConvertException.getColumnIndex();

            String cellColumnString = CellReference.convertNumToColString(cellColumnIndex);
            LOGGER.error("第{}行{}列，数值转换异常：{}", cellRowIndex+1, cellColumnString, exception.getMessage());

            Set<Integer> errColIdxMap = rowIdx2ErrColIdxMap.get(cellRowIndex);
            if (errColIdxMap == null) {
                errColIdxMap = new TreeSet<>();
            }
            errColIdxMap.add(cellColumnIndex);
            rowIdx2ErrColIdxMap.put(cellRowIndex, errColIdxMap);
        } else if (exception instanceof ExcelHeadException) {

            LOGGER.error(exception.getMessage());

            // 表格不符合规范，抛出异常，触发终止解析
            throw exception;
        } else {
            LOGGER.error("第{}行解析失败，但是继续解析下一行。exception: \n{}",
                    context.readRowHolder().getRowIndex() + 1,
                    Arrays.toString(exception.getStackTrace()).replaceAll(",", "\n"));
        }
    }
}
```
解读：
+ 在`invokeHead()`中，借助构造函数传入的变量`headCheckMap`校验excel表头
+ 在`onException()`中，仅当`throw exception`时才会终止excel解析，否则只是跳过到下一行
+ 在`invoke()`中，处理必填项、数值合法性校验等等

## 6. 使用

```java
String file = "D:\\Demo.xlsx";

ReadExcelResult<DemoBizExcelRow> readExcelResult = new ReadExcelResult<>();
ExcelReader excelReader = null;
try {
    MyBizDemoListener<DemoBizExcelRow> myBizDemoListener = new MyBizDemoListener<>(readExcelResult, DemoBizExcelRow.getHeadCheckMap());
    excelReader = EasyExcel.read(file, DemoBizExcelRow.class, null)
            .useDefaultListener(false)
            .registerReadListener(new ReadAllCellDataThrowExceptionLastListener())
            .registerReadListener(myBizDemoListener)
            .build();
    ReadSheet readSheet = EasyExcel.readSheet(0).headRowNumber(DemoBizExcelRow.getHeadRowNumber()).build();
    excelReader.read(readSheet);
} catch (Exception e) {
    if (e instanceof ExcelHeadException) {
        LOGGER.error("Excel模板错误！");
    } else {
        LOGGER.error("其他异常");
    }
} finally {
    if (excelReader != null) {
        excelReader.finish();
    }
}

// 打印 readExcelResult
```
解读：
+ `EasyExcel`默认提供了解析单元格的监听（即上面提到的`ModelBuildEventListener.java`），如果要使用自定义的单元格解析监听，要先去掉默认`useDefaultListener(false)`，再注册自己的`registerReadListener(new ReadAllCellDataThrowExceptionLastListener())`
+ 在使用自定义单元格解析监听情况下，不能通过`EasyExcel.read()`传入自定义的行解析监听，只能通过`registerReadListener()`注册，并且，（这一点很重要）**要放在注册自定义单元格监听之后**。

# 补充

## 格式转换
### 日期
```java
@ExcelProperty("开始时间")
@DateTimeFormat("yyyy-MM-dd")
private Date startTime;
```
* 2020/1/2 --> 1577894400000（即，2020-01-02）
* 你好 --> throw new Exception()
* // 转换失败时，会抛异常

```java
@ExcelProperty("结束时间")
@DateTimeFormat("yyyy-MM-dd")
private String endTime;
```
* 2019/2/5 --> 2019-02-05
* 你好 --> 你好
* // 转换失败时，保持原值，不会抛异常

### 数值
```java
@ExcelProperty("存钱")
@NumberFormat("#.##")
private Double deposit;
```
* 1.2345 --> 1.23
* 你好 --> throw new Exception()
* // 转换失败时，会异常

```java
@ExcelProperty("取钱")
@NumberFormat("#.##")
private String withdraw;
```
* 1.2345 --> 1.23
* 你好 --> 你好
* // 转换失败时，保持原值，不会抛异常
