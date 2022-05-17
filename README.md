# -性能测试重点难点：1、场景分析与设计；2、性能测试数据准备；3、性能测试脚本开发；4、执行性能测试脚本，分析测试结果
性能测试的目的是：通过工具或代码模拟大量用户实现对服务器的并发访问来查找系统可能存在的性能瓶颈。
衡量指标：时间：软件的响应时间；空间：服务器的磁盘使用率，cpu使用率，内存空闲率等
# -分类：负载测试、压力测试、并发测试、配置测试，容量测试，基准测试，稳定性测试
负载测试：通过逐步增加系统负载，测试系统性能的变化，目的是找出系统在正常工作前提下的系统容量（最大用户数，最佳用户数），从而定位系统的瓶颈。
压力测试：通过集合点的方式确定在什么负载条件下系统处于失能状态。集合点是等待所有线程加载完毕，同时向服务器提交请求
并发测试：模拟多个用户同时访问同一个应用或接口，通常是为了测试指定功能，检测服务器是否会出现线程死锁，资源抢占，内存泄漏等情况
配置测试：通过运行一种或多种业务在一定的并发用户数量情况下，获得不同配置的性能指标，用于选择最佳的参数配置。目的是给项目部署环境提供最佳性能配置参照标准。
容量测试：在一定的软硬件及网络环境下，向数据库或硬盘中构造不同数量级别的数据记录，通过运行一种或多种业务在一定用户数量情况下，获取不同数据级别的服务器性能指标。目的是验证数据库，硬盘数据存储的容量最佳值
基准测试：在一定软硬件环境下，模拟少量的用户去请求服务器，用来检测服务器在少量用户访问时的基本性能情况。目的是为了后面的性能测试提供一个基准参考值，调试脚本验证脚本是否正常通过。
稳定性测试：通过给系统加载一定的业务压力（CPU资源在70-90%的使用率）的情况下，运行一段时间（72h）。目的：检查系统是否稳定。是否有异常表现（宕机，错误率飙升）
# - 性能测试常用指标
1、响应时间（Response Time，RT）
说明：用户从客户端发送请求到接收到服务端返回的结果所耗费的时间。
2-5-8原则：2秒以内为快，5秒可以接受，8秒不能接受。包含静态资源加载
平均响应时间：多个用户操作的响应时间的平均值。
接口响应时间：接口请求和获取响应的时间。一般是毫秒级。
2、每秒事务数（TPS，TransactionPer Second）
是指每秒钟服务器能处理的业务（事务）处理量。
事务：把一系列按照特定逻辑来执行的操作，作为一个整体来计算。事务四大特性：原子性，一致性，隔离性，持久性
面试常见问题：一个电商项目 ，每天的交易量达到1亿笔交易，求TPS？
提示：二八原则（百分之八十的事务发生在百分之二十的时间）
TPS = 1亿*80%/（24*3600）*20%
3、资源利用率
通常建议：cpu不高于80%，内存不高于80%，磁盘不高于90%
4、事务失败率
5、并发数
并发数计算方法：并发数 = TPS * 平均响应时间

# - 性能测试流程：分析（性能测试需求分析）--设计（性能测试方案设计）--实现（性能测试脚本开发，脚本调试与优化，性能测试场景设计）--执行（搭建性能测试环境，运行性能测试场景，收集整理测试数据）--分析（分析性能指标，诊断瓶颈，调整并回归测试）--测试报告

场景设计原则：1、主业务流程2、操作比较频繁的功能3、使用用户多的功能4、数据量大的功能5、按照需求来
# - 测试场景分类：
1、单场景压测
目的是找出单场景的最大并发数，确保单业务场景不存在性能瓶颈
测试策略：采用逐步加压的压测策略，每次记录压测结果。通过对比并发数与流量以及错误率的关系，找到一个最合理的系统可支撑最大并发用户数
操作时可以先把并发数往大增加，压出问题后在逐步减少到合理水平
测试时间：一般对于单场景测试，设置梯度加压策略，可以50线程数为一组递增，每组测试时间5-10分钟进行测试，并且在单场景压测记录文档中记录每次压测的相关数据结果

2、复合场景压测
在完成单场景性能测试的前提下，根据业务需求测试复合业务场景下的性能是否达到指定标准。
测试时间：直接按照性能需求分析中的要求进行测试，可以根据实际情况进行多轮测试，观察并记录性能测试结果。每一轮的时间大概运行5-10分钟

# 性能测试环境部署
真实的项目环境中，将应用服务器和数据库服务器分开部署，每个服务器单独进行监控

# 测试数据预埋
通过python操作数据库的方式进行构造。
'''
import pymysql

db = pymysql.connect(host='localhost',user='root',password='123456',database='mms')
cursor = db.cursor()

privilege = "信息查询功能，信息录入功能，信息浏览功能，数据报表功能"

for i in range(1,651):
  sql = "insert into user (uUsername,uPassword,uAccess)values(%s,%s,%s)"
  try:
    cursor.execute(sql,['python'+str(i),'123456',privilege])
    db.commit()
    print('插入成功')
  except Exception as e:
    db.rollback()
    print('插入失败，回滚'，e)
db.close()
'''




