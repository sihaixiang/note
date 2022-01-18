## 基于自带定位信息的几何校正

### MODIS

- 打开文件
- “几何校正”—“传感器地理参考”—“MODIS地理参考”
- 选择“Reflectance Meta File”，进行校正
- UTM—WGS-84，可以不用输出控制点，直接下一步
- 输出位置，完成

### 风云三号

- 打开HDF5文件，分别加载“表观反射率文件”和“经纬度”
- 建立GLT文件
  - x：Longitude
  - y：Latitude
  - 都选择Geo lat/lon
  - output Rotation设置为0，其余默认

- “几何校正”—“利用GLT地理参考”，使用生成好的glt文件和data进行几何校正

### ASAR

- 打开数据，右键默认合成显示
- “几何校正”—“传感器地理参考”—“ASAR地理参考”
- Geo lat/lon—no
- 选择Cubic convolution

## 图像配准

- 准备两幅图像：基准图像和待配准图像
- “几何校正”—“影像几何校正工作流程”
- 输入基准图像与待配准图像
- 默认设置—下一步—删除误差较大的点
- 设置输出路径

## 图像融合

图像融合，是将低分辨率的多光谱影像与高分辨率的单波段影像重采样生 成一副高分辨率多光谱影像遥感的图像处理技术 ，使得处理后的影像既有较高的空间分辨率，又具有多光谱特征。

### 不同传感器图像融合

- 两幅不同卫星同一地区的图像
- “图像融合”—“Gram-Schmidt图像融合”
- 分别选择低分辨率图像和高分辨率图像
- 传感器类型：其他 —重采样方法：双线性
- 设置输出路径

### 同一传感器图像融合

- 两幅同一传感器同一地区的图像
- “图像融合”—“Gram-Schmidt图像融合”
- 分别选择低分辨率图像和高分辨率图像
- 传感器类型：“所选卫星图像的类型” —重采样方法：双线性
- 设置输出路径

### NNDiffuse Pan Sharpening

- 支持众多传感器类型，如 Landsat 8 、 SPOT 、 WorldView 2/3 、 Pléiades 1A/1B 、QuickBird 、
  GeoEye 1 、EO 1 ALI 、IKONOS 、Duba iSat 1/2 、NigeriaSat 2 、国产 卫星等。
- 输入图像支持如下几种地理信息元数据类型：标准地理和投影坐标系统、具备 RPC 信
  息、基于像元位置（无空间坐标系）。
- 此工具支持多线程计算，从而实现高性能处理。
- 融合结果对于色彩、纹理和光谱信息，均能得到很好保留。
- （个人感觉融合后的分辨率比Gram-Schmidt图像融合的高）

## 图像镶嵌

图像镶嵌，指在一定数学基础控制下 把多景相邻遥感图像拼接成一个大范围、无缝的图像的过程。ENVI 的图像镶嵌功能 可 提供交互式的方式 ，将有地理坐标或没有地理坐标的多幅图像合并， 生成一幅单一的合成图像。



### Seamless Mosaic

- 控制图层的叠放顺序
- 设置忽略值、显示或隐藏图层或轮廓线、重新计算有效的轮廓线、选择重采样方法和
  输出范围、可指定输出波段和背景值
- 可进行颜色校正、羽化 调和
- 提供高级的自动生成接边线功能、也可手动编辑接边线
- 提供镶嵌结果的预览

### 步骤

- （1） 数据加载
- （2） 匀色处理
- （3） 接边线与羽化
- （4） 结果输出