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
* Geometry类是feature对象的基本组成部分，Vector类采用Geometry类来存储一个要素的几何信息.通过`feature名.getGeometry()`获取
* feature类有两个部分，Geometry对象和attributes属性,attributes包含要素相关的数据。比如：{type:'circle'},通过getProperties().attributes去获取。
**geometry**
* new ol.geom.Point([x1,y1])  点
* new ol.geom.LineString([[x1,y1],[x2,y2]]) 线
* new ol.geom.LinearRing()
* new ol.geom.MultiLineString([[[x1,y1],[x2,y2]],[[x3,y3],[x4,y4]],[...]]) 多条线
* new ol.geom.MultiPoint([[x1,y1],[x2,y2],[...]]) 多个点
* new ol.geom.Polygon([[[x1,y1],[x2,y2],[x3,y3],[x4,y4],[...],[x1,y1]]]) 几何
如果这些坐标是经纬度坐标的话，都需要坐标转换`applyTransform(ol.proj.getTransform('EPSG:4326', 'EPSG:3857'))`,(部分图形展示请看demo)
```javascript
var polygon = new ol.geom.Polygon([[[110, 39], [116, 39], [116, 33], [110, 33], [110, 39]]]);
polygon.applyTransform(ol.proj.getTransform('EPSG:4326', 'EPSG:3857')); // 坐标转换
var square = new ol.Feature({
    geometry:polygon
})
layer.getSource().addFeature(square)
```
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
    source: new ol.source.Vector({
        features:[new ol.Feature({
            geometry: new ol.geom.Point(ol.proj.transform([104, 30], 'EPSG:4326', 'EPSG:3857')),
        })]
    }),
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
一个层layer有一个featrue，也可以有多个feature，因为`Feature类是Vector类用来在地图上展示几何对象，是Vector图层类一个属性。这个属性是个*要素数组*。 `
对于style样式问题，feature的style的层级比layer.Vector的层级要高，所以feature的style会覆盖layer的style。
ps：feature中的样式不能以属性名的形式写，因为feature类中没有style属性名，只有`geometry`,所以要以方法的形式去添加，setStyle()。

其实关于layer，vector，API上都有其方法，包括set／get，虽然类有很多子类，子类又有很多子子类，也无妨，只要抓住想要操作的是哪一个部分就行，是feature，还是layer，还是geometry等等。

`styleFunction`
    在`feature`中可以使用`styleFunction`来设置自己随心所欲的样式，通过官网API文档可以看到，其类型为ol.FeatureStyleFunction，函数仅带有一个参数resolution，在函数体内this指的是当前的feature，根据文档说明，这个函数要返回一个style数组。
    除了feature可以设置样式之外，layer也是可以设置样式的，同样地也支持styleFunction，但是需要注意的是，其定义和feature的不一样，类型为ol.style.StyleFunction，该函数具有两个参数，第一个参数为feature，第二个参数为resolution，同样地，该函数需要返回style数组。
```javascript
var layer = new ol.layer.Vector({
    source: new ol.source.Vector()
})
var map = new ol.Map({
    layers: [
        new ol.layer.Tile({
        source: new ol.source.OSM()
        }), 
        layer
    ],
    target: 'map',
    view: new ol.View({
        projection: 'EPSG:4326',
        center: [104, 30],
        zoom: 10
    })
});

var anchor = new ol.Feature({
    geometry: new ol.geom.Point([104, 30])
});
// 应用style function，动态的获取样式
anchor.setStyle(function(resolution){
    return [new ol.style.Style({
        image: new ol.style.Icon({
        src: '../img/anchor.png',
        scale: map.getView().getZoom() / 10
        })
    })];
});

layer.getSource().addFeature(anchor);
```
#### controls
地图控件，包括缩放按钮，标尺，版权说明，指北针等等，不会随着地图的放大而放大，缩小而缩小，就相当于position:fixed一样，固定在某个地方。 在实现上，并不是在画布上绘制的，而是使用传统的HTML元素来实现的，便于同地图分离，也便于界面实现。
在openlayers 3 中，默认情况下，在地图上是不会显示这么多地图控件的，只会应用ol.control.defaults()这个函数返回的地图控件，默认包含了ol.control.Zoom，ol.control.Rotate和ol.control.Attribution这个控件。
OpenLayers 3目前内置的地图控件类都在包ol.control下面：
* ol.control.Attribution: 右下角的地图信息控件
* ol.control.FullScreen: 全屏控件
* ol.control.MousePosition: 鼠标位置控件
* ol.control.OverviewMap: 鸟瞰图控件
* ol.control.Rotate: 指北针控件
* ol.control.ScaleLine: 比例尺控件
* ol.control.Zoom: 缩放按钮控件
* ol.control.ZoomSlider: 缩放滚动条控件
* ol.control.ZoomToExtent: 放大到设定区域控件
```javascript
var map = new ol.Map({
    target:'map',
    layers:[new ol.layers.Tile({
        source: new ol.source.OSM()
    })],
    view: new ol.View({
        center:[0,0],
        projection: 'EPSG:4326',
        zoom:10,
    })
    controls: ol.control.defaults({
        attribution: false, // 信息false
        rotate: false,    // 旋转false
        zoom: false      //   缩放false
    })
})
```
map.getControls()是获取控件的信息,
如果想要添加其他的控件，可以使用map中的一个方法，map.addControl()
```javascript
map.addControl(ol.control.OverviewMap)  // 添加鸟瞰图控件
```
控件不是在画布上绘制的，而是用html实现的，所以样式问题可以按照css+html的形式去修改。

