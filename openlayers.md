#### openlayers 知识

------------------


```javascript
var map = new ol.Map({
    target:'map',
    layers:[],
    view:views,
    interactions:interactions,
    controls:controls,
})
```

map是openlayers的`核心`组件，所有的操作方法以及渲染都是挂载到map上面。创建的map对象需要传一个对象进去，最常见的就有以下几个属性，

    * target --> 地图的挂载元素，就是在html元素里声明的id
    * layers --> 层，是个数组，可以放很多层，如果未定义，则将呈现没有图层的地图，图层是按照提供的顺序渲染的，因此，如果您希望（例如）矢量图层显示在图块图层的顶部，则它必须位于图块图层之后。
    * view  --> map的视图，
    * interactions --> 交互，比如鼠标事件导致放大，缩小，旋转之类的，默认都是true
    * controls --> 控件，类似与按钮，有放大，缩小，全屏等的按钮 

最简单的显示地图：

```javascript
document.body.innerHTML = `<div id="map" style="height:400px;width:100%"></div>`;

var map = new ol.Map({
    target:'map',
    layers:[new ol.layers.Tile({
        source: new ol.source.OSM()
    })],
    view: new ol.View({
        center:[0,0],
        projection: 'EPSG:4326',
        zoom:10,
    }),
    interactions: ol.interaction.defaults({
        doubleClickZoom:false,
        altShiftDragRotate:false
    }),
    controls: ol.control.defaults({
        attribution: false,
    })
})

```

### map
这个map不是里面的Map，而是初始化后的map，`map = new ol.Map()`；在openlayers3 的API中，有很多方法，可以通过方法去操作layers，view，interactions，controls。
打印map,可以发现map里面有很多属性，可以简单知道map上有多少个层，target是谁，size是多少等等，也可以通过API方法去获取
```javascript
var target = map.getTarget();    // 获取 target的值  -- map
var size = map.getSize();  // 获取可视区域的宽高。
```
同样的，也可以在map上设置
```javascript
map.addControl(new ol.control.FullScreen()); // 添加全屏控件
map.addLayers( new ol.layer.Vector({
    source:new ol.Source.Vector()
}))  // 添加一个矢量图层

```
*map里有一个函数方法，可以通过点击某个图层或某个点而获取到当前的层和信息。
`forEachFeatureAtPixel`(pixel, callback, opt_options) 
params: pixel：坐标
        callback：回调函数，将使用两个参数调用回调，第一个是feature，第二个是layers
        opt_options：可选
```javascript
map.on('click',function(e){
    var feature = map.forEachFeatureAtPixel(e.pixel,function(f){
        return f;
    })
    console.log(feature);   // ol.Feature
})
```

#### view
`视图`，View对象表示地图的简单2D视图。这是用于更改地图的中心，分辨率和旋转的对象。
    一个视图是由三种状态确定：center，resolution，zoom和rotation。每个状态具有相应的获取和设置。
* *center*：初始坐标，比如：成都的位置 [104.06, 30.67],这样地图初始化会以成都为中心展开(projection:'EPSG:4326')。
* *projection*：一个视图有一个projection。投影确定中心的坐标系，其单位确定分辨率的单位（每像素的投影单位）。
    projection有两种值，默认投影是球形墨卡托（EPSG：3857），是地图的xy坐标，另一种就是经纬度坐标，（EPSG:4326）是WGS84的一种。
    通过一个方法可以进行EPSG:3857与EPSG:4326转化。
    经纬度转至xy(EPSG:4326->EPSG:3857)
    `ol.proj.transform(coordinates,'EPSG:4326','EPSG:3857')`
    xy转至经纬度(EPSG:3857->EPSG:4326)
    `ol.proj.transform(coordinates,'EPSF:3857','EPSG:4326')`
* *zoom*:计算视图初始分辨率的缩放级别。至于resolution初始分辨率，在其未声明时，zoom起作用。对应的它也有maxZoom，minZoom。
* *rotation*:初始旋转度，默认是0，（顺时针正向旋转，0表示北向）,弧度制
```javascript
var view = new ol.View({
    center:[104.06,30.67],
    projection:'EPSG:4326',
    // center: ol.proj.transform([104.06,30,67],'EPSG:4326','EPSG:3857');    // 这个和上面的结合体。
    zoom:10,
    rotation:Math.PI/10,
})
```
视图对应的也有get和set系列的方法，用于获取和设置zoom，center，rotation等等，除此之外，还有约束方法，constrainRotation(),约束旋转度，API上有详细解释。
```javascript
map.getView()   // 就是当前视图，view
view.getZoom()  // 获取缩放级别;
view.getCenter() // 获取初始中心坐标
view.setCenter([20,30])  // 设置初始中心[20,30]，也可以用ol.proj.transform
view.constrainRotation(2,5) // 获取此视图的约束旋转
```

