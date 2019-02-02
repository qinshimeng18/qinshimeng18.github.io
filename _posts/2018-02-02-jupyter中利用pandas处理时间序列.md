

# 前言

## 目录
#### 1.Jupyter Notebook哪里舒服了
#### 2.Pandas常用的时间处理操作
#### 3.案例:适用于时延类指标异常检测的实时DBscan算法

## 1.Jupyter Notebook哪里舒服了
### 1.1 一些优点
1. 是一个交互式笔记本,网页版的IDE,便捷持久化输出和分享,支持多种语言如R\Spark等
2. 不需要用vim在服务器上编码了
> jupyter可以部署到服务器,通过设置的密码登录  
> 或者通过ssh建立一个本机和服务器的端口映射,端口映射命令```ssh -N -f -L localhost:8889:localhost:8888 remote_name```  
> 在本地浏览器可以进行:上传文件,下载导出为.html .md .pdf.py等,运行python,bash命令,交互式的可视化图像
3. 运行模式
    - 一个文件由多个cell组成,每个cell是一个可编辑执行和展示的小单元
    - 变量,函数,包 等都保存在内存中,不需要重复运行,可实时查看
    - 输出结果也会保留再页面上,便于预处理数据和展示,相互不影响
    - 最后一行会直接输出结果,免去print
4. plot出来的图片会保存在页面内,甚至可以进行选取平移等操作,支持markdown,支持github渲染,让分享更快捷
5. 有许多扩展插件:**Configurable nbextensions**(单独安装):
    - Variable Inspector:实时查看变量
    - Code prettify:格式化代码
    - Table of Contents:显示md的目录结构,类似书签
    - 一键隐藏代码或者输出结果等等
    - 冷冻部分代码不运行不可更改
6. 许多魔法命令,如
    - %magic:查看所有魔法命令
    - %%timeit : 测试整个单元中代码的执行时间
