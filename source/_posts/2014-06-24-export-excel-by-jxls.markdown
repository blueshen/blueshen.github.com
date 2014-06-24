---
layout: post
title: "使用 jXLS导出报表"
date: 2014-06-24 15:17
comments: true
categories: Java
tags: [ apache poi, excel, jXLS, jxl ]
---
###常用的excel操作工具

- Apache POI
- jexcelApi

它们都提供了完善的API来支持EXCEL的读写。

###jXLS是什么？

jXLS是基于apache poi的一个扩展。它的功能就类似于jstl在servlet中的作用，你可以自定义一个模板，然后往里面放数据就OK了。

jXLS的基本功能：

- 支持Excel 95-2000的所有版本
- 生成Excel 2000标准格式
- 支持字体、数字、日期操作
- 能够修饰单元格属性
- 支持图像和图表

        <dependency>
               <groupId>net.sf.jxls</groupId>
               <artifactId>jxls-core</artifactId>
               <version>1.0.5</version>
        </dependency>
        
<!--more-->
jXLS的API也很简单：

                Map beans = new HashMap();
                beans.put("department", department);
                XLSTransformer transformer = new XLSTransformer();
                transformer.transformXLS(xlsTemplateFileName, beans, outputFileName);
###jXLS是如何解析模板生成数据的？

查看`net.sf.jxls.transformer.CellTransformer`可以看到：

    Object value = ((Expression) cell.getExpressions().get(0)).evaluate();
而evaluate()方法如下：

    public Object evaluate() throws Exception {
        if (beans != null && !beans.isEmpty()) {
            JexlContext context = new MapContext(beans);
            Object ret = jexlExpresssion.evaluate(context);
            if (aggregateFunction != null) {
                return calculateAggregate(aggregateFunction, aggregateField, ret);
            }
            return ret;
        }
        return expression;
    }
从而可以看出，数据的获得是通过JEXL来实现的。换句话说，模板内可以使用任何的JEXL标签。

###让jXLS支持hyperlink

业务上有这么一个需求，想在单元格内显示一个链接，而链接是动态生成的。通过模板设置单元格为链接始终无法生效，但是如果链接是固定的是可以生效的。如何实现动态的呢，改源码net.sf.jxls.transformer.CellTransformer 82行：

        if (cell.getStringCellValue().toLowerCase().startsWith("${href}") && cell.getExpressions().size() == 3){
               //是链接类型的cell
               HSSFCell hssfCell = (HSSFCell) cell.getPoiCell();
               Expression valueExpr = (Expression)cell.getExpressions().get(1);
               Expression linkExpr = (Expression)cell.getExpressions().get(2);
               hssfCell.setCellValue((String)valueExpr.evaluate());
               Hyperlink link = new HSSFHyperlink(Hyperlink.LINK_URL);
               link.setAddress((String)linkExpr.evaluate());
               hssfCell.setHyperlink(link);
               cell.setPoiCell(hssfCell);
            }
当遇见${href}开头的表达式，则认为是链接CELL。此处${href}只是一个标记，可以任意替换的啦。然后将后面的第1个表达式作为显示值，第2个表达式作为链接地址就实现啦。