#### interactions

交互，就是人与机之间的交互模式，比如用鼠标左键双击地图可以放大地图，按住鼠标左键拖动地图可以移动浏览地图，用滚动鼠标中间的滑轮可以放大缩小地图等等。这些都是openLayers内置的，其实也可以自己去interact。
##### 内置的交互
内置的交互在map中都是默认的。
```javascript
var map = new ol.Map({
    interactions: ol.interaction.defaults().extends()
    // .... 其余代码
})
```
ol.interaction.defaults()默认包含以下交互：
* 鼠标
    * 按住alt+shift键，用鼠标左键拖动地图，就能让地图旋转，对应的交互类为ol.interaction.DragRotate。
    * 用鼠标左键双击地图，就可以放大地图，对应的交互类为ol.interaction.DoubleClickZoom。
    * 用鼠标左键，拖拽地图，就可以平移地图，对应的交互类为ol.interaction.DragPan。
    * 滚动鼠标中间的滑轮，就可以缩放地图，对应的交互类为ol.interaction.MouseWheelZoom。
    * 按住shift键，同时用鼠标左键在地图上拖动，就可以放大地图，对应的交互类为ol.interaction.DragZoom。
* 触摸屏
    * 在触摸屏上，用两个手指在触摸屏上旋转，就可以旋转地图，对应的交互类为ol.interaction.PinchRotate。
    * 在触摸屏上，用两个手指在触摸屏上缩放，就可以缩放地图，对应的交互类为ol.interaction.PinchZoom。
* 键盘（`注意：需要设置tabindex，才能使div获得键盘事件`,就是在声明id的那个div中添加：tabindex="0"）
    * 用键盘上的上下左右键，就可以平移地图，对应的交互类为ol.interaction.KeyboardPan。
    * 用键盘上的+/-键，就可以缩放地图，对应的交互类为ol.interaction.KeyboardZoom。

如果想要取消某个交互事件
```javascript
var map = new ol.Map({
    interactions: ol.interaction.defaults({
        mouseWheelZoom: false, // 取消滚动鼠标中间的滑轮交互
        shiftDragZoom: false, // 取消shift+wheel左键拖动交互
    })
    // .... 其余代码
})
```
extend()为扩展
```javascript
var map = new ol.Map({
    interactions: ol.interaction.defaults().extend()
    // .... 其余代码
})
```
map.getInteraction()是获取控件的信息,
如果想要添加其他的交互，可以使用map中的一个方法，map.addInteraction()

```javascript
map.addInteraction(new ol.interaction.MouseWheelZoom); 
```

