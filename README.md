# FooProxy

评分制 IP代理池 + API服务提供，可以自己插入采集器进行代理IP的爬取，支持 MongoDB 4.0 使用 Python3.7 
## 背景
 因为平时爬取某些网站数据时，经常被封IP，同时网上很多的接口又不方便，免费的也少，稳定的更少，所以自己写了一个评分制的ip代理API进行爬虫的供给
 起初对MySQL和MongoDB进行了兼容的编写，后来发现在高并发的情况下，MySQL并不能很好的读写数据，经常莫名其妙的出现死机、读写巨慢、缓执行等各种奇
 葩现象，对比MongoDB高效的数据文档读写，最终还是放弃了mysql的兼容。*(dev分支保留了对mysql的部分支持，如爬取评分)*
## 环境  
> **开发环境**
* PyCharm 2018.2.4 (Professional Edition)
* Python 3.7
* MongoDB 4.0
* Windows 7 64bits
> **需安装的库**
* pymongo
* flask
* gevent
* requests
* bs4
* lxml
## 项目目录
> * APIserver
>>  一个简单的代理API接口服务器，使用Flask实现，可以自己按需求写路由逻辑。这部分当然可以独立出来写，只是集成写在了项目里面。
> * components
>> 项目的主要运行部分，采集器、验证器、打分检测等功能实现的模块。
> * **config**
>> 其中的DBsettings是数据库的设置，用户名密码之类的，以及存储的数据库名，还有备用有效数据库(standby)的自定义名字和高分稳定数据库(stable)的自定义名字。config文件是整个项目的主要参数配置，可以设置采集器采集间隔、验证器的抓取验证数量间隔和协程并发极值等。
> * const
>> 项目的静态配置，一般不用动的设置参数
> * **custom**
>> 自定义模块，可以编写自己要增加的爬虫采集函数到采集器中进行数据采集
> * log
>> 项目日志记录模块配置以及日志
> * tools
>> 一些工具函数
> * **main.py**
>> 项目入口文件
## 基本流程
整个项目的流程其实很简单，采集数据模块主要是编写代理网站的爬虫，可以进行任意的爬虫增减
* 采集数据
* 验证数据
* 打分存储
* 循环检测
* 择优剔劣
* API调用

 流程图：
![流程图](https://github.com/01ly/FooProxy/blob/dev/chart.png)
## 评分
> 简单的评分可以使代理ip筛选更加简单，其中的具体设置可以再const.settings中更改，一个代理IP数据的得分主要是：
> 1. 一次请求的基础分 score-basic ：100-10x(响应时间-1)
> 2. 请求成功的得分 score-success ： (基础分+测试总数x上一次分数)/(测试总数+1)+自定义成功加分数x成功率x连续成功数
> 3. 请求失败的得分 score-fail : (基础分+测试总数x上一次分数)/(测试总数+1)-自定义失败减分数x失败率x连续失败数
> 4. 稳定性 stability : 得分x成功率x测试数/自定义精度 

> 与三个变量成正比的稳定性根据得分设置可以很快的两极化稳定与不稳定的代理，从而进行筛选。
## 使用
* 确保本机安装MongoDB，并且下载好所有需要安装库 
* 可以先进行自定义的模式，在config中进行配置,可以运行单独的模块进行测试，如：
```
 #运行模式,置 1 表示运行，置 0 表示 不运行,全置 0 表示只运行 API server
  MODE = {
   'Collector' : 1,    #代理采集
   'Validator' : 1,    #验证存储
   'Scanner'   : 1,    #扫描本地库
   'Detector'  : 1,    #高分检测
 }
 ```
 * 按照自己需求更改评分量（const.setting中,默认不用更改）
 * 配置后可以直接运行**main.py**
 * 运行一段时间就可以看到稳定的效果
 ## 不足
 * 稳定性没有很好的标准判断，不过100次测试85%以上的成功率就已经很好了
 * 没有编写验证器与API服务器的超时请求代理功能
 * API 服务器没有单独拿出来编写
 * 还没有加入存活时间的考量
 * 还没接入爬虫测试
 * ...
 ## 效果
 1. **备用有效数据库**，开启1.5个小时后:
 ![备用有效数据库](https://github.com/01ly/FooProxy/blob/dev/pic/2018-10-09_2-07-47.png)
 2. **高分稳定数据库**
 ![高分稳定数据库](https://github.com/01ly/FooProxy/blob/dev/pic/2018-10-09_2-09-42.png)
 ## 后话
 * 经过连续两天的测试，程序运行正常
 * 只有一个数据采集爬虫的情况下，一个小时采集一次，一次最多1000条数据，10个小时内稳定的有效代理995左右，高分稳定的有200条左右，主要在于
  代理网站的质量。
