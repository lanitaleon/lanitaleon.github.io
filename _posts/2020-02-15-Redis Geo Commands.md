---
layout: post
title: RedisGeoCommands
tags:
  - redis
---

Redis可存储地理空间位置，包含经度、纬度、名称，有如下相关指令。

> geoadd, geodist, geohash, geopos, georadius, georadiusbymember

#### 1.[geoadd](http://www.redis.cn/commands/geoadd.html)

指令：

```shell
GEOADD key longitude latitude member [longitude latitude member ...]
```

添加位置，一个key可对应一个或多个位置信息。

返回值为添加位置的个数。

实例：

```shell
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2
```

#### 2.[geodist](http://www.redis.cn/commands/geodist.html)

指令：

```shell
GEODIST key member1 member2 [unit]
```

unit默认米，可选单位如下：

- m，米
- km，千米
- mi，英里
- ft，英尺

返回给定两个位置之间的距离，计算时假设地球为完美球形，极限误差0.5%。

返回值为双精度浮点数，位置不存在返回空。

实例：

```shell
redis> GEODIST Sicily Palermo Catania
"166274.15156960039"
redis> GEODIST Sicily Palermo Catania km
"166.27415156960038"
redis> GEODIST Sicily Palermo Catania mi
"103.31822459492736"
redis> GEODIST Sicily Foo Bar
(nil)
```

#### 3.[geohash](http://www.redis.cn/commands/geohash.html)

指令：

```shell
GEOHASH key member [member ...]
```

返回一个或多个位置元素的GeoHash表示。

GeoHash是一种地址编码方法，将经纬度信息编码成一个字符串，便于范围计算，字符串长度与精确度成正比。

返回值是11位字符串。

实例：

```shell
redis> GEOHASH Sicily Palermo Catania
1) "sqc8b49rny0"
2) "sqdtr74hyu0"
```

#### 4.[geopos](http://www.redis.cn/commands/geopos.html)

指令：

```shell
GEOPOS key member [member ...]
```

返回key+member对应的所有位置包含的经纬度信息。

返回值为数组，每项的第一个元素是经度，第二个元素是纬度。

member不存在时返回空。

实例：

```shell
redis> GEOPOS Sicily Palermo Catania NonExisting
1) 1) "13.361389338970184"
   2) "38.115556395496299"
2) 1) "15.087267458438873"
   2) "37.50266842333162"
3) (nil)
```

#### 5.[georadius](http://www.redis.cn/commands/georadius.html)

指令：

```shell
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
```

在key对应的位置中查询以给定经纬度为中心，与中心距离不超过指定最大距离的所有位置名称。

范围可选单位如下：

- m，米
- km，千米
- mi，英里
- ft，英尺

返回值默认为位置名称（member），设置如下选项可返回额外信息：

- withdist

位置元素与中心的距离，单位与范围单位一致。

- withcoord

位置元素的经度和纬度。

- withhash

52位有效符号整数，位置元素经过原始 geohash 编码的有序集合分值。

返回值默认不排序，设置如下参数可进行排序：

- asc

根据中心位置由近到远。

- desc

根据中心位置由远到近。

返回值默认返回所有匹配的位置元素，设置`COUNT<count>`可获取前N个元素。

以上选项参数可叠加，在返回值中的顺序为：

1.距离

2.geohash

3.经度和纬度

实例：

```shell
redis> GEORADIUS Sicily 15 37 200 km WITHDIST
1) 1) "Palermo"
   2) "190.4424"
2) 1) "Catania"
   2) "56.4413"
redis> GEORADIUS Sicily 15 37 200 km WITHCOORD
1) 1) "Palermo"
   2) 1) "13.361389338970184"
      2) "38.115556395496299"
2) 1) "Catania"
   2) 1) "15.087267458438873"
      2) "37.50266842333162"
redis> GEORADIUS Sicily 15 37 200 km WITHDIST WITHCOORD
1) 1) "Palermo"
   2) "190.4424"
   3) 1) "13.361389338970184"
      2) "38.115556395496299"
2) 1) "Catania"
   2) "56.4413"
   3) 1) "15.087267458438873"
      2) "37.50266842333162"
redis> GEORADIUS Sicily 15 37 200 km WITHDIST WITHCOORD WITHHASH
1) 1) "Palermo"
   2) "190.4424"
   3) "3479099956230698"
   4) 1) "13.36138933897018433"
      2) "38.11555639549629859"
2) 1) "Catania"
   2) "56.4413"
   3) "3479447370796909"
   4) 1) "15.08726745843887329"
      2) "37.50266842333162032"
```

#### 6.[georadiusbymember](http://www.redis.cn/commands/georadiusbymember.html)

指令：

```shell
GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
```

与georadius相同，查询指定范围内的元素，中心点由给定的位置元素名称（member）决定。

实例：

```shell
redis> GEOADD Sicily 13.583333 37.316667 "Agrigento"
(integer) 1
redis> GEORADIUSBYMEMBER Sicily Agrigento 100 km
1) "Agrigento"
2) "Palermo"
```