关于new Interaction(),总共有7个子类，可以用在extend([])中，也可以用addInteraction()
* DoubleClickZoom interaction，双击放大交互功能；
* DragAndDrop interaction，以“拖文件到地图中”的交互添加图层；
* DragBox interaction，拉框，用于划定一个矩形范围，例子常用于放大地图；
* DragPan interaction，拖拽平移地图；
* DragRotateAndZoom interaction，拖拽方式进行缩放和旋转地图；
* DragRotate interaction，拖拽方式旋转地图；
* DragZoom interaction，拖拽方式缩放地图；
* `Draw` interaction，绘制地理要素功能；
* KeyboardPan interaction，键盘方式平移地图；
* KeyboardZoom interaction，键盘方式缩放地图；
* `Select` interaction，选择要素功能；
* `Modify` interaction，更改要素；
* MouseWheelZoom interaction，鼠标滚轮缩放功能；
* PinchRotate interaction，手指旋转地图，针对触摸屏；
* PinchRoom interaction，手指进行缩放，针对触摸屏；
* Pointer interaction，鼠标的用户自定义事件基类；
* Snap interaction，鼠标捕捉，当鼠标距离某个要素一定距离之内，自动吸附到要素。

这些子类都可以查API，有详细解释。这些子类中，有个常用的属性`condition`或方法`handleEvent(mapBrowserEvent)`

**condition**

代表的是事件名称，比如：click，doubleClick，主要是依据`ol.events`。主要有以下几种，默认是singleClick
* altKeyOnly  仅按下alt键则返回true
* shiftKeyOnly 仅按下shift键则返回true
* altShiftKeysOnly  仅按下alt键+shift则返回true
* always 
* never
* click，只要是点击，包括singleClick，doubleClick都返回true
* doubleClick
* singleClick
* focus 如果地图具有焦点，则返回true。此条件需要具有tabindex属性的地图目标元素，例如<div id="map" tabindex="1">。
* mouseOnly 从鼠标设备发起为true
* noModifierKeys 没有组合键则返回true
* pointerMove 指针移动时返回true
* targetNotEditable 如果目标元素不可编辑，则返回true,即不是input,select或textarea元素false。
* platformModifierKeyOnly
* primaryAction 从一个主指针在与表面接触或起源如果按下鼠标左键

**handleEvent(mapBrowserEvent)**
```javascript
function (mapBrowserEvent){
    return ol.events.condition.click(mapBrowserEvent) && ol.events.condition.shiftKeyOnly(mapBrowserEvent);
}
```


###### Select
是个选择图形的类，用于交互.
`new ol.interaction.Select(options)`
options是个对象参数，包括：

* style
* layers 应从中选择要素的图层列表。或者，可以提供过滤功能。将为地图中的每个图层调用该函数，并应返回true您想要选择的图层。如果该选项不存在，则所有可见图层都将被视为可选择。
* condition 类型为 ol.events.ConditionType，规定了什么情况下触发 select 操作，默认不需要特殊条件进行触发。
* addCondition 
* removeCondition
* toggleCondition 获取module:ol/MapBrowserEvent-MapBrowserEvent和返回布尔值的函数，以指示是否应该处理该事件。这是condition事件的补充。默认情况下， module:ol/events/condition-shiftKeyOnly即按下shift以及condition事件，如果当前未选中，则将该功能添加到当前选择，如果是，则将其删除。见add而remove 如果你想使用的，而不是一个触发不同的事件。
* multi 用于确定默认行为是否应仅在单击的地图位置选择单个feature或所有（重叠）feature。默认值false表示单选。
* features 
* filter 过滤 见demo
* wrapX 当地图水平显示多个相同位置时，是否显示多个勾绘任务，默认为 false
**select**点击事件
```javascript
selectSingleClick.on('select', function (event) {
    console.log(event)
})
```
该select也有set／get方法，用来获取或者设置属性，比如获取选中的features，获取所有属性名称和值的对象getProperties。

###### Draw 
是个绘制图形的类，默认支持绘制的图形类型包含 Point（点）、LineString（线）、Polygon（面）和Circle（圆）。触发的事件包含 drawstart和drawend，分别在勾绘开始时候（单击鼠标）和结束时候触发（双击鼠标）。
`new ol.interaction.Draw(options)`
options是个对象参数，包括：

* type: 几何类型（加粗为默认）
    * `Point`
    * `LineString`
    * LinearRing
    * `Polygon`
    * MultiPoint
    * MultiLineString
    * MultiPolygon
    * GeometryCollection
    * `Circle`
* source 数据源,如果想要保存绘制的图形，需要有一个载体来存储绘制的图形，比如新建一个layer，这个layer的source就是该绘制的source。
    * ```javascript
        var drawLayer = new ol.layer.Vector({
            source:new ol.source.Vector()
        }) // 新建一个层
        map.addLayer(drawLayer)
        var draw = new ol.interaction.Draw({
            source: drawLayer.getSource(), // 数据源就是新建层的数据源
            type: 'Point',
        })
        map.addInteraction(draw);
      ```
