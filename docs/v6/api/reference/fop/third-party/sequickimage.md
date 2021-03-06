---
layout: docs
title: 大图极速浏览服务
order: 300
---

<a id="sequickimage"></a>

# 大图极速浏览服务（sequickimage）

<a id="sequickimage-description"></a>

## 描述

大图极速浏览服务(`sequickimage`)是巨大高清图片的快速浏览服务，支持在Web与移动终端上秒级流畅浏览，支持图像大小超过TB级。此外，支持高清图片动态生成瓦楞图片，无需切图，直接支持传统瓦楞金字塔调用方式。

本服务是由`苏州超擎图形软件科技发展有限公司`（以下简称`超擎`）基于七牛云提供的专业级图像服务。启用服务后，您存储在七牛云空间的文件将在您主动请求的情况下被提供给`超擎`以供其计算使用。服务价格请您参考具体的价格表及计费举例，您使用本服务产生的费用由七牛代收。启用服务则表示您知晓并同意以上内容。

<a id="sequickimage-open"></a>

## 如何开启

进入`https://portal.qiniu.com/service/market`, 找到看大图云服务点击开始使用。

<a id="sequickimage-request"></a>

## 请求

<a id="sequickimage-convert"></a>

### 请求语法

``` 
GET <ImageDownloadURI>?sequickimage/convert HTTP/1.1
Host: <ImageDownloadHost>
```

**注意：**当您下载私有空间的资源时，`ImageDownloadURI`的生成方法请参考七牛的[下载凭证](http://developer.qiniu.com/docs/v6/api/reference/security/download-token.html)。

**示例：**资源为`http://78re52.com1.z0.glb.clouddn.com/resource/gogopher.jpg`，处理命令为`sequickimage/convert`。

``` 
#构造下载URL
DownloadUrl = 'http://78re52.com1.z0.glb.clouddn.com/resource/gogopher.jpg?sequickimage/convert'
……
#最后得到
RealDownloadUrl = 'http://78re52.com1.z0.glb.clouddn.com/resource/gogopher.jpg?sequickimage/convert&e=×××&token=MY_ACCESS_KEY:×××'
```

##### 请求头部

| 参数   | 必填   | 说明                                       |
| :--- | :--- | :--------------------------------------- |
| Host | 是    | 下载服务器域名，可为七牛三级域名或自定义二级域名，参考[七牛自定义域名绑定流程](http://kb.qiniu.com/53a48154) |

##### 支持转换的图片格式

``` 
JPG、TIF、TIFF、BMP等常见格式。
```

因为考虑到转换后的格式不能直接在浏览器显示，为达到更好的显示效果，需要使用异步处理来进行图片的转换，以[触发持久化处理](http://developer.qiniu.com/docs/v6/api/reference/fop/pfop/pfop.html)形式，这一部分建议使用七牛提供的各种语言的SDK来处理：

``` 
POST /pfop/ HTTP/1.1
Host: api.qiniu.com  
Content-Type: application/x-www-form-urlencoded  
Authorization: QBox <AccessToken>  

bucket=ztest&key=sequickimage_test.jpg&fops=sequickimage/convert&notifyURL=http%3A%2F%2Ffake.com%2Fqiniu%2Fnotify
```

备注： 

+ 转换后的文件是二进制文件，无法直接浏览
+ 持久化保存的方法请参考 [触发持久化处理](http://developer.qiniu.com/docs/v6/api/reference/fop/pfop/pfop.html)

<a id="sequickimage-getinfo"></a>

### 查看结果文件信息

在浏览器地址栏中输入转换后的文件链接+“ ?sequickimage/getinfo”，即可获取转换后的文件信息。

例如： [http://7qnbh1.com1.z0.glb.clouddn.com/ch.tif?sequickimage/getinfo](http://7qnbh1.com1.z0.glb.clouddn.com/ch.tif?sequickimage/getinfo)

##### 响应报文格式

``` 
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
    // ...sequickimage data...
}
```

##### 响应内容

返回的结果已格式化，便于阅读：

``` 
{
    "height": 9472,
    "result": 0,
    "width": 11008
}
```

<a id="sequickimage-view"></a>

### 浏览或者操作

#### 使用参数链接获取动态预览图或指定瓦片图

获取进行格式转换后的图片的`URL`地址： [http://7qnbh1.com1.z0.glb.clouddn.com/ch.tif](http://7qnbh1.com1.z0.glb.clouddn.com/ch.tif)，在地址后面加上`?sequickimage/gettile/0-0-0.5-256`。如下图所示，即可获取瓦片图。

![瓦片图](http://7qnbh1.com1.z0.glb.clouddn.com/AF48F5F4-02F7-435E-AB60-14214A0F2020.png)

#### 使用超擎的前端API来浏览

##### 在html页面中引入超擎JS库和CSS库

例如：

``` 
<link rel="stylesheet" type="text/css" href="http://61.155.169.20/qiniuimg/css/se.css"/>
<script src="http://61.155.169.20/qiniuimg/js/se.js" type="text/javascript" charset="utf-8"></script>
```



##### 调用方法createTileLayer(url,imgWidth,imgHeight)，创建img图层



| 参数        | 参数类型   | 是否必须 | 说明                                     |
| :-------- | :----- | :--- | :------------------------------------- |
| url       | String | 是    | 图片对应的“外部链接地址”+“ ?sequickimage/gettile” |
| imgWidth  | Number | 是    | 图片的宽度                                  |
| imgHeight | Number | 是    | 图片的高度                                  |

返回值： `img` 图层对象。

例：

``` javascript
var url = "http://7qnbh1.com1.z0.glb.clouddn.com/ch.tif?sequickimage/getinfo";
var tileLayer = createTileLayer(url, 9472, 11008);
```



##### 调用方法createMap(id)，创建图片对象



| 参数   | 参数类型   | 是否必须 | 说明     |
| :--- | :----- | :--- | :----- |
| id   | String | 是    | 图片容器id |

返回值：图片对象

例:

``` javascript
<div id="img"></div>

var map = createMap("img")
```



##### 调用方法initMap(map,tileLayer)，将img图层添加到map中，初始化map



| 参数        | 参数类型   | 是否必须 | 说明      |
| :-------- | :----- | :--- | :------ |
| map       | object | 是    | 图片对象    |
| tileLayer | object | 是    | img图层对象 |

例如：

``` javascript
var url = "http://7qnbh1.com1.z0.glb.clouddn.com/ch.tif?sequickimage/getinfo";
var tileLayer = createTileLayer(url, 9472, 11008);
var map = createMap("img");
initMap(map, tileLayer);
```

这里是一个 [Demo](http://7xlmcc.com1.z0.glb.clouddn.com)

<a id="sequickimage-price"></a>

## 服务价格

|           |         |
| :-------- | :------ |
| 范围（次）     | 单价（元/次） |
| 0 - 50    | 0       |
| 51 - 1000 | 1       |
| > 1000    | 0       |

说明：

* 计费只计`sequickimage/convert`的费用，`sequickimage/getinfo`和`sequickimage/gettile`不计费。

<a id="sequickimage-price-example"></a>

## 计费示例

某公司2015年10月使用超擎看大图服务，共发起3000起convert请求，则当月使超擎看大图服务产生的费用为：

总计费用：50次 * 0元/次 + (1000次 - 50次) * 1元/次 + (3000次 - 1000次) * 0元/次 = 950元

