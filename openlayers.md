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

##### map
这个map不是里面的Map，而是初始化后的map，map = new ol.Map()；在openlayers3 的API中，有很多方法，可以通过方法去操作layers，view，interactions，controls。
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

###### view
视图，View对象表示地图的简单2D视图。这是用于更改地图的中心，分辨率和旋转的对象。
    一个视图是由三种状态确定：center，resolution，zoom和rotation。每个状态具有相应的获取和设置。
    center：初始坐标，比如：成都的位置 [104.06, 30.67],这样地图初始化会以成都为中心展开(projection:'EPSG:4326')。
    projection：一个视图有一个projection。投影确定中心的坐标系，其单位确定分辨率的单位（每像素的投影单位）。
        projection有两种值，默认投影是球形墨卡托（EPSG：3857），是地图的xy坐标，另一种就是经纬度坐标，（EPSG:4326）是WGS84的一种。
        通过一个方法可以进行EPSG:3857与EPSG:4326转化。
        经纬度转至xy(EPSG:4326->EPSG:3857)
        `ol.proj.transform(coordinates,'EPSG:4326','EPSG:3857')`
        xy转至经纬度(EPSG:3857->EPSG:4326)
        `ol.proj.transform(coordinates,'EPSF:3857','EPSG:4326')`
    zoom:计算视图初始分辨率的缩放级别。至于resolution初始分辨率，在其未声明时，zoom起作用。对应的它也有maxZoom，minZoom。
    rotation:初始旋转度，默认是0，（顺时针正向旋转，0表示北向）,弧度制
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
view.getZoom()  // 获取缩放级别;
view.getCenter() // 获取初始中心坐标
view.setCenter([20,30])  // 设置初始中心[20,30]，也可以用ol.proj.transform
view.constrainRotation(Math.PI/2,5) //......
```

###### layers
层，是一个数组，地图上所有的交互以及图片都是通过一层一层叠加进去的。



