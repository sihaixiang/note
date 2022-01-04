```Python
#根据时间、经纬度，查找某一类型数据，并显示该数据的波段信息

#查找数据，并显示缩略图

from pystac_client import Client

catalog = Client.open("https://planetarycomputer.microsoft.com/api/stac/v1")
#抓取数据集的目录  包括ID和标题




area_of_interest = {
    "type": "Polygon",
    "coordinates": [
        [
            [-122.27508544921875, 47.54687159892238],
            [-121.96128845214844, 47.54687159892238],
            [-121.96128845214844, 47.745787772920934],
            [-122.27508544921875, 47.745787772920934],
            [-122.27508544921875, 47.54687159892238],
        ]
    ],
}
time_range = "2020-12-01/2020-12-31"
search = catalog.search(
    collections=["sentinel-2-l2a"], intersects=area_of_interest, datetime=time_range
)

items = list(search.get_items())
for item in items:
    print(f"{item.id}: {item.datetime}")
#根据经纬度范围和时间范围查找某一类型的数据，并返回数据名称，以landsat-8为例。



selected_item = sorted(items, key=lambda item: item.properties["eo:cloud_cover"])[0]
#使用eo扩展程序按云量对项目进行排序



for asset_key, asset in selected_item.assets.items():
    print(f"{asset_key:<25} - {asset.title}")
#查看项目中可用的数据


```

