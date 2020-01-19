---
title: 阿里Easy Excel自定义样式注解
date: 2019-07-31 22:36:06
tags: open source
permalink: common-excel
keywords: Easy Excel, 样式, 注解
rId: MB-19073101
---

现在是第一版common-excel，基于`alibaba/easyexcel`依赖进行样式扩展封装而来的，增加了一些样式注解


## 可以解决什么问题

1. 注解式样式
2. 代码方式自定义样式
3. 冻结行列样式
4. 合并单元格样式
5. 设置内容下拉框
6. 背景颜色
7. 字体设置
8. 列宽度设置
9. 边框样式
10. 居中方式
11. 是否换行

## 效果

![img - 1](../static/MB19073101-1.png) 

![img - 2](../static/MB19073101-2.png) 

## DEMO

model代码示例

```

@CellFreeze(freezeRow = 2, freezeCol = 2)
// 宽度默认值
@ColumnWidth(width = 40)
// 同时作用于表标题和内容的样式默认值
@CellStyle(fontStyle = @FontStyle(color = IndexedColors.LIGHT_BLUE, size = 14))
public class MyModel1 extends BaseRowModel {

    // easyexcel 解析注解
    @ExcelProperty(value={"pri", "pr1"}, index = 0)
    private String pr1;

    @ExcelProperty(value={"pri", "pr2"}, index = 1)
    private String pr2;

    @ExcelProperty(value = {"propValue1"}, index = 2)
    // 数组方式指定的下拉框约束
    @ExplicitConstraint(source = {"aaa1", "aaa2", "aaa3"})
    @ColumnWidth(width = 100)
    private String propValue1;

    @ExcelProperty(value = {"propValue2"}, index = 3)
    @ExplicitConstraint(enumSource = MyConstraintEnum.class)
    private String propValue2;

    @ExcelProperty(value = {"score"}, index = 4)
    @ColumnWidth(width = 50)
    private Integer score;

    @ExcelProperty(value = {"date"}, index = 5, format = "yyyy-MM-dd HH:mm:ss")
    private Date date;

    public String getPr1() {
        return pr1;
    }

    public void setPr1(String pr1) {
        this.pr1 = pr1;
    }

    public String getPr2() {
        return pr2;
    }

    public void setPr2(String pr2) {
        this.pr2 = pr2;
    }

    public String getPropValue1() {
        return propValue1;
    }

    public void setPropValue1(String propValue1) {
        this.propValue1 = propValue1;
    }

    public String getPropValue2() {
        return propValue2;
    }

    public void setPropValue2(String propValue2) {
        this.propValue2 = propValue2;
    }

    public Integer getScore() {
        return score;
    }

    public void setScore(Integer score) {
        this.score = score;
    }

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }
}
```

详细代码戳下方链接
[仓库主页](https://github.com/Uetty/common-excel)

[有标题头demo](https://github.com/Uetty/common-excel/blob/master/src/main/java/com/uetty/common/excel/demo/withhead/TestWithHeadModel.java)

[无标题头demo1](https://github.com/Uetty/common-excel/blob/master/src/main/java/com/uetty/common/excel/demo/headless/TestHeadlessModel1.java)

[无标题头demo2](https://github.com/Uetty/common-excel/blob/master/src/main/java/com/uetty/common/excel/demo/headless/TestHeadlessModel2.java)