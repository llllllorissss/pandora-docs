### 产品简介

**Pandora 智能异常检测与预测** 是通过机器学习算法、深度学习算法，对历史时序数据自学习、自动为用户选择最优算法进行异常检测、预测与告警的分析产品，用户通过使用异常检测与预测得到的分析结果来指导自身的业务。

![机器学习](https://pandora-kibana.qiniu.com/nginx_abnormal.png)

智能异常检测与预测主要应用的场景有：

* IT 运维场景

   1. 通过流量异常检测进行系统安全分析，如 DNS/HTTP 数据泄露检测、可疑登陆检测等。
   
   2. 通过流量异常检测监控服务质量，如监控 nginx 服务响应变化发现服务问题。
   
   3. 通过流量异常检测监控机房运转情况，如监控 cpu 等指标探测系统问题。
   
* 医学场景：患者的异常身体指标检测，及时跟进患者身体状况。
* 营销场景：识别异常的购物车增加或减少行为，帮助衡量网站的转化情况。
* 金融类场景：股票走势预测。
* 其他：天气预测、比赛结果预测。

### 产品功能

* 对指定分析指标进行历史异常值的检测

通过配置分析指标，对指标进行历史异常值的检测，并给出每个异常点的详细信息。异常值包括突变的数据、超出波动范围的数据、周期波动的数据等，通过研究这些异常数据，及时发现业务请求数、拒绝数、响应时间、流水和订单等数据的异常波动，保证业务的稳定性。

* 对指定分析指标进行未来时间范围内的预测

配置分析指标，系统通过对历史时序数据的学习对未来的数据走势做出预测。如对网约车出行流量预测、股价走势预测、客户流失率预测等。用户可根据预测结果及时调整策略，做到有备无患。

### 产品优势

 Pnadora 智能异常检测与预测的主要优势如下：
 
 **友好的操作方式**
 
 通过对底层的算法进行封装，提供可视化的操作环境，用户通过页面可视化配置任务指标即可创建异常检测任务，无需编程语言，降低了用户的学习成本，让数据挖掘变成一件简单的事情。
 
 ![](https://pandora-kibana.qiniu.com/ml_interface.png)
 
 **自动选择最优算法**
 
 系统自动为用户选择的指标选择最优算法分析，用户无需任何算法相关知识即可完成异常检测与预测，得到分析结果，极大地降低了使用门槛。
 
 **优质的机器学习算法**
 
 提供诸如聚类、回归、神经网络等机器学习及深度学习算法，进行精准的异常检测与预测，准确度比传统机器学习算法高出很多。
 
 **与 Pandora 智能日志管理平台完美融合**
 
 机器学习直接基于 Pandora 智能日志管理平台存储的数据进行建模分析，无需复杂的导入导出。
 
### 快速入门

点击进入[机器学习](https://portal.qiniu.com/pandora/logdb)入口。通过如下图操作逻辑实现对历史异常值的检测以及指标数据的预测。

![](https://pandora-kibana.qiniu.com/ml_structure.png)

**异常检测**

创建一个对 nginx 日志的请求体指标进行异常检测的任务操作步骤如下：

1. 点击`创建任务`开始配置异常检测指标。

![](https://pandora-kibana.qiniu.com/create_task2.png)

2. 通过选择日志仓库、分析指标与字段、时间范围确定异常检测指标。

![](https://pandora-kibana.qiniu.com/create_nginx.png)

3. 点击`预览指标`查看指标的历史数据。

![](https://pandora-kibana.qiniu.com/create_nginx_task.png)

!>配置异常指标之后需要点击预览指标，系统将根据预览结果决定是否允许创建异常检测任务。预览无数据或者历史数据量不够均达不到创建异常检测条件。

4. 点击`创建任务`对所配指标进行异常检测。

5. 查看异常检测结果。

![](https://pandora-kibana.qiniu.com/nginx_abnormal.png)

**预测**

1. 系统对历史时序数据异常检测完成之后，在异常检测页面右上角点击`预测`，创建预测任务。

![](https://pandora-kibana.qiniu.com/create_nginx_forecast.png)

2. 查看所选指标预测结果。

 ![](https://pandora-kibana.qiniu.com/forecast_result1.png)
 
### 用户指南

#### 异常检测

##### 配置异常检测指标

指定需要分析的仓库、指标、字段(数值类型)、时间字段以及时间范围来确定需要分析的指标。

![](https://pandora-kibana.qiniu.com/create_nginx.png)

|配置项|解释|
|:--|:--|
|仓库|分析的日志所在的仓库|
|指标|需要分析的指标，目前支持计数， 求和|
|字段|目标分析字段|
|时间字段|若日志里有多个时间字段，选择感兴趣的时间字段来分析该字段的时序范围内的异常|
|时间范围|分析的时序范围|
|时间聚合粒度|时间聚合粒度|

##### 异常检测结果解析

![](https://pandora-kibana.qiniu.com/nginx_abnormal.png)

1. Pandora 异常检测结果包括如下几个信息：

   * 所选指标的预期值范围，包括上界值、下界值和期望值、实际值。

     实际值偏离预期范围的点即为异常点，异常点高亮标示。异常点偏离期望曲线越远，表示异常程度越严重。
   
   * 异常的严重程度与根据机器学习算法得出的分值一一对应。

      |异常程度|异常分值|
      |:--|:--|
      |非常严重|0.75～1.00|
      |严重|0.50～0.75|
      |中度|0.25～0.50|
      |轻度|0～0.25|  

2. 可以拖动时间轴对视图进行控制。

3. 异常列表显示此任务范围内的全部异常情况，可根据异常程度对异常结果进行筛选。

![](https://pandora-kibana.qiniu.com/ml_result_detail.png)

4. 通过分析异常结果来指导业务，保证业务的稳定性。具体来说，如图所示，通过对 pipeline_nginx 的请求体进行异常检测来查看 nginx 服务运转情况，与异常原因结合起来分析，是入口机器宕机、网卡损坏或是其他原因。通过回顾历史服务问题，包括异常频率、异常程度、异常原因来辅助业务上的决策。

#### 预测

##### 预测结果分析

![](https://pandora-kibana.qiniu.com/forecast_result1.png)

Pandora 预测结果包括如下几个信息：

所选指标的预期值范围，包括上界值、下界值和期望值。

预测通常给出数据未来一段时间的走势，预测结果是否满足预期，如果不满足，可能需要根据趋势对现有决策进行调整，防患于未然。预测功能提供给用户更多操作空间，避免问题来临时手忙脚乱。

### 场景分析

#### 服务监控

检测 nginx 服务响应时间异常，实时把握最新系统状态，及时发现问题，告别滞后、延误，提高服务的稳定性。

![](https://pandora-kibana.qiniu.com/nginx_monitor.png)

#### 机器性能监控

将机器的多种指标的异常检测结合起来分析，如 cpu 使用率、网络流量等来发现问题，若 cpu 使用率很高而流量很少，那么可能需要优化应用进程、对机器进行扩容来维持机器性能的稳定性。

![](https://pandora-kibana.qiniu.com/cpu_abnormal.png)

![](https://pandora-kibana.qiniu.com/bytes_monitor.png)

#### 机房资源需求预测

针对机器预测其磁盘未来负载情况，避免应用系统因出现存储容量耗尽的情况而导致应用系统负载率过高，最终引发系统故障。

![](https://pandora-kibana.qiniu.com/disk_monitor.png)

对机房而言，根据机房历史资源消耗预测机房资源需求走势，抢先一步指导机房扩缩容。