#### layers
层，是一个对象数组，将那些与数据显示方式相关的属性组合在一起，形成一个层，openlayer就是由一个一个的层形成的，包括地图，瓦片地图，图片，图形都是由layer形成而呈现在map上的
    简单的来说，layer可以是一个地图，也可以是一个图形，也可以是个图片，因为是个数组，可以互相叠加的，[map,image,shape]，在相同位置的情况下，后者会覆盖前者。
    
![layer](https://github.com/sqh17/openLayers-Learn/tree/master/images/ol_layer_Base.png)

根据这个图片可知，layers有三种形式
* **ol.layer.Tile** 瓦片，瓦片地图源于一种大地图解决方案，针对一整块非常大的地图进行切片，分成很多相同大小的小块地图，在用户访问的时候，再一块一块小地图加载，拼接在一起，从而还原成一整块大的地图。这样做的优点在于，用户在同一时间，同一个可见视图内，只能看到地图的一部分，而不是全部。提高用户体验。通常用这个作为底图。
* **ol.layer.Image** 对应的是一整张图，而不像瓦片那样很多张图，从而无需切片，也可以加载一些地图，适用于一些小场景地图
* **ol.layer.Vector** 矢量，使用直线和曲线来描述图形，这些图形的元素是一些点、线、矩形、多边形、圆和弧线等等，它们都是通过数学公式计算获得的。由于矢量图形可通过公式计算获得，所以矢量图形文件体积一般较小。矢量图形最大的优点是无论放大、缩小或旋转等不会失真。

每个形式都需要一个source，数据源，所以每一个形式都对应一个数据源，Source和Layer是一对一的关系，有一个Source，必然需要一个Layer，然后把这个Layer添加到Map上，就可以显示出来了。

##### ol.layer.Tile
对于瓦片数据源，也有很多属性，
* opacity，改变透明度，默认是1
* visible，可视，默认是true
* zIndex，图层渲染的z-index。在渲染时，将按照Z-index然后按位置对图层进行排序
* `source` 有很多形式，
* **在线服务的Source**
    * *ol.source.OSM*
    * *ol.source.BingMaps*
    * *ol.source.TileImage*
* **支持协议标准的Source**，包括ol.source.TileArcGISRest，ol.source.TileWMS，ol.source.WMTS，ol.source.UTFGrid，ol.source.TileJSON。如果要使用它们，首先你得先学习对应的协议，之后必须找到支持这些协议的服务器来提供数据源，这些服务器可以是地图服务提供商提供的，也可以是自己搭建的服务器，关键是得支持这些协议。
* **ol.source.XYZ**，而且现在很多地图服务（在线的，或者自己搭建的服务器）都支持xyz方式的请求。国内在线的地图服务，高德，天地图等，都可以通过这种方式加载的

```javascript
var OSMtile = new ol.layer.Tile({ 
    source:new ol.source.OSM()  
})
var qqTile = new ol.layer.Tile({
    source: new ol.source.XYZ({
        url: 'http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineCommunity/MapServer/tile/{z}/{y}/{x}'
    }),
})
map.addLayer(OSMtile);  // 添加open street Map作为底图
map.addLayer(qqTile)  // 添加腾讯地图
```
map.addLayer()这个方法就相当于初始化中的layers的数组中添加一样,如下代码，这俩是等价的，只不过后者更灵活些。

```javascript
var map = new ol.Map({
    layers:[OSMtile,qqTile]
})  // 等价于下面
map.addLayer(OSMtile); 
map.addLayer(qqTile)
```
ps！当添加地图作为底图时，会发现腾讯底图覆盖率OSM地图，是因为layer是一个数组，遵遁先来先渲染，后来居上的原则。
`layer也有get/set方法，用来设置或获取属性，其实通过API就可以知道响应方法去操作。`
##### ol.layer.Image
关于静态图片层，运用的不多，一般作为写死的，不经常换的实例中。
同样，ol.layer.Image也有数据源，一般就是ol.source.ImageStatic，ol.source.ImageCanvas等等。
```javascript  
var center = ol.proj.transform([104.06667, 30.66667], 'EPSG:4326', 'EPSG:3857');
// 计算静态图映射到地图上的范围，图片像素为 550*344，保持比例的情况下，把分辨率放大一些
var extent = [center[0]- 550*1000/2, center[1]-344*1000/2, center[0]+550*1000/2, center[1]+344*1000/2];
var imageStatic = new ol.layer.Image({
    source: new ol.source.ImageStatic({
        url: '..／pandaBase.jpg', // 静态图
        imageExtent: extent     // 映射到地图的范围
    })
})
map.addLayer(imageStatic);
```

##### ol.layer.Vector
    矢量图层，是用来渲染矢量数据的图层类型，在OpenLayers里，它是可以定制的，可以控制它的透明度，颜色，以及加载在上面的要素形状等。常用于从数据库中请求数据，接受数据，并将接收的数据解析成图层上的信息。比如将鼠标移动到中国，相应的区域会以红色高亮显示出来，高亮便是矢量图层的行为
既然是个矢量图，可以改变大小，颜色，以及形状。包含source数据源类，style类设置样式，以及其他的zIndex，opacity等等。
`Feature类是Vector类用来在地图上展示几何对象，是Vector图层类一个属性。这个属性是个*要素数组*。 `
###### source
对应的数据源ol.source.Vector也是个对象。最常见属性的是`features`和`format`以及`url`。通常url和format一块，url按理传来的是geojson格式的矢量图，需要format格式化一下。最常用还是features。
**ol.Feature**
* feature 具有几何和其他属性属性的地理要素的矢量对象，类似于GeoJSON等矢量文件格式中的要素。
* Geometry类是feature对象的基本组成部分，Vector类采用Geometry类来存储一个要素的几何信息.
* feature类有两个部分，Geometry对象和attributes属性,attributes包含要素相关的数据。比如：{type:'circle'},通过getProperties().attributes去获取。
**geometry**
* new ol.geom.Point(coordinate)  点
* new ol.geom.LineString([[start],[end]]) 线
###### style
    一个style对象，包含Style类，有7个属性：
* geometry 返回要为此样式渲染的几何的要素属性或几何或函数
* fill 填充样式。
* image 图像样式
    * icon : new ol.style.Icon()
        * anchor :**[0.5，0.5]**锚点，
        * color
        * src :图标源
        * ...
    * circle: new ol.style.Circle()
        * fill :new ol.style.Fill()
        * radius : 半径
        * stroke :new ol.style.Stroke()
    * regularShape :new ol.style.RegularShape()
        * fill :new ol.style.Fill()
        * points : 点数，几个点
        * radius : 半径
        * radius1 : 外半径
        * radius2 : 内半径
        * angle  **0** 形状的角度以弧度表示。值为0将使其中一个形状的点朝上。
        * stroke :new ol.style.Stroke()
        * rotation : **0** 旋转度
        * ...
* renderer 自定义渲染器。配置时fill，stroke和image将被忽略，并且将为每个几何体的每个渲染帧调用提供的函数。
* stroke 边线样式
    * color 颜色
    * lineCap :butt, **round**, square 线帽
    * lineJoin :bevel, **round**, miter. 线条连接形状
    * width 宽度
* text 文字样式
    * font :'**10px sans-serif**'
    * text
    * textAlign left'，'right'，'**center**'，'end' start'
    * fill :new ol.style.Fill()
    * stroke :new ol.style.Stroke()
    * backgroundFill
    * padding
    *
* zIndex 层级
// set*()在重新呈现使用该样式的要素或图层之前，通过方法对样式或其子项所做的任何更改都不会生效
```javascript
// 最简单的呈现方式，复杂的请参考demo
var layer = new ol.layer.Vector({
    source: new ol.source.Vector([
        new ol.Feature({
            geometry: new ol.geom.Point(ol.proj.transform([104, 30], 'EPSG:4326', 'EPSG:3857')),
        })]),
    style: new ol.style.Style({
        image: new ol.style.Circle({
            radius: 30,
            fill: new ol.style.Fill({
                color: 'red'
            })
        })
    }),
    opacity:0.5
})
map.addLayer(layer)
```
