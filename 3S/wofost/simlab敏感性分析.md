```python
from pcse.fileinput import CABOFileReader
from pcse.fileinput import YAMLAgroManagementReader
from pcse.models import Wofost71_WLP_FD
from pcse.base import ParameterProvider
from pcse.fileinput import ExcelWeatherDataProvider
import datetime
import os
import progressbar  # 需要按装包 pip install progressbar2
import pandas as pd


# simlab输出的参数读取
para_dir = r'C:\Users\eat_meat\Desktop\敏感性分析'  #simlab输出文件的位置
# 模拟的位置
lat = 32.5   #维度
lon = 114.5 #经度
# 更改参数列表
# sow_date = dict(zip([i+1 for i in range(30)],[datetime.date(2019,10,i+1) for i in range(30)] ))
change_data = {'TDWI':0,'LAIEM':1,'RGRLAI':2,'SPAN':6}
#  读取模型参数
weatherdataprovider = ExcelWeatherDataProvider(os.path.join(para_dir, "NASA天气文件lat={0:.1f},lon={1:.1f}.xls".format(lat,lon)))
cropdata = CABOFileReader(os.path.join(para_dir,'WWH101.CAB'))
soildata = CABOFileReader(os.path.join(para_dir,'EC3.NEW'))
sitedata = {'SSMAX'  : 0.,
    'IFUNRN' : 0,
    'NOTINF' : 0,
    'SSI'    : 0,
    'WAV'    : 20,
    'SMLIM'  : 0.03,
    'CO2'    : 360,
   'RDMSOL'  : 120}
parameters = ParameterProvider(cropdata=cropdata, soildata=soildata, sitedata=sitedata)
agromanagement = YAMLAgroManagementReader(os.path.join(para_dir,'wheat_none.agro'))


#  创建文档储存模型数值结果
with open(os.path.join(para_dir,'总结果.txt'),'w') as fp3:
    fp3.writelines(['2', '\n', 'TWSO', '\n', 'TAGP','\n', 'time = no', '\n'])
    with open(os.path.join(para_dir,'按生育期结果.txt'), 'w') as fp2:
        fp2.writelines(['1', '\n', 'LAI', '\n', 'time = yes', '\n'])

        #  打开simlab输出的文档（事先在simlab中设置好）（根据PARA.FAC文件导出）
        with open(os.path.join(para_dir, 'PARA.sam'), 'r') as fp:
            fp.readline()  # 第一行
            number = fp.readline()  # 第二行 生成参数的个数(455)
            fp.readline()  # 第三行 变量个数(7)
            fp.readline()  # 第四行 0  第四行之后开始读参数
            fp2.write(str(number))   #number = 455
            fp3.write(str(number))
            for i in progressbar.ProgressBar()(range(int(number))):  #进度条
                sim_paraments = list(map(float,fp.readline().split('\t')[:-1]))
                # 更改参数（将先前的参数修改为模拟参数）
                for iterm,value in change_data.items():
                    parameters[iterm] = sim_paraments[value]
                parameters['SLATB'][1]=sim_paraments[3]
                parameters['SLATB'][3]=sim_paraments[4]
                parameters['SLATB'][5]=sim_paraments[5]
                
                wf = Wofost71_WLP_FD(parameters, weatherdataprovider, agromanagement)
                wf.run_till_terminate()    #运行wofost模型

                summary_output = wf.get_summary_output()
                output = wf.get_output()   #结果

                date=pd.date_range(start=summary_output[0]['DOE'], end=summary_output[0]['DOM'], freq='D')
                df = pd.DataFrame(output).set_index("day")
                fp2.writelines(['RUN',' ',str(i),'\n'])
                fp2.write(str(len(date)))   #196天
                fp2.write('\n')
                fp3.writelines([str(summary_output[0]['TWSO']), '\t', str(summary_output[0]['TAGP'])])
                fp3.write('\n')
                number = 0
                for j in date:
                    number += 1
                    fp2.writelines([str(number),' ' ,str(df.loc[df.index == j, 'LAI'][0]),'\n'])

```







###  the-sensitivity-analysis-of-WOFOST-with-simlab

the file of SA3.0_all in_1.py is the main program. the dir of the file need change by yourself.
.NEW is soil file, .CAB is crop file, .agro is management file, .SAM is the parameters that generated from simlab.

SA3.0_all in_1.py 是主程序。文件的目录需要自己更改。
.NEW是土壤文件，.CAB是作物文件，.agro是管理文件，.SAM是simlab生成的参数。





### 步骤

- 利用PARA.fac文件生成PARA.sam文件
- 程序调用sam文件，生成所需要的数据（总结果.txt    按生育期结果.txt）
- 利用simlab软件执行模型，统计处理数据