* style
* stopClick
* condition 类型为 ol.events.ConditionType，规定了什么情况下触发 draw 操作，默认不需要特殊条件进行触发。
* clickTolerance: 数值类型，单位是像素，判断用户鼠标（PC）或者手指（移动设备）的行为是添加点，还是按住鼠标或者手指不松开进行拖拽地图，默认值是 6 像素，也就是说当按下鼠标和抬起鼠标左键之间的这段时间段内，如果地图被拖动没有超过 6 像素，那么默认为添加一个点，相反如果超过了 6 像素，那么不会添加点，只是平移一下地图。
* snapTolerance 数值，像素为单位，默认值是 12 像素，当一定位置内最后一个点吸附到第一个点,对多边形时有用
* features 
* maxPoints 表示绘制单个要素（面和线）最多的点数限制，默认没有限制
* minPoints 表示绘制单个要素（面和线）需要的最少点数，面默认为 3，线默认为 2
* geometryName 字符串类型，绘制的 geometry 的名称。
* geometryFunction 因为默认type只有四种：Point（点）、LineString（线）、Polygon（面）和Circle（圆），此方法就是可以用来绘制规则或者自己想要绘制的图形的方法，该函数有两个参数，一个是坐标集合、一个是勾绘的 geometry 类型，返回构造好的 geometry 对象，用来初始化要素。
    * ```javascript
        var draw = new ol.interaction.Draw({ // 绘制矩形
            type: 'LineString',
            maxPoints: 2,
            geometryFunction: function(coordinates, geometry){
                if(!geometry){
                    geometry = new ol.geom.Polygon(null);
                }
                var start = coordinates[0];
                var end = coordinates[1];
                geometry.setCoordinates([
                    [start, [start[0], end[1]], end, [end[0], start[1]], start]
                ]);
                return geometry;
            }
            // 其余代码块
        })
      ```
    * 上面的例子用LineString的形式去绘制矩形，这里的原理就是捕捉鼠标点击的点，然后获取限制的两个点的坐标，用来初始化一个矩形框。
* wrapX 当地图水平显示多个相同位置时，是否显示多个勾绘任务，默认为 false
* freehand 默认是false，是否以曲线的形式，不是规范化的，看demo-freehandDraw.html
* freehandCondition 类型同condition，也是 ol.events.ConditionType，这个选项规定了什么情况下触发不用重复点击绘制模式，即拖拽鼠标就可以绘制图形的模式，默认情况下是按下 shift 按键，这种模式对于不容易绘制的曲线比较方便，而且释放 shift 情况下，如果没有完成绘制，可以继续使用点击绘制。(就是需要辅助键，例如shift，ctrl时，会以曲线形式绘画)--demo-secondDraw.html，按shift绘画可以发现。
**drawstart/drawend**
```javascript
// 绘制结束时进行回调
draw.addEventListener('drawend', function (evt) {
    // 获取绘制图形的所有坐标点（终止点是起始点）
    var feature = evt.feature
    var geometry = feature.getGeometry()
    var coordinate = geometry.getCoordinates()
    // console.log(coordinate)
    var lent = coordinate[0].length // 坐标点个数
})
```

###### modify
用于修改要素几何的交互。要修改已添加到现有源的功能，请使用该source选项构建修改交互，如果要修改集合中的要素（例如，选择交互使用的集合），请使用该features选项构建交互。`必须使用source或features构建交互`
`默认情况下，交互将允许在alt 按下键时删除顶点。要配置具有不同删除条件的交互，请使用该deleteCondition选项。`
`new ol.interaction.Modify(options)`
options是一个参数对象，如下：

* condition 
* style
* source
* features
* wrapX
* insertVertexCondition 一个函数，它接受module:ol/MapBrowserEventMapBrowserEvent并返回一个布尔值，以指示是否可以将新顶点添加到草图要素中。默认是module:ol/events/condition-always。  同deletCondition
* deleteCondition  获取module:ol/MapBrowserEventMapBrowserEvent和返回布尔值的函数，以指示是否应该处理该事件。默认情况下， module:ol/events/condition-singleClick与 module:ol/events/conditionaltKeyOnly在顶点缺失的结果。看demo-modifyDifficult.html（先选中后操作）


