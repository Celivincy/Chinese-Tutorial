---
title: 根据距离为商店排序
description: 这篇教程将指导你通过代码去构建一个商店定位工具，它能搜索离你的当前位置最近的商店。
thumbnail: 地理编码和排序
level: 3
language:
- JavaScript
prereq: 熟悉前端开发的概念，以及一些进阶的JavaScript使用技巧.
topics:
- web apps
- geocoding
prependJs:
  - "从 '@mapbox/mr-ui/icon'导入图标;"
  - "从 '@mapbox/dr-ui/demo-iframe'导入DemoIframe;"
  - "从 '../../constants'导入* 作为常量;"
  - "从 '@mapbox/dr-ui/note'导入注释;"
  - "从 '@mapbox/dr-ui/book-image'导入BookImage;"
contentType: 初学者
---

这篇教程将指导你如何去使用 [Mapbox GL JS](https://www.mapbox.com/mapbox-gl-js/),以及 [Mapbox GL Geocoder plugin](https://github.com/mapbox/mapbox-gl-geocoder)和 [Turf.js](http://turfjs.org/) 来根据离地理编码点的距离从近到远排序周围的商店。这篇教程是在 [Build a store locator using Mapbox GL JS](/help/tutorials/building-a-store-locator) 教程中所创建的地图上进行扩充。如果你还没有完成上一篇教程， 你需要点击链接完成该教程后再来开启本项目。 如果你是初次接触 Mapbox GL JS,，你可能还需要先阅读一下我们的网页app开发指导 [Web applications guide](/help/how-mapbox-works/web-apps)。

{{
  <DemoIframe src="/help/demos/geocode-and-sort/index.html" />
}}

## 准备开始

在这个项目中，我们建议你创建一个名为 "sort-store-locator" 的本地文件夹来存放你的项目文件。在下文中，这个文件夹将被称为*项目文件夹*.

你需要在开始前准备以下资源：

- __存储定位工具的最终项目__. 这个初学者教程建立在 [Build a store locator using Mapbox GL JS](/help/tutorials/building-a-store-locator) 教程中的代码上。确保你复制了链接中的最终版本代码到你的项目文件夹中，或者你从这里下载一份 <a href="/help/demos/geocode-and-sort/starter-code.zip">{{<Icon name='arrow-down' inline={true} />}} 下载预备代码</a>
- [__一个密钥access token__](/help/glossary/access-token/) 来自你的个人账号。你将会使用这个密钥access token来绑定你创建的地图到你的账户。你拥有的所有密钥都在这里 [Account page](https://account.mapbox.com/)。
- [__Mapbox GL JS__](https://www.mapbox.com/mapbox-gl-js/) Mapbox 的 JavaScript 库使用了 WebGL 来渲染带有 Mapbox GL styles 的可交互地图。
- [__Mapbox GL Geocoder__](https://www.mapbox.com/mapbox-gl-js/plugins/) 插件。 用于 [Mapbox 地理位置编码的 API](https://docs.mapbox.com/api/search/#geocoding)的Mapbox GL JS 包装器wrapper库。
- [__Turf.js__](http://turfjs.org)。一个开源的分析库，它可以在浏览器和 Node.js中执行空间分析。
- __一个文本编辑器.__ 在这里编写 HTML, CSS, 和 JavaScript的代码文件。

## 添加插件并初始化地图

下载 `预备代码` 压缩文件。在里面你可以找到一个名为 `index.html` 的文件以及一个 `img` 文件夹，该文件夹中包含了你用来展示商店位置的自定义marker。在文本编辑器中打开 `index.html` 文件，把文件中的 `mapboxgl.accessToken`替换为你的密钥access token。

### 添加 Mapbox GL geocoder 插件以及 Turf.js

下一步，在你的HTML文件的`head`添加 Mapbox GL Geocoder 插件和 Turf.js 库的链接来启动你的文档：拷贝并粘贴以下代码在 Mapbox GL JS 的链接之后即可完成。

```html
  <!-- Geocoder plugin -->
  <script src='https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-geocoder/{{constants.VERSION_GLJS_GEOCODER_PLUGIN}}/mapbox-gl-geocoder.min.js'></script>
  <link rel='stylesheet' href='https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-geocoder/{{constants.VERSION_GLJS_GEOCODER_PLUGIN}}/mapbox-gl-geocoder.css' type='text/css' />

  <!-- Turf.js plugin -->
  <script src='https://npmcdn.com/@turf/turf/turf.min.js'></script>
```

### 添加地理编码控件

使用 `new mapboxgl.Geocoder`构造器来添加地理编码控件到你的 JavaScript代码中。这样你就可以通过`bbox`参数把搜索结果限制在华盛顿特区。还有许多其他你可以指定的参数。更多关于可用参数的信息可以在 [documentation on GitHub](https://github.com/mapbox/mapbox-gl-geocoder/blob/master/API.md)中阅读并学习。

在你的 `script` 标签对中，你需要把以下代码添加到方法 `map.on('load', function (e) { ... });` 里。

```js
var geocoder = new MapboxGeocoder({
  accessToken: mapboxgl.accessToken, // Set the access token
  mapboxgl: mapboxgl, // Set the mapbox-gl instance
  marker: false, // Do not use the default marker style
  bbox: [-77.210763, 38.803367, -76.853675, 39.052643] // Set the bounding box coordinates
});

map.addControl(geocoder, 'top-left');
```

现在添加一些 CSS 样式来装饰你的地理搜索栏。你可以添加以下代码在 `</style>` 关闭标签的左侧.

```css
.mapboxgl-ctrl-geocoder {
  border: 0;
  border-radius: 0;
  position: relative;
  top: 0;
  width: 800px;
  margin-top: 0;
}

.mapboxgl-ctrl-geocoder > div {
  min-width: 100%;
  margin-left: 0;
}
```

保存你的 HTML 文档，并且刷新你的网页浏览器，运行效果会和下面展示的一致：

{{
  <DemoIframe src="/help/demos/geocode-and-sort/step-one.html" />
}}

注意当你使用你所创建网页搜索地址时会发生什么。地图会跳转到你预先指定的位置上，而并没能正确显示与搜索结果匹配的位置。在下一步中，你需要在 Mapbox GL 地理编码器成功找到一个位置后添加一个新的定位点。

##当搜索时添加定位点

{{<Note title="添加自定义Maker的样式" imageComponent={<BookImage />}>}}
  Mapbox GL 地理编码器在搜索到结果后使用默认样式来初始化一个 marker。这个例子中我们把自定义的marker添加为新图层来代替默认样式。当你想要使用默认的marker时，你可以在地理编码器的初始化中移除 `marker: false,` 这一行。
{{</Note>}}

现在你的地理编码器正在正确运行，你可以通过代码来为你搜索到的结果添加定位点到地图中。所有的代码会添加在 `map.on('load', function (e) { ... });` 里你上一步代码的后面。步骤如下：首先，你需要使用 `map.addSource()` 方法来添加一个空的源到你存储地理编码器结果的地方，接下来你需要通过`map.addLayer()`方法来使用这个源中的样式图层。

```js
map.addSource('single-point', {
  type: 'geojson',
  data: {
    type: 'FeatureCollection',
    features: [] // Notice that initially there are no features
  }
});

map.addLayer({
  id: 'point',
  source: 'single-point',
  type: 'circle',
  paint: {
    'circle-radius': 10,
    'circle-color': '#007cbf',
    'circle-stroke-width': 3,
    'circle-stroke-color': '#fff'
  }
});
```

下一步，创建一个事件监听器来监听用户点击某个地理编码结果的事件。当你的用户点击了返回结果列表中的某一个地理位置后， 将坐标保存在名为`searchResult`的变量中，接着，使用你在上面声明的 id `single-point` 来把源中的数据设置为 `searchResult`：拷贝并粘贴以下代码到 `map.addSource()` 方法 `map.addLayer()` 方法后。

```js
geocoder.on('result', function(ev) {
  var searchResult = ev.result.geometry;
  map.getSource('single-point').setData(searchResult);
});
```

运行效果如下图所示：

{{
  <DemoIframe src="/help/demos/geocode-and-sort/step-two.html" />
}}

## 根据距离为商店列表排序

下一步，计算搜索到的位置与商店之间的距离，把结果添加到你的 GeoJSON 数据中，并且根据离搜索点的距离为商店列表排序。

### 找到所有位置的距离

接下来你将会使用 Turf.js 计算你的搜索点和每个商店位置的距离。Turf.js 可以实现各种各样的空间分析功能，你可以在这个文档中 [documentation](http://turfjs.org/docs/)阅读更详细的信息。在本教程中，你将使用 [**距离distance**](http://turfjs.org/docs/#distance)来进行分析。

在你的 `geocoder.on('result', function(){...});` 方法中，使用 `forEach` 循环来迭代遍历 GeoJSON 中所有的商店位置（记住，你已经把这些信息存储在了 `stores` 变量中），为每个商店对象定义一个叫 `distance`的新属性，并且把这个属性的值设置为`searchResult`中存储的坐标和该商店的坐标点之间的距离。你可以使用 `turf.distance()`方法来接收三个参数：`from, to, options`并计算距离结果。

```js
var options = { units: 'miles' };
stores.features.forEach(function(store) {
  Object.defineProperty(store.properties, 'distance', {
    value: turf.distance(searchResult, store.geometry, options),
    writable: true,
    enumerable: true,
    configurable: true
  });
});
```

For each feature in your GeoJSON, a `distance` property is applied or will be updated each time a new geocoder result is selected.

#### Sort store list by distance

Now that you have the `distance` value for each store location, you can use it to sort the list of stores by distance.

First, sort the objects in the `stores` array by the `distance` property you added earlier. Copy and paste the following code snippet inside the `geocoder.on('result', function(){...});` function.

```js
stores.features.sort(function(a, b) {
  if (a.properties.distance > b.properties.distance) {
    return 1;
  }
  if (a.properties.distance < b.properties.distance) {
    return -1;
  }
  // a must be equal to b
  return 0;
});
```

Then, remove the current list of stores and rebuild the list using the reordered array you created. The individual listings are nested within the `div` with id `listings`.

```js
var listings = document.getElementById('listings');
while (listings.firstChild) {
  listings.removeChild(listings.firstChild);
}

buildLocationList(stores);
```

Now the listing for each store will be in ascending order of distance from the point that was searched. To make the new list of locations more useful to your viewers, add text that describes each listing's distance from the point they searched for. When you built your initial interactive store locator in the previous tutorial, you created a `buildLocationListing()` function. You will need to find and change that function to check if there is a `distance` property, and if there is, add the value of that property to each listing. Copy and paste the following code before the `link.addEventListener()` function within the `buildLocationListing()` function.

```js
if (prop.distance) {
  var roundedDistance = Math.round(prop.distance * 100) / 100;
  details.innerHTML += '<p><strong>' + roundedDistance + ' miles away</strong></p>';
}
```

The result should look like this:

{{
  <DemoIframe src="/help/demos/geocode-and-sort/step-three.html" />
}}

## Fit bounds to search result and closest store

Finally, when you search for a location, you can change the view to include both the location that was searched and the closest store to show more context. You can do this by using `map.fitBounds()` and specifying a [bounding box](/help/glossary/bounding-box/). But the bounds need to be in a specific order. The first point you specify should be the lower left corner of the bounding box, and the second should be the upper right corner. Add the following code inside the `geocoder.on()` function to create a `bbox` with this syntax from the geocoded location and the closest store, fly to it, and open the closest store's popup.

```js
function sortLonLat(storeIdentifier) {
  var lats = [stores.features[storeIdentifier].geometry.coordinates[1], searchResult.coordinates[1]];
  var lons = [stores.features[storeIdentifier].geometry.coordinates[0], searchResult.coordinates[0]];

  var sortedLons = lons.sort(function(a, b) {
    if (a > b) {
      return 1;
    }
    if (a.distance < b.distance) {
      return -1;
    }
    return 0;
  });
  var sortedLats = lats.sort(function(a, b) {
    if (a > b) {
      return 1;
    }
    if (a.distance < b.distance) {
      return -1;
    }
    return 0;
  });

  map.fitBounds([
    [sortedLons[0], sortedLats[0]],
    [sortedLons[1], sortedLats[1]]
  ], {
    padding: 100
  });
}

sortLonLat(0);
createPopUp(stores.features[0]);
```

## Finished product

You have created a store locator with geocoding and spatial analysis.

{{
  <DemoIframe src="/help/demos/geocode-and-sort/index.html" />
}}

## Next steps

After this guide, you should have everything you need to create your own store locator. You can complete the [Create a custom style tutorial](/help/tutorials/create-a-custom-style/) to create a branded map style or use [Cartogram](https://apps.mapbox.com/cartogram/), a drag and drop tool, to create a custom style from your logo in minutes. To do more with Mapbox GL JS, explore our [examples page](https://www.mapbox.com/mapbox-gl-js/examples/) and the Mapbox GL JS on the [help page](/help/tutorials/).
