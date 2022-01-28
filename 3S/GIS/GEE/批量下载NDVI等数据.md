### 1、选择你需要的数据源，这里我们选择NDVI/EVI/LAI/NPP/ET：

```json
//MODIS13Q1产品下的NDVI和EVI产品,16天/幅,分辨率为250米
function ModisNDVI(roi, startTime, endTime) {
    startTime = ee.Date(startTime).millis();
    endTime = ee.Date(endTime).millis();

    var Modis_250 = ee.ImageCollection("MODIS/006/MOD13Q1")
        .filterDate(startTime, endTime)
        .select("NDVI")
        // .select('EVI')
        .filterBounds(roi)
        .map(function (image) {
            return image.clip(roi)
        })
    return Modis_250
}
//MOD15A2H产品下的LAI产品,8天/幅,分辨率为500米
function ModisLAI(roi, startTime, endTime) {
    startTime = ee.Date(startTime).millis();
    endTime = ee.Date(endTime).millis();

    var dataset = ee.ImageCollection("MODIS/006/MOD15A2H")
        .filter(ee.Filter.date(startTime, endTime))
        .filterBounds(roi)
        .select('Lai_500m')
        .map(function (image) {
            return image.clip(roi)
        });
    return dataset
}
//MOD17A3HGF产品下的NPP产品,1年/幅,分辨率为500米
function ModisNpp(roi, startTime, endTime) {
    startTime = ee.Date(startTime).millis();
    endTime = ee.Date(endTime).millis();

    var dataset = ee.ImageCollection("MODIS/006/MOD17A3HGF")
        .filter(ee.Filter.date(startTime, endTime))
        .filterBounds(roi)
        .select('Npp')
        .map(function (image) {
            return image.clip(roi)
        });
    return dataset
}
//MOD16A2产品下的ET产品,8天/幅,分辨率为500米
function ModisET(roi, startTime, endTime) {
    startTime = ee.Date(startTime).millis();
    endTime = ee.Date(endTime).millis();

    var dataset = ee.ImageCollection("MODIS/006/MOD16A2")
        .filter(ee.Filter.date(startTime, endTime))
        .filterBounds(roi)
        .select('ET')
        .map(function (image) {
            return image.clip(roi)
        });
    return dataset
}
```

### 2、定义批量输出函数：

```json
//----------------------数据批量输出函数-----------------------//  
function exportImageCollection(imgCol, scale, roi, taskName, fileName) {
    var indexList = imgCol.reduceColumns(ee.Reducer.toList(), ["system:index"]).get("list");
    indexList.evaluate(function (indexs) {

        for (var i = 0; i < indexs.length; i++) {
            var image = imgCol.filter(ee.Filter.eq("system:index", indexs[i])).first();
            image = image.toInt16(); //
            Export.image.toDrive({
                image: image,
                description: taskName + "_" + indexs[i],
                fileNamePrefix: fileName + "_" + indexs[i],
                region: roi,
                scale: scale,
                crs: "EPSG:4326",
                maxPixels: 1e13
            });
        }
    });

}
```

### 3、定义批量显示函数（如果不想显示可以不用）：

```json
// 批量显示图像(单波段)
function BatchMap(IC, visParam, listLimt, preName) {
    function getICByIndex(IC, index) {
        return ee.Image(IC.toList(1, index).get(0));
    }
    for (var i = 0; i < IC.toList(listLimt).length().getInfo(); i++) {
        var image = getICByIndex(IC, ee.Number(i));
        //以时间戳为图层名  
        var imageDate = getICByIndex(IC, i).get("system:index").getInfo();
        Map.addLayer(image,
            visParam,
            preName + "_" + imageDate, false);
    }
}
```

### 4、将你想要的数据类型和范围加进去，然后便能直接导出：

```json
//主函数
function main() {
     //放入你的研究区范围
    var ROI = table.filter(ee.Filter.eq('provinces', 'sichuan'));
    //显示ROI轮廓
    Map.centerObject(ROI, 5.3);
    var empty = ee.Image().clip(ROI.geometry().bounds()).byte();
    var shp = empty.paint({ featureCollection: ROI, width: 4 });
    Map.addLayer(shp, { palette: '#000000' }, 'SiChuan_region');

    //查询 NDVI(EVI)数据
    var Modis_NDVI = ModisNDVI(ROI, "2020-01-01", "2020-03-01");
    //显示 NDVI(EVI)数据
    var visParam_NDVI = { min: 0, max: 8000 };
    BatchMap(Modis_NDVI, visParam_NDVI, 50, "Modis_NDVI");
    print(Modis_NDVI, "Modis_VI_collection");

    //查询 LAI数据
    var Modis_LAI = ModisLAI(ROI, "2020-01-01", "2021-03-01");
    //显示 LAI数据
    var visParam_LAI = { min: 0, max: 70 };
    BatchMap(Modis_LAI, visParam_LAI, 50, "Modis_LAI");
    print(Modis_LAI, "Modis_VI_collection");

    //查询 NPP数据 
    var Modis_NPP = ModisNpp(ROI, "2020-01-01", "2021-01-01");
    //显示 NPP数据
    var visParam_NPP = { min: 0, max: 10000 };
    BatchMap(Modis_NPP, visParam_NPP, 100, "Modis_NPP");
    print(Modis_NPP, "Modis_NPP_collection");

    //查询 ET数据 
    var Modis_ET = ModisET(ROI, "2020-01-01", "2020-03-01");
    //显示 ET数据
    var visParam_ET = { min: 0, max: 200 };
    BatchMap(Modis_ET, visParam_ET, 100, "Modis_ET");
    print(Modis_ET, "Modis_NPP_collection");

    //数据导出
    exportImageCollection(Modis_NDVI, 250, ROI, "MODIS_NDVI", "MODIS_NDVI");
    exportImageCollection(Modis_LAI, 500, ROI, "MODIS_LAI", "MODIS_LAI");
    exportImageCollection(Modis_NPP, 500, ROI, "MODIS_NPP", "MODIS_NPP");
    exportImageCollection(Modis_ET, 500, ROI, "MODIS_ET", "MODIS_ET");
}
main();
```

### 5、如果你的数据量太大了，不想一直点击RUN，可以按F12进入控制台，复制下面的代码，一键RUN你的数据：

```json
function runTaskList(){
    var tasklist = document.getElementsByClassName('task local type-EXPORT_IMAGE awaiting-user-config');
    for (var i = 0; i < tasklist.length; i++)
            tasklist[i].getElementsByClassName('run-button')[0].click();
}

function confirmAll() {
    var ok = document.getElementsByClassName('goog-buttonset-default goog-buttonset-action');
    for (var i = 0; i < ok.length; i++)
        ok[i].click();
}
runTaskList();
confirmAll();
```

### 6、如果你想取消一键RUN，可以在Task Manager中，找到Bulk cancel mode模块，一键取消：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3YmaWSj9gtQicCjJQRuzxreoPC2wFqSphXfv6I4bxVzThu8T5kTvJNl04pctfdCwMFWRia2ib3aCb2Sd5xoUHSFcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 7、结果显示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/3YmaWSj9gtQicCjJQRuzxreoPC2wFqSph385Md8R6GLkxK3CgPnc9kqTw3ibZIxFeO0Vvh7Ry76RAMHbbuuDHBGA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)