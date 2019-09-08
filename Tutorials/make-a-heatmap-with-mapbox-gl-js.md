---
title: 使用Mapbox GL JS制作一张热图
description:添加自定义HTML标记，设置样式，并使用Mapbox GL JS添加工具提示.
thumbnail: makeAHeatmapWithMapboxGlJs
level: 2
topics:
- web apps
- map design
- data
language:
- JavaScript
prereq: 熟悉前端开发概念.
prependJs:
  - "从'../../constants'导入常量;"
  - "从'@mapbox/dr-ui/note'导入注释;"
  - 从'@mapbox/dr-ui/book-image'导入BookImage;"
  - "从'@mapbox/mr-ui/icon'导入图标;"
  - "从'../../components/user-access-token'导入UserAccessToken;"
  - "从'@ mapbox / dr-ui / demo-iframe'导入DemoIframe;"
  - "从'@mapbox/mr-ui/button'导入按钮;"
contentType: 初学者
---

在这篇教程中, 你会学到如何使用[Mapbox GL JS](https://www.mapbox.com/mapbox-gl-js/api/) 来制作一张热图。 热图用于以视觉上吸引人的方式地展示海量的数据信息，并且鼓励您的用户积极探索您所开发的地图。

{{
  <DemoIframe src="/help/demos/heatmap/index.html" />
}}

## 准备开始

- **一个 Mapbox 账号以及相应的 access token**. 在 mapbox.com/signup 注册一个账号，你可以在你的账号界面[Account page](https://www.mapbox.com/account)找到你拥有的access tokens。
- [**Mapbox GL JS**](https://www.mapbox.com/mapbox-gl-js). 包含Mapbox 中用于制作web地图的JavaScript API接口.
- **Data**. 在这篇教程中, 你将使用一个包含匹斯堡市街道树木信息 [Western Pennsylvania Regional Data Center](http://www.wprdc.org/)的GeoJSON 文件.

{{
<Button href="/help/demos/heatmap/trees.geojson" passthroughProps={{ download: "trees.geojson" }} >
    <Icon name='arrow-down' inline={true} /> 下载GeoJSON
</Button>
}}

## 热图的主要目的是什么?

"热图"这个术语可以指许多不同种类的制图可视化，你可能看到他用于总统选举支持率图，人口密度图，甚至是气象图。

在你网络上可以找到的地图中，最常见的有两种热图：第一种是鼓励用户取探索密集点数据的热图，第二种是连续曲面上离散插值，并且在这些点直接创建平滑渐变的热图。后者比前者用的更少一些，它通常被用于科学出版物或者当一个现象以一种可预测的方式分布在一个区域时。例如，你的城市可能只有很少的几个气象预测站，但是你最喜欢的天气应用程序会显示整个城市区域的平滑温度梯度。对于你的城镇本地天气服务，我们可以合理地假设：如果两个相邻的报告了不同的温度，那么他们之间的温度将会平滑地从一个站点变化到下一个站点。

这篇教程的目的在于, 我们指出一种不同类型的可视化，它能更好地展示区域上点的密集度。这种类型的热图并不是通过以[choropleth](/help/tutorials/choropleth-studio-gl-pt-1/)映射的方式来聚合一组边界内的特征点来显示密度, 而是在两个点之间连续地渐变来展示密度。 

热图不仅用于可视化地展示密度，它还有助于可视化地展示这些点之间的差异。你可以根据每个点在数据集中的特征值来为每个点分配更高或者更低的权重。在这篇教程中，你将会你数据集中的`DBH`属性进行加权. `DBH` 代表 "胸部直径" 并且这是在离地4.5英尺测量树木直径的标准方法。通常而言，可以安全地假设拥有更高DBH的树更加年老且体积更大。当你给这些比较大的树一些相比较小树苗更高的权重时，你的可视化可以是区域森林覆盖率的有效近似。 

## 热图绘制属性
<!-- copyeditor ignore represents -->
添加一个热图图层 [heatmap layer](https://www.mapbox.com/mapbox-gl-js/style-spec/#layers-heatmap) 到你的地图, 你需要配置一些属性。理解这些属性含义是创建出准确表示你的数据地图的关键，并且在太多细节和简单通用之间找到正确的平衡。
- **heatmap-weight热图权重**: 用于衡量每个点各自为你的热图表现起多大的作用。热图图层的权重默认是1.0，这意味着所有点的权重都是均等的。把 `heatmap-weight` 属性增加到5.0 和把五个点放在同一个位置具有同样的效果。你可以使用一个暂停的功能来为每个点设置基于指定属性的权重。
- **heatmap-intensity热图强度**: 一个在 `heatmap-weight`之上的乘数， 主要用于方便地根据缩放级别来调整热图外观。
- **heatmap-color热图颜色**: 定义了热图的颜色渐变，从最小值到最大值。颜色取决于每个像素点的热图密度 [heatmap-density](https://www.mapbox.com/mapbox-gl-js/style-spec#expressions-heatmap-density) (范围从`0.0` 到 `1.0`)，这个属性值是 [expression](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions) 它使用热图密度作为输入。有关热图颜色的选择灵感，请查看 [Color Brewer](http://colorbrewer2.org/#type=sequential&scheme=BuGn&n=3).
- **heatmap-radius热图半径**: 为每个点设置以像素为单位的半径，半径越大，热图越平滑，而细节也越少。
- **heatmap-opacity热图不透明度**: 控制热图图层的全局不透明度.

## 使用 Mapbox GL JS 创建你的地图

Now that you understand the purpose of heatmaps and the paint properties you will be working with, it's time to set up your map. For this example, you will be using the Mapbox Dark [template style](https://www.mapbox.com/studio-manual/reference/styles/#mapbox-template-styles). You can find the [Style URLs](/help/glossary/style-url) for each of the template styles in [our API documentation](https://docs.mapbox.com/api/maps/#styles).

In your text editor, create a new `index.html` file, then copy and paste the below code into it. Make sure to replace `{{ <UserAccessToken /> }}` with an [access token](/help/glossary/access-token/) that is associated with your account. Once you add this code and save your `index.html` file, you can preview the file in your browser to make sure you see the map.

{{
  <Note imageComponent={<BookImage />}>
    <p>Be sure to save and store the GeoJSON file in the same directory as your project. You will also need to run this application from a local web server, otherwise you will receive a <a href='http://en.wikipedia.org/wiki/Cross-origin_resource_sharing'>Cross-origin Resource Sharing (CORS)</a> error. <a href='http://www.2ality.com/2014/06/simple-http-server.html'>Python's SimpleHTTPServer</a> is included on many computers and is a good choice if this is your first time running a local server.</p>
  </Note>
}}

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset='utf-8' />
    <title></title>
    <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
    <script src='https://api.tiles.mapbox.com/mapbox-gl-js/{{constants.VERSION_MAPBOXGLJS}}/mapbox-gl.js'></script>
    <link href='https://api.tiles.mapbox.com/mapbox-gl-js/{{constants.VERSION_MAPBOXGLJS}}/mapbox-gl.css' rel='stylesheet' />
    <style>
      body {
        margin: 0;
        padding: 0;
      }

      #map {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100%;
      }
    </style>
</head>
<body>
  <div id='map'></div>
  <script>
    mapboxgl.accessToken = '{{ <UserAccessToken /> }}';
    var map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/dark-v{{constants.VERSION_DARK_STYLE}}',
      center: [-79.999732, 40.4374],
      zoom: 11
    });

   // we will add more code here in the next steps

  </script>
</body>
</html>

```
### Add your data

You will first need to add the GeoJSON you downloaded at the beginning of this guide as the source for your heatmap. You can do this by using the [`addSource`](https://www.mapbox.com/mapbox-gl-js/api/#map#addsource) method. This source will be used to create not only a heatmap layer but also a circle layer. The heatmap layer will fade out while the circle layer fades in to show individual data points at higher zoom levels. Add the following code after the map you initialized in the previous step.

```js
map.on('load', function() {

  map.addSource('trees', {
    type: 'geojson',
    data: './trees.geojson'
  });
  // add heatmap layer here
  // add circle layer here
});
```

### Add the heatmap layer

Next, use the [`addLayer`](https://www.mapbox.com/mapbox-gl-js/api/#map#addlayer) method to create a new layer for your heatmap. Once you've created this layer, you will make use of the heatmap properties discussed earlier to fine-tune your heatmap's appearance.

For `heatmap-weight`, specify a range that reflects your data (the `dbh` property ranges from 1-62 in the GeoJSON source). Because larger trees have a high `dbh`, give them more weight in your heatmap by creating a stop function that increases `heatmap-weight` as `dbh` increases.

Since `heatmap-intensity` is a multiplier on top of `heatmap-weight`, `heatmap-intensity` can be increased as the map zooms in to preserve a similar appearance throughout the zoom range. The images below show the impact of `heatmap-intensity` on your map's appearance. The image on the left shows `heatmap-intensity` that increases with zoom level and the one on the right shows `heatmap-intensity` that uses the default of `1`.

{{
<div className='grid'>
  <img className='col' alt='demonstrates a heatmap-intensity that increases with zoom level' style={{ width: "50%", paddingRight: "10px", height: "50%" }} src='/help/img/gl-js/heatmap-intensity-two.png' />
  <img className='col' alt='demonstrates heatmap intensity default of one' style={{ width: "50%", paddingLeft: "10px", height: "50%" }} src='/help/img/gl-js/heatmap-intensity-one.png' />
</div>
}}

For `heatmap-color`, add an [interpolate](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions-interpolate) expression that defines a linear relationship between heatmap-density and heatmap-color using a set of input-output pairs. If you are interested in learning more about Mapbox GL JS Expressions, read the [Get Started with Mapbox GL JS expressions guide](/help/tutorials/mapbox-gl-js-expressions) and the [Mapbox GL JS documentation](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions).

Finish configuring your heatmap layer by setting values for `heatmap-radius` and `heatmap-opacity`. `heatmap-radius` should increase with zoom level to preserve the smoothness of the heatmap as the points become more dispersed. `heatmap-opacity` should be decreased from `1` to `0`  between zoom levels 14 and 15 to provide a smooth transition as your circle layer fades in to replace the heatmap layer. Add the following code within the 'load' event handler after the `addSource` method.

```js
map.addLayer({
  id: 'trees-heat',
  type: 'heatmap',
  source: 'trees',
  maxzoom: 15,
  paint: {
    // increase weight as diameter breast height increases
    'heatmap-weight': {
      property: 'dbh',
      type: 'exponential',
      stops: [
        [1, 0],
        [62, 1]
      ]
    },
    // increase intensity as zoom level increases
    'heatmap-intensity': {
      stops: [
        [11, 1],
        [15, 3]
      ]
    },
    // assign color values be applied to points depending on their density
    'heatmap-color': [
      'interpolate',
      ['linear'],
      ['heatmap-density'],
      0, 'rgba(236,222,239,0)',
      0.2, 'rgb(208,209,230)',
      0.4, 'rgb(166,189,219)',
      0.6, 'rgb(103,169,207)',
      0.8, 'rgb(28,144,153)'
    ],
    // increase radius as zoom increases
    'heatmap-radius': {
      stops: [
        [11, 15],
        [15, 20]
      ]
    },
    // decrease opacity to transition into the circle layer
    'heatmap-opacity': {
      default: 1,
      stops: [
        [14, 1],
        [15, 0]
      ]
    },
  }
}, 'waterway-label');
```

### Add the circle layer
Next, add a circle layer. As you zoom in to your heatmap, the points stop overlapping visually and it is no longer necessary to show their distribution and density. At this point, you can show the points themselves and allow viewers to explore the data interactively.

Remember how you used a stop function in the previous step to fade the heatmap layer out between zoom level 14 and 15? You'll need to replace that layer by fading your circle layer in, using a zoom function to increase its `circle-opacity` between zooms 14 and 15. For `circle-radius`, use a zoom-and-property function to increase the radius by zoom level and property (as demonstrated below). Add the following code after the heatmap layer you added in the last step.

```js
map.addLayer({
  id: 'trees-point',
  type: 'circle',
  source: 'trees',
  minzoom: 14,
  paint: {
    // increase the radius of the circle as the zoom level and dbh value increases
    'circle-radius': {
      property: 'dbh',
      type: 'exponential',
      stops: [
        [{ zoom: 15, value: 1 }, 5],
        [{ zoom: 15, value: 62 }, 10],
        [{ zoom: 22, value: 1 }, 20],
        [{ zoom: 22, value: 62 }, 50],
      ]
    },
    'circle-color': {
      property: 'dbh',
      type: 'exponential',
      stops: [
        [0, 'rgba(236,222,239,0)'],
        [10, 'rgb(236,222,239)'],
        [20, 'rgb(208,209,230)'],
        [30, 'rgb(166,189,219)'],
        [40, 'rgb(103,169,207)'],
        [50, 'rgb(28,144,153)'],
        [60, 'rgb(1,108,89)']
      ]
    },
    'circle-stroke-color': 'white',
    'circle-stroke-width': 1,
    'circle-opacity': {
      stops: [
        [14, 0],
        [15, 1]
      ]
    }
  }
}, 'waterway-label');
```

## Add some additional interactivity
The following code adds interactivity to your map by allowing your viewers to click on your circle layer to view a popup containing the tree's `DBH` value. Include the code below after your circle layer.

{{<img src='/help/img/gl-js/heatmap-popup.gif' alt='demonstrates how to add a popup to the circle layer of a heatmap' className="w-full" />}}

```js
map.on('click', 'trees-point', function(e) {
  new mapboxgl.Popup()
    .setLngLat(e.features[0].geometry.coordinates)
    .setHTML('<b>DBH:</b> ' + e.features[0].properties.dbh)
    .addTo(map);
});

```

## Finished product

Congrats! You made your first heatmap with Mapbox GL JS!

{{
  <DemoIframe src="/help/demos/heatmap/index.html" />
}}
