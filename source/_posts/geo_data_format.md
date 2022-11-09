---
title: 空间搜索-数据格式
date: 2020-01-16 22:03:21
tags: [ spatail search, map, wkt, geojson ]
categories: geo
---

![geojson-bj](/images/geo-data-format/geojson-bj.jpeg)

### 几何对象类型

- Point  点
- MultiPoint 多点
- LineString  线条
- MultiLineString 多线条
- Polygon   多边形
- MultiPolygon N个多边形
- GeometryCollection 几何图形集合

<!--more-->

### Geo数据描述格式

#### WKT(Well-Known Text)

|                            类型                             |                             图形                             | 例子                                                         |
| :---------------------------------------------------------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
|   [Point](https://en.wikipedia.org/wiki/Point_(geometry))   | ![SFA Point.svg](/images/geo-data-format/320px-SFA_Point.svg.png) | `POINT (30 10)`                                              |
| [LineString](https://en.wikipedia.org/wiki/Polygonal_chain) | ![SFA LineString.svg](/images/geo-data-format/320px-SFA_LineString.svg.png) | `LINESTRING (30 10, 10 30, 40 40)`                           |
|      [Polygon](https://en.wikipedia.org/wiki/Polygon)       | ![SFA Polygon.svg](/images/geo-data-format/320px-SFA_Polygon.svg.png) | `POLYGON ((30 10, 40 40, 20 40, 10 20, 30 10))`              |
|    [Polygon有孔](https://en.wikipedia.org/wiki/Polygon)     | ![SFA Polygon with hole.svg](/images/geo-data-format/320px-SFA_Polygon_with_hole.svg.png) | `POLYGON ((35 10, 45 45, 15 40, 10 20, 35 10),(20 30, 35 35, 30 20, 20 30))` |


| 类型                                                         | 图形                                                         | 例子                                                         |
| :----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [MultiPoint](https://en.wikipedia.org/wiki/Point_(geometry)) | ![SFA MultiPoint.svg](/images/geo-data-format/320px-SFA_MultiPoint.svg.png) | `MULTIPOINT ((10 40), (40 30), (20 20), (30 10))` 或者  `MULTIPOINT (10 40, 40 30, 20 20, 30 10)` |
| [MultiLineString](https://en.wikipedia.org/wiki/Line_segment) | ![SFA MultiLineString.svg](/images/geo-data-format/320px-SFA_MultiLineString.svg.png) | `MULTILINESTRING ((10 10, 20 20, 10 40),(40 40, 30 30, 40 20, 30 10))` |
| [MultiPolygon](https://en.wikipedia.org/wiki/Polygon)        | ![SFA MultiPolygon.svg](/images/geo-data-format/320px-SFA_MultiPolygon.svg.png) | `MULTIPOLYGON (((30 20, 45 40, 10 40, 30 20)),((15 5, 40 10, 10 20, 5 10, 15 5)))` |
| [MultiPolygon有孔](https://en.wikipedia.org/wiki/Polygon)    | ![SFA MultiPolygon with hole.svg](/images/geo-data-format/320px-SFA_MultiPolygon_with_hole.svg.png) | `MULTIPOLYGON (((40 40, 20 45, 45 30, 40 40)),((20 35, 10 30, 10 10, 30 5, 45 20, 20 35),(30 20, 20 15, 20 25, 30 20)))` |
| GeometryCollection                                           | ![SFA GeometryCollection.svg](/images/geo-data-format/320px-SFA_GeometryCollection.svg.png) | `GEOMETRYCOLLECTION (POINT (40 10),LINESTRING (10 10, 20 20, 10 40),POLYGON ((40 40, 20 45, 45 30, 40 40)))` |

GeometryCollection是一个符合类型，用于表示多个复杂基本类型的集合。

**需要重点关注**：Polygon多边形的表示方式，当顶点顺序**逆时针**标识的是内部的多边形，**顺时针**标识的是外部的多边形，这点容易出错。应仔细观察图中有孔洞的表达形式。


#### GeoJSON

![GeoJson](/images/geo-data-format/1*X0tZ674WA4iAvERMqApzxQ.png)

GeoJSON的类型与WKT一样，从名字上也可以看出是通过JSON来表达几何图形。除了GeometryCollection外，都必须包含一个coordinates成员，用来标识坐标。

##### Point

```json
{ 
  "type": "Point",
  "coordinates": [100.0, 0.0]
}
```

##### MultiPoint

```json
{
    "type": "MultiPoint",
    "coordinates": [
        [ 100, 0 ],
        [ 101, 1 ]
    ]
}
```

##### LineString

```json
{
    "type": "LineString",
    "coordinates": [
        [ 100, 0 ],
        [ 101, 1 ]
    ]
}
```

##### MultiLineString

```json
{
    "type": "MultiLineString",
    "coordinates": [
        [ [100.0, 0.0], [101.0, 1.0] ],
        [ [102.0, 2.0], [103.0, 3.0] ]
    ]
}
```

##### Polygon

```json
{
    "type": "Polygon",
    "coordinates": [
        [
            [ 100, 0 ],
            [ 101, 0 ],
            [ 101, 1 ],
            [ 100, 1 ],
            [ 100, 0 ]
        ]
    ]
}
```

有孔的(注意顺序)：

```json
{
    "type": "Polygon",
    "coordinates": [
        [
            [ 100, 0 ],
            [ 101, 0 ],
            [ 101, 1 ],
            [ 100, 1 ],
            [ 100, 0 ]
        ],
        [
            [ 100.2, 0.2 ],
            [ 100.8, 0.2 ],
            [ 100.8, 0.8 ],
            [ 100.2, 0.8 ],
            [ 100.2, 0.2 ]
        ]
    ]
}
```

##### MultiPolygon

```json
{
  "type": "MultiPolygon",
  "coordinates":
    [ 
        [
          [[102.0, 2.0], [103.0, 2.0], [103.0, 3.0], [102.0, 3.0], [102.0, 2.0]]
        ],
        [
            [
                [100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0]
            ],
            [
                [100.2, 0.2], [100.8, 0.2], [100.8, 0.8], [100.2, 0.8], [100.2, 0.2]
            ]
        ]
    ]
}
```

##### GeometryCollection

```json
{ 
	"type": "GeometryCollection",
  "geometries": [
    { 
    	"type": "Point",
      "coordinates": [100.0, 0.0]
    },
    { 
    	"type": "LineString",
      "coordinates": [ [101.0, 0.0], [102.0, 1.0] ]
    }
  ]
}
```

此外，GeoJSON支持为集合类型添加特征属性，这就是`feature`类型，而多个`feature`类型可以组织成`FeatureCollection`形式。
##### Feature对象

特征对象必须由一个名字为"geometry"的成员，这个几何成员的值是上面定义的几何对象或者JSON的null值。

特征必须有一个名字为“properties"的成员，这个属性成员的值是一个对象（任何JSON对象或者JSON的null值）。

如果特征是常用的标识符，那么这个标识符应当包含名字为“id”的特征对象成员。

```json
{
    "type":"Feature",
    "properties":{},
    "geometry":{ 
      "type": "Point", 
      "coordinates": [100.0, 0.0]
    }
}
```

##### FeatureCollection

类型为"FeatureCollection"的对象必须由一个名字为"features"的成员, 用于包含多个Feature对象

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "name": "顺义区",
        "id": "110113"
       },
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [
        ]
     }
}
```

##### crs 坐标参考系统对象

GeoJSON对象的坐标参考系统（CRS）是由它的"crs"成员（指的是下面的CRS对象）来确定的。如果对象没有crs成员，那么它的父对象或者祖父对象的crs成员可能被获取作为它的crs。如果这样还没有获得crs成员，那么默认的CRS将应用到GeoJSON对象。

##### bbox边界框

GeoJSON对象可能有一个名字为"bbox的成员。bbox成员的值必须是2*n数组，这儿n是所包含几何对象的维数，并且所有坐标轴的最低值后面跟着最高者值。

---

参考文档：

https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry

https://geojson.org/


https://juejin.im/post/5d8e0eaa5188250915506b9b

http://geojson.io/

https://tools.ietf.org/html/rfc7946