> !+linux命令 就可以直接运行,还用什么shell  
> ?pd.qcut() 显示相关函数的说明文档,附带sample  
> %run ./note.ipynb 可以直接运行其他的notebook,其渲染的图片也会显示在当前的shell中  
> %load ./hello_world.py 载入py文件的代码到当前cell里  
> %store data 将在不同的notebook间共享变量,输入%store -r data 即可加载  
> %%writefile pythoncode.py  可以把当前cell的内容保存到外部文件中区,%%pycat 可以查看  
> %prun some_useless_slow_function() 显示每个内部函数的耗时情况  
> LaTex公式:$$ P(A \mid B) = \frac{P(B \mid A) , P(A)}{P(B)} $$  
> %%bash %%ruby... 在不同cell开头加上声明,就可以运行不同内核的代码了  
> conda install -c r r-essentials 安装r等不同的内核命令
### 1.2 Notebook 页面介绍及使用
- **工具栏**:  
从左到右依次是 Cell的常规操作(移动,中断,重启);Cell类型(md还是运行的代码);隐藏当前cell的代码;全部隐藏;冰冻cell使其不能run或者change;监控变量;字的大小;书签展示;美化代码等
![image](http://note.youdao.com/yws/res/18374/3003C52BFC5D46E683BBD1C6C0E402A9)
- **目录** 这个插件我觉得非常的有用,值得推荐!  
你可以先用markdown写一下算法的大致流程,然后一个一个步骤的去完成,同时也会清晰的看到自己目前正在编写第几个步骤(黄色标识),点击即可跳转,想必都体验过一行一行找代码的痛苦吧  
![image](http://note.youdao.com/yws/res/18373/21E3390DE3A24461A0F2CEC78C8CA3F7)
- **主页面以及Cell**
> cell就是一个命令窗口,里面可以放markdown文本或者要执行的代码  
> 第一个是代码块,运行一次,结果很持久;后面是渲染的一个图,分享给别人打开就能看到图,不需要再次运行 
![image](http://note.youdao.com/yws/res/18372/3255DCE0CEAD43529CA34A33EC0704E2)
### 1.3 Tips:
- 两种状态:命令模式(Command Mode)与编辑模式(Edit Mode),编辑下可以ctrl-enter运行cell内的代码,或者Esc切换到命令状态;shitf-enter运行当前代码并移动到下一个cell
- esc后敲
    - s保存(可以回滚到最近一个保存的checkpoint);
    - m将cell变成一个markdown;
    - 数字变成md的Heading级别;
    - a在上方插入一个cell;
    - b在下方插入一个cell;
    - z回滚最近的操作;
    - l展示行数
- 创建的时候可以选择不同的kernel,比如py2,py3,R,spark等

## 2. Pandas常用的时间处理操作
- 基础操作
```python
# 生成
pd.DataFrame([1,2,3,4])
# 读取文件,json sql html...
pd.read_csv('a.csv')
# 写入文件
df.to_csv()
# 合并
pd.merge()
pd.concat()
# 移除重复数据（去重）
df.duplicated()
# df.map() df.apply() 可以将操作应用到一列或者每一个元素
# 选取
df[df['column'] == -1] = 1
```
- 更多内容看 [十分钟了解pandas](https://www.jianshu.com/p/edd373307f7f)
- 时间序列的一些操作
    
    - 时间戳->日期 + 时区转换
    ```python
    # 列转换 2.7需要加dt,localize  时间戳->日期的转换
    pd.to_datetime(back_time,unit='ms').dt.tz_localize('Asia/Shanghai')
    # 格式化 单个字符串没有dt
    pd.to_datetime(df['timestamp'], unit='s').dt.strftime('%Y-%m-%d %H:%M')
    #时区问题 单个字符串,convert
    pd.to_datetime(df['time_stamp'][1],unit='ms',utc=True).tz_convert('Asia/Shanghai')
    # 索引设置为时间datetime后,可以这么操作:
    df['2019-01']
    df['2019-01-24 01:41':'2019-01-24 01:43']
    time_stamp	value  
    2019-01-24 01:41:00+08:00	1548294060000 
    2019-01-24 01:42:00+08:00	1548294120000 
    2019-01-24 01:43:00+08:00	1548294180000 
    ```
    - 改变时间间隔
    ```
    df['time'].asfreq('45Min', method='pad')
    ```
    - 字符生成日期格式
    ```
    pd.Timestamp('2012-05-01')
    Timestamp('2012-05-01 00:00:00')
    ```
    - 时间重采样,粒度
    ```python
    # index 需要设置为timedate类型: 
    df.index = pd.to_datetime()
    df.resample('24H',how='count')
    ```
    - 时间的生成
    ```
    # 每个月末
    pd.date_range(start, end, freq='BM')
    # 每隔一周
    pd.date_range(start, end, freq='W')
    # end 往前数
    pd.bdate_range(end=end, periods=20)
    # start 往后数
    pd.bdate_range(start=start, periods=20)
    ```
    - 选取时间-isin:
    ```python 
    df['date'] = pd.date_range('2017-1-1', periods=30, freq='D')
    in_range_df = df[df["date"].isin(pd.date_range("2017-01-15", "2017-01-20"))]
    ...
    ```
    - 选取时间-between_time('23:00','00:00')
    ```python
    back_time.between_time('23:00','00:00')
    out:
    2018-12-19 23:29:00+08:00    1545262140000
    2018-12-21 23:55:00+08:00    1545436500000
    ```
    - 比较
    ```
    # 直接比较 datetime64 类型的column
    df_time['date']>'2018-12-19'
    ```

## 3. 案例: 适用于时延类指标异常检测的实时DBscan算法

情景描述:如下图,监控的时延类指标中,耗时经常有大有小不固定,没有交易量的时候,数值又是-1,这种情况很难去预测曲线的走势进行异常的告警,但是可以根据历史实时的计算出一个可信的上边界(类似DBSCAN,当某个'高值'范围内的点数大于min_sample,那么这个高值就是我们的实时阈值),下面的代码可以让我们计算出这个可信上边界.  

当然,这个边界的确定还要结合历史几天的高值来综合判断的,对于有周期性的曲线,还要按照不同的时间段来计算,这里只用修改的DBscan来处理一天的情况

首先,引入一些我常用的包
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns #可视化的一个包
import numpy as np
import warnings
warnings.filterwarnings('ignore') #为了整洁，去除弹出的warnings
#两个用一个,inline是普通模式,notebook可以提供可视化的交互操作
%matplotlib inline
%matplotlib notebook
```
数据内容:  
![image](http://note.youdao.com/yws/res/18386/207555E8AA19400DA55FC1F9797AD2F5)  

算法代码:
```python
min_sample,eps=3,50# 设置邻域内最少点和半径
up_point = df_detect.max() #默认上边界为最大值

df.index = df['time_stamp'] # 设置时间戳为索引
time_now = df.iloc[-1]['time_stamp'] # 当前时间戳
df_detect = df.loc[time_now-1440*60000:]['value'] # 过去一天的数据

for value in df_detect.sort_values(ascending=False): # 排序
    bin_count = int(((df_detect >= (value-eps)) &(df_detect <= value)).sum()) # 计算范围内的个数
    if bin_count > min_sample:
        up_point = value #计算出上边界即可
        break
print up_point

```
绿色的点是被认为的异常点
![image](http://note.youdao.com/yws/res/18381/202B2BBDDBEC40618E259C91D5BF0CBF)


> ---
> layout: mypost  
> title: 在jupyter中利用pandas处理时间序列  
> categories: [pandas,jupyter,时间序列]  
> ---  

layout: mypost
title: 在jupyter中利用pandas处理时间序列
categories: [pandas,jupyter,时间序列]