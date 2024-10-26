# 基于百度地图(bmap)的停车场信息查询与导航系统

## 一、概述

基于百度地图的停车场信息查询与导航系统主要包括以下功能：

1. 用户位置定位
2. 停车场信息查询
3. 显示空余车位数量
4. 导航到最近的停车场
5. 清除导航
6. 下拉刷新

## 二、功能实现

### 1. 用户位置定位

通过调用百度地图API中的Geolocation类，实现用户当前位置的定位。

```javascript
var geolocation = new BMapGL.Geolocation();
geolocation.getCurrentPosition(function (r) {
  if (this.getStatus() == BMAP_STATUS_SUCCESS) {
    // 定位成功，在地图上添加标记
    var mk = new BMapGL.Marker(r.point);
    map.addOverlay(mk);
    map.panTo(r.point);
    // 更新位置信息
    const info_p = document.querySelector('.info-box p');
    info_p.innerHTML = `我的位置：${r.address.city}${r.address.district}${r.address.street}`;
  } else {
    alert('failed' + this.getStatus());
  }
});
```

### 2. 停车场信息查询

使用LocalSearch类进行本地搜索，限定搜索范围为2000米，查询停车场信息。

```javascript
var local = new BMapGL.LocalSearch(map, {
  renderOptions: { map: map, panel: "my-panel", }
});
local.search(`${r.address.district}停车场`, { location: r.point, radius: 20000 });
```

### 3. 显示空余车位数量

通过setMarkersSetCallback回调函数，每隔30秒随机生成空车位数量，并更新地图上的标注。

```javascript
local.setMarkersSetCallback(function (pois) {
  setInterval(function () {
    for (var i = 0; i < pois.length; i++) {
      pois[i].parking_num = Math.floor(Math.random() * 100);
      var content = pois[i].title + "<br>" + '空车位：' + pois[i].parking_num;
      pois[i].marker.setLabel(new BMapGL.Label(content, { offset: new BMapGL.Size(20, -10) }));
    }
  }(), 30000);
});
```

### 4. 导航到最近的停车场

通过DrivingRoute类实现导航功能。首先计算用户位置与停车场之间的距离，找到最近的停车场，然后添加导航按钮事件。

```javascript
const nav_button = document.querySelectorAll('.info-box button')[1];
nav_button.addEventListener('click', function () {
  driving.search(r.point, pois[min_index].point);
});
```

### 5. 清除导航

为清除导航按钮添加事件，清除地图上的导航线路和重新查询停车场信息。

```javascript
const clear_button = document.querySelectorAll('.info-box button')[2];
clear_button.addEventListener('click', function () {
  driving.clearResults();
  // 重新添加用户位置标记和查询停车场信息
  map.addOverlay(mk);
  mk.setLabel(label);
  local.search(`${r.address.district}停车场`, { location: r.point, radius: 20000 });
});
```

## 三、界面交互

### 1. 下拉刷新

通过监听info-box的touch事件，实现下拉刷新功能。

```javascript
box.addEventListener('touchstart', function (e) {
  // ...
});
box.addEventListener('touchmove', function (e) {
  // ...
});
box.addEventListener('touchend', function (e) {
  // ...
});
```

### 2. 查看车位按钮

点击查看车位按钮，显示停车场信息面板。

```javascript
search_button.addEventListener('click', function () {
  const my_panel = document.querySelector('#my-panel');
  my_panel.style.display = 'block';
  const close = document.querySelector('.close');
  close.style.display = 'block';
  close.addEventListener('click', function () {
    my_panel.style.display = 'none';
    close.style.display = 'none';
  });
});
```

# 基于高德地图(amap)的停车场信息查询与导航系统

## 一、系统概述

基于高德地图API开发的停车场信息查询与导航系统和上文使用百度地图做的实现的是同样的功能，集成了高德地图的定位、搜索、导航等功能。

## 二、系统功能实现

### 1. 用户位置定位

系统通过高德地图的Geolocation插件实现用户位置的精确定位。

```javascript
AMap.plugin('AMap.Geolocation', function () {
  geolocation = new AMap.Geolocation({ /* 配置选项 */ });
  map.addControl(geolocation);
  geolocation.getCurrentPosition(function (status, result) {
    // 定位成功后的回调函数
  });
});
```

### 2. 停车场信息查询

使用PlaceSearch插件，根据用户当前城市编码搜索周边停车场。

```javascript
AMap.plugin('AMap.PlaceSearch', function () {
  placeSearch = new AMap.PlaceSearch({ city: current_position, /* 其他配置 */ });
  placeSearch.search('停车场');
});
```

### 3. 显示停车场详细信息

在地图上为每个停车场添加标记，并通过InfoWindow插件显示详细信息。

```javascript
data.poiList.pois.forEach((item) => {
  const marker = new AMap.Marker({ position: item.location, /* 其他配置 */ });
  marker.on('click', function () {
    // 显示信息窗口
  });
});
```

### 4. 导航到最近的停车场

利用Driving插件，为用户提供从当前位置到最近停车场的导航路线。

```javascript
nav_button.addEventListener('click', function () {
  AMap.plugin('AMap.Driving', function () {
    driving = new AMap.Driving({ map: map, /* 其他配置 */ });
    driving.search([{ keyword: '我的位置', city: current_position }, { keyword: '停车场', city: current_position }]);
  });
});
```

### 5. 下拉刷新

通过监听触摸事件，实现下拉刷新功能，实时更新停车场信息。

```javascript
box.addEventListener('touchstart', function (e) {
  // 触摸开始
});
box.addEventListener('touchmove', function (e) {
  // 触摸移动
});
box.addEventListener('touchend', function (e) {
  // 触摸结束
});
```

## 三、系统集成

系统通过高德地图Loader.js异步加载地图API，确保页面加载效率。

```javascript
AMapLoader.load({ key: "您的开发者Key", version: "2.0" })
  .then((AMap) => { /* 初始化地图和应用功能 */ })
  .catch((e) => { console.error(e); });
```

# 总结

通过这些简单又实用的功能，我们的系统让你停车无忧，出行更轻松！快来体验一下吧，让你的停车位不再是难题！🚗🌟