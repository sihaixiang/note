```javascript
//研究区
var roi = ee.FeatureCollection("users/sihaixiang/shandong").geometry(); //("users/....")自己填入研究shp文件路径
 
//批量下载函数
function exportImage(image, roi, fileName) {  
    Export.image.toDrive({  
       image: image,  
       description: "Drive-image-"+fileName,  
       fileNamePrefix: fileName+'_LAI',  //文件命名
       folder: "MODIS_LAI",  //保存的文件夹
       scale: 500,  //分辨率
       region: roi,  //研究区
       maxPixels: 1e13,  //最大像元素，默认就好
       crs: "EPSG:4326"  //设置投影
   });  
 } 
 
//加载数据集
var data = ee.ImageCollection("MODIS/006/MOD15A2H");  //这里以MODIS LAI 数据为例
Map.centerObject(roi, 4);  //图层显示
 
//筛选数据
var data_selected = data.filterBounds(roi) 
               .filterDate("2018-3-1", "2018-5-31")
               
print("data_selected", data_selected); 
 
//生成列表，迭代下载
var indexList = data_selected.reduceColumns(ee.Reducer.toList(), ["system:index"]).get("list"); 
print("indexList", indexList);
indexList.evaluate(function(indexs) { 
for (var i=0; i<indexs.length; i++) {  
        var image = data_selected.filter(ee.Filter.eq("system:index", indexs[i]))
              .first()
              .select('LAI') //看情况是否需要
              .toInt16()  //设置数据类型 
              .clip (roi);   //裁剪数据
        exportImage(image, roi, indexs[i]);  //保存图像至Google网盘
   }
 });
```

```json
//研究区
var roi = ee.FeatureCollection("users/sihaixiang/shandong").geometry(); //("users/....")自己填入研究shp文件路径
 
//批量下载函数
//Export.table.toDrive(collection, description, folder, fileNamePrefix, fileFormat, selectors, maxVertices)
function exportTable(image, roi, fileName) {  
    Export.table.toDrive({  
       collection: image,  
       description: "Drive-image-"+fileName,  
       fileNamePrefix: fileName+'_ERA5',  //文件命名
       folder: "ERA5",  //保存的文件夹
       fileFormat: "CSV",
       //scale: 500,  //分辨率
       //region: roi,  //研究区
       //maxPixels: 1e13,  //最大像元素，默认就好
       //crs: "EPSG:4326"  //设置投影
   });  
 } 
 
//加载数据集
var data = ee.ImageCollection("ECMWF/ERA5/DAILY");  //这里以MODIS LAI 数据为例
Map.centerObject(roi, 4);  //图层显示
 
//筛选数据

var data_selected = ee.ImageCollection('ECMWF/ERA5/DAILY')
                   .select('mean_2m_air_temperature')
                   .select('total_precipitation')
                   .select('surface_pressure')
                   .select('mean_sea_level_pressure')
                   .select('dewpoint_2m_temperature')
                   .filter(ee.Filter.date('2019-07-01', '2019-07-31'));
print(data_selected);


 
//生成列表，迭代下载
var indexList = data_selected.reduceColumns(ee.Reducer.toList(), ["system:index"]).get("list"); 
print("indexList", indexList);
indexList.evaluate(function(indexs) { 
    for (var i=0; i<indexs.length; i++) {  
        var image = data_selected.filter(ee.Filter.eq("system:index", indexs[i]))
              .first()
              
              .toInt16()  //设置数据类型 
              .clip (roi);   //裁剪数据
        exportTable(image, roi, indexs[i]);  //保存图像至Google网盘
   }
 });
 

```