###### snap
在修改或绘制矢量要素时处理矢量要素的捕捉。这些功能可以来自一个module:ol/source/Vector或module:ol/Collection-Collection 任何交互对象，允许用户使用鼠标与功能进行交互可以从捕捉中受益，只要它之前添加。

快照交互会修改地图浏览器事件coordinate和pixel 属性，以强制对其进行的任何交互进行快照。
`new ol.interaction.Snap(options)`   看API
options:
* features 应提供此选项或来源。
* edge 抓住边缘。默认值时true
* vertex 捕捉到顶点。默认值是true
* source 捕捉此来源的功能。应提供此选项或功能

官方例子`Snap Interaction`

##### 事件
###### 简单例子
```javascript
var map = new ol.Map({
    layers: [
        new ol.layer.Tile({
        source: new ol.source.OSM()
        })
    ],
    target: 'map',
    view: new ol.View({
        center: ol.proj.transform(
            [104, 30], 'EPSG:4326', 'EPSG:3857'),
        zoom: 10
    })
});

// 监听singleclick事件
map.on('singleclick', function(event){
    // 通过getEventCoordinate方法获取地理位置，再转换为wgs84坐标，并弹出对话框显示
    alert(ol.proj.transform(map.getEventCoordinate(event), 'EPSG:3857', 'EPSG:4326'));
})
```
任意的事件应用，必然会有三个步骤：

* 找准事件发送者，比如上面这个例子，map就是事件发送者。 如何找到它呢？ 一般都是要交互的对象。
* 找准事件名称，比如上面例子中的singleclick，切忌不要随便想象，或者按照惯例来写名称-condition，
* 编写事件响应函数，在OpenLayers中，事件发送者都会有一个名字为on的函数，调用这个函数，就能监听指定的事件，响应函数listener具有一个参数event，这个event类就对应于API文档中事件名称后边括号里的类。
forEachFeatureAtPixel(pixel, callback, opt_options)
###### 注销事件
* ```javascript
    // 创建事件监听器
    var singleclickListener = function(event){
        alert('...');
        // 在响应一次后，注销掉该监听器
        map.un('singleclick', singleclickListener);
    };
    map.on('singleclick', singleclickListener);
  ```
* ```javascript
    // 使用once函数，只会响应一次事件，之后自动注销事件监听
    map.once('singleclick', function(event){
        alert('...');
    })
  ```
###### 自定义事件
要添加自定义事件，需要知道这样一个事实：ol.Feature继承于ol.Object，而ol.Object具有派发事件(dispatchEvent)和监听事件(on)的功能。 

* dispatchEvent(event) 分发一个事件，并调用侦听此类事件的所有监听器，event参数可以是字符串，也可以是具有type属性的Object
* on(type, listener) 触发type类型的监听器。

```javascript
// 为地图注册鼠标移动事件的监听
map.on('pointermove', function (event) {
    map.forEachFeatureAtPixel(event.pixel, function (feature) {
        // 为移动到的feature发送自定义的mousemove消息
        feature.dispatchEvent({ type: 'mousemove', event: event });
        // feature.dispatchEvent('mousemove');
    });
});

// 为feature1(之前创建的一个feature)注册自定义事件mousemove的监听
feature1.on('mousemove', function (event) {
    // 修改feature的样式为半径100像素的园，用蓝色填充
    this.setStyle(new ol.style.Style({
        image: new ol.style.Circle({
            radius: 100,
            fill: new ol.style.Fill({
                color: 'blue'
            })
        })
    }));
});
```
    
    dispatchEvent的参数具有type和event属性，必须这样构造吗？在回答这个问题之前，需要先看一下API文档，发现参数类型为goog.events.EventLike，说明它其实用的是google的closure库来实现的，通过closure库的源码我们知道，派发的事件如果是一个对象，那么必须包含type属性，用于表示事件类型。其他的属性可以自由定义，比如此处定义了event属性，并设置对应的值，为的是让鼠标事件传递给feature1的监听函数。dispatchEvent的参数会被原封不动的传递给事件响应函数，对应代码`feature1.on('mousemove', function(event){}`里参数event，可以通过调试窗口看到此处的event和dispatchEvent的参数是一样的。注意事件名称是可以自定义的，只要派发和监听使用的事件名称是一致的就可以。

`除了可以通过dispatchEvent({type: 'mousemove', event: event})这种形式派发一个事件之外，还可以通过dispatchEvent('mousemove')这中形式直接发送mousemove事件。`

