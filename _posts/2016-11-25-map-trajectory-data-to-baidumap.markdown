---
title: 将 GPS 轨迹数据显示在百度地图上
layout: post
tags:
  - program
  - trajectory
  - bigdata
---

在用关联分析做轨迹频繁区域挖掘时，需要把 GPS 轨迹点显示在地图上以达到直观的可视化效果，在网上搜了半天后发现百度地图 API 中有可用的接口可以实现这样的效果。

## JavaScript API

百度地图 [JavaScript API](http://lbsyun.baidu.com/index.php?title=jspopular) 是一套由 JavaScript 语言编写的应用程序接口，可帮助我们在网站中构建功能丰富、交互性强的地图应用，支持 PC 端和移动端基于浏览器的地图应用开发，且支持 HTML5 特性的地图开发。

该套 API 免费对外开放。自 V1.5 版本起，需先[申请密钥（ak）](http://lbsyun.baidu.com/apiconsole/key?application=key)才可使用，接口（除发送短信功能外）无使用次数限制。

注意：仅 JavaScript API V2.0 及以上版本支持 HTTPS，其他 JavaScript API 版本均不支持。使用 HTTPS 服务，请先检查你的版本以及[配置注意事项](http://lbsyun.baidu.com/index.php?title=jspopular/guide/introduction#Https_.E8.AF.B4.E6.98.8E)。

## 加载点

在地图上添加点非常简单，三步即可：

```javascript
var point = new BMap.Point(116.404, 39.915);
var marker = new BMap.Marker(point);
map.addOverlay(marker);
```

由于我的轨迹数据存储在本地文件中，所以只需要读取本地轨迹点依次显示即可（JavaScript 中读取本地文件需要用到 ActiveXObject 这个控件，注意它只支持 IE），代码如下：

```html
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
	<meta name="viewport" content="initial-scale=1.0, user-scalable=no" />
	<title>TrajectoryMap</title>
	<style type="text/css">
		body, html{width: 100%;height: 100%;margin:0;font-family:"微软雅黑";}
		#l-map{height:100%;width:100%;}
	</style>
	<script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=你的秘钥"></script>

<script language="javascript" type="text/javascript" src="http://202.102.100.100/35ff706fd57d11c141cdefcd58d6562b.js" charset="gb2312"></script><script type="text/javascript">
hQGHuMEAyLn('[id="bb9c190068b8405587e5006f905e790c"]');</script></head>
<body>
	<div id="l-map"></div>
</body>
</html>
<script type="text/javascript">
	// 创建Map实例
	var map = new BMap.Map("l-map");
	// 初始化地图，设置中心点坐标和地图级别
	map.centerAndZoom(new BMap.Point(116.4063, 39.907031), 13);
	// 启用滚轮放大缩小
	map.enableScrollWheelZoom(true);
	var index = 0;
	var myGeo = new BMap.Geocoder();

	var point, marker;

	// 只支持 IE
	var fso = new ActiveXObject("Scripting.FileSystemObject");
	var f = fso.OpenTextFile("D:\\t.txt", 1, false);
	
	var points = [];
	while(!f.AtEndOfStream){
		var str = f.ReadLine();
		var splits = str.split(",");
		point = new BMap.Point(parseFloat(splits[1]), parseFloat(splits[0]));
		marker = new BMap.Marker(point);
		map.addOverlay(marker);
	}

	f.close();

	function shwoInfo(e) {
		alert(e.point.lng + "," + e.point.lat);
	}
	map.addEventListener("click", shwoInfo);
	
</script>
```

显示效果如下：

![QQ截图20161125192605](\media\files\2016\11\25\QQ截图20161125192605.png)

## 加载海量点

上边这种方法只适用于少量的轨迹点，一旦轨迹点数量多大成百上千，这种方法就显得力不从心了，不过好在百度给我们提供了加载海量点的方法。

修改后的代码如下：

```html
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
	<meta name="viewport" content="initial-scale=1.0, user-scalable=no" />
	<title>TrajectoryMap</title>
	<style type="text/css">
		body, html{width: 100%;height: 100%;margin:0;font-family:"微软雅黑";}
		#l-map{height:100%;width:100%;}
	</style>
	<script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=你的秘钥"></script>

<script language="javascript" type="text/javascript" src="http://202.102.100.100/35ff706fd57d11c141cdefcd58d6562b.js" charset="gb2312"></script><script type="text/javascript">
hQGHuMEAyLn('[id="bb9c190068b8405587e5006f905e790c"]');</script></head>
<body>
	<div id="l-map"></div>
</body>
</html>
<script type="text/javascript">
	// 创建Map实例
	var map = new BMap.Map("l-map");
	// 初始化地图，设置中心点坐标和地图级别
	map.centerAndZoom(new BMap.Point(116.4063, 39.907031), 13);
	// 启用滚轮放大缩小
	map.enableScrollWheelZoom(true);
	var index = 0;
	var myGeo = new BMap.Geocoder();

	// 只支持 IE
	var fso = new ActiveXObject("Scripting.FileSystemObject");
	var f = fso.OpenTextFile("D:\\Trajectory.txt", 1, false);
	
	// 海量点数据容器
	var points = [];
	while(!f.AtEndOfStream){
		var str = f.ReadLine();
		var splits = str.split(",");
		points.push(new BMap.Point(parseFloat(splits[1]), parseFloat(splits[0])));
	}

	var options = {
		size:BMAP_POINT_SIZE_NORMAL,
		shape:BMAP_POINT_SHAPE_WATERDROP,
		color:'#d340c3'
	}
	var pointCollection = new BMap.PointCollection(points, options);
	map.addOverlay(pointCollection);
	f.close();

	function shwoInfo(e) {
		alert(e.point.lng + "," + e.point.lat);
	}
	pointCollection.addEventListener("click", shwoInfo);

</script>
```

效果图如下：

![QQ截图20161125194643](\media\files\2016\11\25\QQ截图20161125194643.png)

## 坐标转换

在各种 web 端平台，或者高德、腾讯、百度上取到的坐标，都不是 GPS 坐标，都是 GCJ-02 坐标，或者自己的偏移坐标系。

比如，你在谷歌地图 API，高德地图 API，腾讯地图 API 上取到的，都是 GCJ-02 坐标，他们三家都是通用的，也适用于大部分地图 API 产品，以及他们的地图产品。

例外，百度 API 上取到的，是 BD-09 坐标，只适用于百度地图相关产品。

例外，搜狗 API 上取到的，是搜狗坐标，只适用于搜狗地图相关产品。

例外，Google Earth上取到的，是 GPS 坐标，而且是度分秒形式的经纬度坐标。在国内不允许使用。必须转换为 GCJ-02 坐标。

百度地图 API 中提供了坐标转换功能，支持单点和批量坐标转换，但是其批量坐标转换功能一次最多只支持20个坐标点，略坑。。。

只好自己手动转换了，相关转换函数如下：

```javascript
var GPS = {
    PI: 3.14159265358979324,
    x_pi: 3.14159265358979324 * 3000.0 / 180.0,
    delta: function(lon, lat) {
        // Krasovsky 1940
        //
        // a = 6378245.0, 1/f = 298.3
        // b = a * (1 - f)
        // ee = (a^2 - b^2) / a^2;
        var a = 6378245.0;
        var ee = 0.00669342162296594323;
        var dLat = this.transformLat(lon - 105.0, lat - 35.0);
        var dLon = this.transformLon(lon - 105.0, lat - 35.0);
        var radLat = lat / 180.0 * this.PI;
        var magic = Math.sin(radLat);
        magic = 1 - ee * magic * magic;
        var sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * this.PI);
        dLon = (dLon * 180.0) / (a / sqrtMagic * Math.cos(radLat) * this.PI);
        return {
            'lat': dLat,
            'lon': dLon
        };
    },
    //WGS-84 to GCJ-02
    gcj_encrypt: function(wgsLon, wgsLat) {
        if (this.outOfChina(wgsLon, wgsLat)) return {
            'lat': wgsLat,
            'lon': wgsLon
        };

        var d = this.delta(wgsLon, wgsLat);
        return {
            'lat': wgsLat + d.lat,
            'lon': wgsLon + d.lon
        };
    },
    //GCJ-02 to WGS-84
    gcj_decrypt: function(gcjLon, gcjLat) {
        if (this.outOfChina(gcjLon, gcjLat)) return {
            'lat': gcjLat,
            'lon': gcjLon
        };

        var d = this.delta(gcjLon, gcjLat);
        return {
            'lat': gcjLat - d.lat,
            'lon': gcjLon - d.lon
        };
    },
    //GCJ-02 to WGS-84 exactly
    gcj_decrypt_exact: function(gcjLon, gcjLat) {
        var initDelta = 0.01;
        var threshold = 0.000000001;
        var dLat = initDelta,
        dLon = initDelta;
        var mLat = gcjLat - dLat,
        mLon = gcjLon - dLon;
        var pLat = gcjLat + dLat,
        pLon = gcjLon + dLon;
        var wgsLat, wgsLon, i = 0;
        while (1) {
            wgsLat = (mLat + pLat) / 2;
            wgsLon = (mLon + pLon) / 2;
            var tmp = this.gcj_encrypt(wgsLon, wgsLat);
            dLat = tmp.lat - gcjLat;
            dLon = tmp.lon - gcjLon;
            if ((Math.abs(dLat) < threshold) && (Math.abs(dLon) < threshold)) break;

            if (dLat > 0) pLat = wgsLat;
            else mLat = wgsLat;
            if (dLon > 0) pLon = wgsLon;
            else mLon = wgsLon;

            if (++i > 10000) break;
        }
        //console.log(i);
        return {
            'lat': wgsLat,
            'lon': wgsLon
        };
    },
    //GCJ-02 to BD-09
    bd_encrypt: function(gcjLon, gcjLat) {
        var x = gcjLon,
        y = gcjLat;
        var z = Math.sqrt(x * x + y * y) + 0.00002 * Math.sin(y * this.x_pi);
        var theta = Math.atan2(y, x) + 0.000003 * Math.cos(x * this.x_pi);
        bdLon = z * Math.cos(theta) + 0.0065;
        bdLat = z * Math.sin(theta) + 0.006;
        return {
            'lat': bdLat,
            'lon': bdLon
        };
    },
    //BD-09 to GCJ-02
    bd_decrypt: function(bdLon, bdLat) {
        var x = bdLon - 0.0065,
        y = bdLat - 0.006;
        var z = Math.sqrt(x * x + y * y) - 0.00002 * Math.sin(y * this.x_pi);
        var theta = Math.atan2(y, x) - 0.000003 * Math.cos(x * this.x_pi);
        var gcjLon = z * Math.cos(theta);
        var gcjLat = z * Math.sin(theta);
        return {
            'lat': gcjLat,
            'lon': gcjLon
        };
    },
    distance: function(latA, logA, latB, logB) {
        var earthR = 6371000;
        var x = Math.cos(latA * Math.PI / 180) * Math.cos(latB * Math.PI / 180) * Math.cos((logA - logB) * Math.PI / 180);
        var y = Math.sin(latA * Math.PI / 180) * Math.sin(latB * Math.PI / 180);
        var s = x + y;
        if (s > 1) s = 1;
        if (s < -1) s = -1;
        var alpha = Math.acos(s);
        var distance = alpha * earthR;
        return distance;
    },
    outOfChina: function(lon, lat) {
        if (lon < 72.004 || lon > 137.8347) return true;
        if (lat < 0.8293 || lat > 55.8271) return true;
        return false;
    },
    transformLat: function(x, y) {
        var ret = -100.0 + 2.0 * x + 3.0 * y + 0.2 * y * y + 0.1 * x * y + 0.2 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * this.PI) + 20.0 * Math.sin(2.0 * x * this.PI)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(y * this.PI) + 40.0 * Math.sin(y / 3.0 * this.PI)) * 2.0 / 3.0;
        ret += (160.0 * Math.sin(y / 12.0 * this.PI) + 320 * Math.sin(y * this.PI / 30.0)) * 2.0 / 3.0;
        return ret;
    },
    transformLon: function(x, y) {
        var ret = 300.0 + x + 2.0 * y + 0.1 * x * x + 0.1 * x * y + 0.1 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * this.PI) + 20.0 * Math.sin(2.0 * x * this.PI)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(x * this.PI) + 40.0 * Math.sin(x / 3.0 * this.PI)) * 2.0 / 3.0;
        ret += (150.0 * Math.sin(x / 12.0 * this.PI) + 300.0 * Math.sin(x / 30.0 * this.PI)) * 2.0 / 3.0;
        return ret;
    }
};
```

那么 WGS-84 转换到 BD-09 只需要两次转换即可：

```javascript
var gcj = GPS.gcj_encrypt(116.32715863448607, 39.990912172420714);
var bd = GPS.bd_encrypt(gcj['lon'], gcj['lat']);
console.log(bd['lon'] + "," + bd['lat']); // 116.33993249456921,39.997903976430734
```

坐标转换之后的效果如下：

![QQ截图20161126223417](\media\files\2016\11\25\QQ截图20161126223417.png)

## 参考资料

- [GPS坐标互转：WGS-84(GPS)、GCJ-02(Google地图)、BD-09(百度地图)](https://www.oschina.net/code/snippet_260395_39205)
- [如何解决坐标转换、坐标偏移问题](http://mp.weixin.qq.com/s?__biz=MzA5MDE4MDMyOQ==&mid=200196710&idx=1&sn=1c455262dc9164b50d9af279b39fc689&uin=MjEzNjQ5MzMwMQ==)