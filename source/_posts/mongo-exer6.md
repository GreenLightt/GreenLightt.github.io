---
title: 练习 MongoDB 操作 —— 地理位置搜索 （六）
date: 2018-02-07 14:17:41
description: 摘录书中关于地理位置索引章节的知识点
tags:
- MongoDB3.x
categories:
- NoSQL
---

本文摘自 《`Mongodb` 权威指南》特殊的索引和集合一章；

`MongoDB` 支持几种类型的地理空间索引。其中最常用的是 `2dsphere` 索引（用于地球表面类型的地图）和 `2d` 索引（用于平面地图和时间连续的数据）；


`2dsphere` 允许使用 `GeoJSON` 格式（[http://geojson.org/](http://geojson.org/)）指定点、线和多边形。点可以形如 [`longitude`， `latitude`] ([经度，纬度])的两个元素的数组表示：

```
{
    "name": "New York City",
    "loc": {
        "type": "Point",
        "coordinates": [50, 2],
    }
}
```

线可以用一个由点组成的数组来表示：

```
{
    "name": "Hudson River",
    "loc": {
        "type": "Line",
        "coordinates": [[0, 1], [0, 2], [1, 2]]
    }
}
```

多边形的表示方式与线一样（都是一个由点组成的数组）， 但是 `type` 不同：

```
{
    "name": "New England",
    "loc": {
        "type": "Polygon",
        "coordinates": [[0, 1], [0, 2], [1, 2]]
    }
}
```

`loc` 字段的名字可以是任意的，但是其中的子对象是由 `GeoJson` 指定的，不能改变；

在 `ensureIndex` 中使用 `2dsphere` 选项就可以创建一个地理空间索引：

```
> db.world.ensureIndex({"loc": "2dsphere"})
```


# 地理空间查询的类型

可以使用多种不同类型的地理空间查询：交集（`intersection`）、包含（`within`）以及接近(`nearness`)。查询时，需要将希望找到的内容指定为形如 `{"$geometry": geoJsonDesc}` 的 `GeoJson` 对象。

例如，可以使用 `$geoIntersects` 操作符找出与查询位置相交的文档：

```
> var eastVillage = {
      "type": "Polygon",
      "coordinates": [[
           [-73.9917900, 40.7264100],
           [-73.9917900, 40.7321400],
           [-73.9829300, 40.7321400],
           [-73.9829300, 40.7264100],
           [-73.9917900, 40.7264100]
    ]]
  }
> db.open.street.map.find({"loc": {"$geoIntersects": {"$geometry": eastVillage}}})
```

可以使用 `$geoWithin` 查询完全包含在某个区域的文档，例如: `East Village` 有哪些餐馆？

```
> db.open.street.map.find({"loc": {"$geoWithin": {"$geometry": eastVillage}}})
```

与第一个查询不同，这次不会返回那些只是经过 `East Village` （比如街道）或者部分重叠（比如用于表示曼哈顿的多边形）的文档。

最后，可以使用 `$near` 查询附近的位置：

```
> db.open.street.map.find({"loc": {"$near": {"$geometry": {
      "type": "Point",
      "coordinates": [-73.9917900, 40.7264100]
    }
}}})
```

注意， `$near` 是唯一一个会对查询结果进行自动排序的地理空间操作符； `$near` 的返回结果是按照距离由近及远排序的。

地理位置查询有一点非常有趣：不需要地理空间索引就可以使用 `$geoIntersects` 或者  `$within` （`$near`需要使用索引）。但是，建议在用于表示地理位置的字段上建立地理空间索引，这样可以显著提高查询速度。


# 复合地理空间索引

如果有其他类型的索引，可以将地理空间索引与其他字段组合在一起使用，以便对更复杂的查询进行优化。上面提到过一种可能的查询： "`East Village`有哪些餐馆？"。如果仅仅使用地理空间索引，我们只能查找到 `East Village` 内的所有东西，但是如果要将 "`restaurants`" 或者是 "`pizza`" 单独查询出来，就需要使用其他索引中的字段了：

> db.open.street.map.ensureIndex({"tags": 1, "loc": "2dsphere"})

然后就能够很快地找到 `East Village` 内的披萨店了：

```
> db.open.street.map.find({"loc": {"$geoWithin": {"$geometry": eastVillage}}, "tags": "pizza"})
```

其他索引字段可以放在 `2dsphere` 字段前面也可以放在后面，这取决于我们希望首先使用其他索引的字段进行过滤还是首先使用位置进行过滤。应该将那个能够尽可能多的结果的字段放在前面。

# 2d 索引

对于非球面地图（游戏地图、时间连续的数据等），可以使用 `2d` 索引代替 `2dsphere`；

> db.hyrule.ensureIndex({"tile": "2d"})

`2d`索引用于扁平表面，而不是球体表面。 `2d`索引不应该用在球体表面上，否则极点附近会出现大量的扭曲变形；

文档中应该使用包含两个元素的数组表示 `2d` 索引字段。示例如下：

```
{
    "name": "Water Temple",
    "tile": [32, 32]
}
```

`2d` 索引只能对点进行索引。可以保存一个由点组成的数组，但是它只会被保存为由点组成的数组，不会被当成线。特别是对于  `$geoWithin` 查询来说，这是一项重要的区别。如果将街道保存为由点组成的数组，那么如果其中的某个点位于给定的形状之内，这个文档就会与 `$geoWithin` 相匹配。但是，由这些点组成的线并不一定会完全包含在这个形状之内。

默认情况下，地理空间索引是假设你的值都介于 -180 ~ 180。可以根据需要在 `ensureIndex` 中设置更大或更小的索引边界值：

```
> db.star.trek.ensureIndex({"light-years": "2d"}, {"min": -1000, "max": 1000})
```

这会创建一个 2000 * 2000 大小的空间索引。

使用 `2d` 索引进行查询比使用 `2dsphere` 要简单许多。可以直接使用 `$near` 或者 `$geoWithin`， 而不必带有 `$geometry` 子对象。可以直接指定坐标：

```
> dy.hyrule.find({"tile": {"$near": [20, 21]}})
```

这样会返回 `hyrule` 集合内的全部文档，按照距离（20，21）这个点的距离排序。如果没有指定文档数量限制，默认最多返回 100 个文档。如果不需要这么多的结果，应该根据需要设置返回文档的数量以节省服务器资源。例如，下面的代码只会返回距离（20，21）最近的 10 个文档：

```
> dy.hyrule.find({"tile": {"$near": [20, 21]}}).limit(10)
```

`$geoWithin` 可以查询出某个形状（矩形、圆形或者是多边形）范围内的所有文档。如果要使用矩形，可以指定 `$box` 选项

```
> dy.hyrule.find({"tile": {"$geoWithin": {"$box": [[10, 20], [15, 30]]}}})
```

`$box` 接受一个两元素的数组：第一个元素指定左下角的坐标，第二个元素指定右上角的坐标。

类似地，可以使用 `$center` 选项返回圆形范围内的所有文档，这个选项也是接受一个两元素数组作为参数：第一个元素是一个点，用于指定圆心；第二个参数用于指定半径：

```
> db.hyrule.find({"tile": {"$geoWithin": {"$center": [[12, 25], 5]}}})
```

还可以使用多个点组成的数组来指定多边形：

```
> db.hyrule.find({"tile": {"$geoWithin": {"$polygon": [[0, 20], [10, 0], [-10, 0]]}}})
```

这个例子会查询出包含给定三角形内的点的所有文档。列表中的最后一个点会被连接到第一个点，以便组成多边形；

# 参考阅读

- [MongoDB 官方文档 geo 操作符](https://docs.mongodb.com/manual/reference/operator/query-geospatial/)
- [地理空间数据格式——GeoJSON](http://blog.csdn.net/yaoxiaochuang/article/details/53117379)




