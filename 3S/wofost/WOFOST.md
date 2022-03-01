### 概述

WOFOST (WOrld FOod STudies) 是一个模拟模型，用于定量分析一年生大田作物的生长和产量。

它是一种机械的、动态的模型，可以根据光合作用、呼吸作用等基本过程以及这些过程如何受环境条件影响来解释作物的日常生长。 

使用 WOFOST，您可以计算给定有关土壤、作物、天气和作物管理（例如播种日期）的知识的位置的可达到的作物产量、生物量、用水量等。WOFOST 已被世界各地的许多研究人员使用，并已应用于各种气候和管理条件下的许多作物。WOFOST 是[欧洲 MARS 作物产量预测系统的关键组成部分之一。](https://ec.europa.eu/jrc/en/mars)在全球产量差距图集 ( [GYGA](http://www.yieldgap.org/)) WOFOST 用于根据当前气候和可用的土壤和水资源估计现有农田未开发的作物生产潜力。

### 模型

 

|         Name          |                         Description                          |
| :-------------------: | :----------------------------------------------------------: |
|      Wofost72_PP      |             针对潜在生产场景的 WOFOST 7.2 实施。             |
|    Wofost72_WLP_FD    |   WOFOST 7.2 的实施，用于具有自由排水土壤的限水生产方案。    |
|   Wofost80_PP_beta    |   WOFOST 7.2 的实施，用于具有自由排水土壤的限水生产方案。    |
| Wofost80_WLP_FD_beta  | WOFOST 8.0 用于限水生产场景的实施，包括用于自由排水土壤的 N/P/K 动态。 |
| Wofost80_NWLP_FD_beta | WOFOST 8.0 的实施，适用于水受限和养分受限的生产场景，包括自由排水土壤的 N/P/K 动态。 |
|        LINTUL3        |     LINTUL3 模型在限水和限氮生产情景下的生产情景的实现。     |
|       FAO_WRSI        | 用水需求满意度指数模型的实现。这重新使用了 WOFOST 的组件来创建一种更简单的方法来计算用水需求和可用水量。 |
|  Wofost72_Phenology   | WOFOST 7.2 中的物候模块作为独立模型。这纯粹是为了方便，因为在某些情况下运行物候学就足够了，而且这比运行完整的 WOFOST 模型要快得多。 |

### 运行所需文件

- 参数文件
  - 作物参数（Crop parameters）
  - 土壤参数（Soil parameters）
  - 场地参数（Site parameters）

- 农业管理文件（AgroManagement）
- 天气数据（Daily weather observations）



### 入门

```python
import pcse
from pcse import start_wofost
import pandas as pd

#创建一个 WOFOST 对象，用于在自由排水的土壤（模式='wlp'）的限水条件下，在西班牙南部（网格 31031）的一个位置（网格 31031）上模拟冬小麦（作物=1）
wofost_object = pcse.start_wofost(grid=31031, crop=1, year=2000, mode='wlp')

wofost_object.run(days=10)  #表示第11天的数据（从0开始）
a = wofost_object.get_variable('LAI')   #当前状态下的叶面积指数（11天）
#print(a)

#继续运行,直到模型因作物成熟或收获日期而终止来结束作物季节
wofost_object.run_till_terminate()

#输出模拟结果
output = wofost_object.get_output()   

#使用 pandas 包将模拟结果转换为更易于处理并且可以导出为不同文件类型的数据
df = pd.DataFrame(output)
df.to_csv("wofost_result.csv")      #将数据保存为CSV文件
```

<img src="https://s2.loli.net/2022/02/28/AsdMYhwKprvtCU1.png" alt="image-20220228163808671" style="zoom:50%;" />