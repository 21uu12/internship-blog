---
title: 今日实习学习沉淀：资金域表理解与 DolphinScheduler 入门
date: 2026-06-22 18:00:00
categories: 每日记录
tags:
  - 实习
  - 大数据
  - 资金域
  - DolphinScheduler
  - 数据仓库
---

> 主题：资金域 / 资产域核心表理解 + DolphinScheduler 调度任务实践

---

## 1. 今天学习的主线

今天主要围绕两个方向展开：

1. **资金域 / 资产域业务理解**
   - Asset Info 表
   - Asset Per Info 表
   - Delay 延期表
   - 借款、放款、还款、延期、结清之间的业务关系

2. **DolphinScheduler 海豚调度入门**
   - 创建项目
   - 创建工作流
   - 添加 Shell 任务
   - 手动运行任务
   - 查看任务日志
   - 判断任务是否执行成功

今天的重点不是单纯记字段，而是开始建立：

> 表结构 → 业务流程 → 调度链路 → 数据产出

这一整套数据开发思维。

---

## 2. 资金域 / 资产域的业务理解

资金域，也可以理解为资产域，主要关注的是：

> 用户借款之后，钱有没有放出去、应该还多少、实际还了多少、有没有延期、是否结清或逾期。

它和用户域、申请域的区别在于：

- 用户域更关注用户是谁、用户属性是什么；
- 申请域更关注用户有没有申请、申请是否通过；
- 资金域 / 资产域更关注借款真正形成之后的资金流转和还款状态。

一个简化的业务流程可以理解为：

```text
用户注册
  ↓
提交借款申请
  ↓
申请审核通过
  ↓
生成资产 / 借据
  ↓
放款成功
  ↓
生成还款计划
  ↓
用户按期还款 / 延期 / 逾期
  ↓
最终结清
```

今天重点学习的是从“生成资产 / 借据”之后的部分。

---

## 3. Asset Info 表：资产主表 / 借据主表

### 3.1 表的核心定位

`Asset Info` 可以理解为：

> 一笔借款的主记录。

它通常是一笔借款一行，记录这笔借款从生成、放款、到期、还款、结清整个生命周期中的核心信息。

### 3.2 业务粒度

常见粒度：

```text
一笔资产 / 一笔借据 一行
```

也就是说，如果一个用户借了 1 次钱，一般会在资产主表里形成 1 条资产记录。

### 3.3 主要关注字段

常见字段包括：

```text
asset_id：资产ID / 借据ID
asset_apply_id：申请ID
user_id：用户ID
grant_time：放款时间
due_time：到期时间
finish_time：结清时间
asset_status：资产状态
contract_principal_amt：合同本金
granted_principal_amt：实际放款本金
repaid_principal_amt：已还本金
interest_amt：应还利息
repaid_interest_amt：已还利息
fee_amt：费用
repaid_fee_amt：已还费用
```

### 3.4 可以回答的业务问题

Asset Info 表适合回答：

```text
今天放款了多少金额？
今天有多少笔资产生成？
某个用户有几笔借款？
某笔借款是否结清？
某笔借款的本金、利息、费用分别是多少？
某天到期的资产有多少？
```

### 3.5 一句话记忆

> Asset Info 是资产主表，用来看整笔借款的生命周期状态。

---

## 4. Asset Per Info 表：资产期次表 / 还款计划表

### 4.1 表的核心定位

`Asset Per Info` 可以理解为：

> 一笔借款拆成多期之后，每一期应该怎么还。

如果一笔借款分 3 期，那么：

- Asset Info 可能只有 1 行；
- Asset Per Info 可能有 3 行。

### 4.2 业务粒度

常见粒度：

```text
一笔资产的一期还款计划 一行
```

例如：

```text
asset_id = A001，第1期，一行
asset_id = A001，第2期，一行
asset_id = A001，第3期，一行
```

### 4.3 和 Asset Info 的区别

可以这样区分：

```text
Asset Info：看整笔借款
Asset Per Info：看每一期还款
```

举例：

用户借款 3000 元，分 3 期：

```text
Asset Info：
A001，借款3000元，分3期，当前状态还款中

Asset Per Info：
A001，第1期，应还1000元
A001，第2期，应还1000元
A001，第3期，应还1000元
```

### 4.4 可以回答的业务问题

Asset Per Info 表适合回答：

```text
某笔借款第几期到期？
某一期应还多少钱？
某一期实际还了多少钱？
某一期是否已经结清？
某一期是否逾期？
未来几天有多少期应还？
```

### 4.5 一句话记忆

> Asset Per Info 是期次表，用来看每一期应该还多少、实际还多少、是否完成。

---

## 5. Delay 表：延期表

### 5.1 表的核心定位

`Delay` 表主要记录：

> 用户对某笔资产或某一期还款进行延期的行为。

它不是资产主表，也不是还款计划表，而是围绕“延期”这个动作产生的业务记录。

### 5.2 业务场景

用户原本应该在某天还款，但因为暂时无法还款，选择支付延期费用，把还款日往后推。

简化流程：

```text
原到期日
  ↓
用户申请延期
  ↓
支付延期费用
  ↓
生成新的延期到期日
  ↓
后续继续还款或再次延期
```

### 5.3 常见字段

常见字段包括：

```text
asset_id：资产ID
user_id：用户ID
delay_seq_cnt：延期次数
due_time：原到期时间
delay_due_time：延期后到期时间
delay_pay_time：延期支付时间
delay_finish_time：延期完成时间
fee_amt：延期费用
repaid_fee_amt：已支付延期费用
```

### 5.4 可以回答的业务问题

Delay 表适合回答：

```text
哪些用户发生了延期？
某笔资产延期了几次？
原到期日是什么？
延期后到期日是什么？
延期费用是多少？
延期是否支付成功？
延期后是否继续逾期？
```

### 5.5 一句话记忆

> Delay 表是延期行为表，用来看用户有没有把某期还款往后推。

---

## 6. 三类表之间的关系

今天可以先按照下面这个关系理解：

```text
Asset Info 资产主表
  ↓ 1 对多
Asset Per Info 资产期次表
  ↓ 发生延期时产生记录
Delay 延期表
```

更业务化地说：

```text
一笔借款形成一条 Asset Info
一笔借款如果分多期，会形成多条 Asset Per Info
某一期如果延期，会在 Delay 表中产生延期记录
```

简化关系：

```text
asset_info  1 : N  asset_per_info
asset_per_info  1 : N  delay
```

实际公司表设计可能会因为业务不同有所差异，但这个理解适合作为入门框架。

---

## 7. DolphinScheduler 海豚调度学习

### 7.1 核心理解

DolphinScheduler 可以理解为：

> 大数据任务的调度平台，用来把 Shell、Python、Hive、Spark、Flink、DataX、StarRocks 等任务按依赖关系串起来，并定时自动执行。

它不负责真正计算数据，真正计算的是 Hive、Spark、Flink、StarRocks 等组件。

DolphinScheduler 负责的是：

```text
什么时候跑
先跑哪个
后跑哪个
失败了看哪里
失败后怎么重跑
每天是否自动执行
```

### 7.2 今天完成的操作

今天已经完成了第一个 Shell 工作流任务：

```bash
echo "hello dolphinscheduler"
```

任务日志中出现：

```text
hello dolphinscheduler
exitStatusCode:0
processExitValue:0
```

说明任务执行成功。

### 7.3 如何判断任务成功

重点看：

```text
exitStatusCode:0
processExitValue:0
```

在 Linux 和调度系统中：

```text
退出码 0 = 成功
非 0 = 失败
```

所以今天这个任务已经跑通了。

---

## 8. DolphinScheduler 的实际价值

海豚调度的价值不是只执行一个 `echo`，而是可以把真实的数据链路组织起来。

例如：

```text
1. DataX 同步 MySQL 原始业务表到 Hive ODS
        ↓
2. Hive SQL 清洗 ODS，生成 DWD 明细表
        ↓
3. Spark / Hive 计算 DWS 汇总表
        ↓
4. StarRocks 写入 ADS 报表表
        ↓
5. Python 做数据质量校验
        ↓
6. 失败时发送告警
```

这就是从“手动跑脚本”升级为“自动化数据生产链路”。

---

## 9. 对“一劳永逸”的修正理解

今天也明确了一个点：

> DolphinScheduler 不是让任务永远不用管，而是让日常重复执行自动化。

真实工作中仍然需要维护：

```text
上游数据是否正常到达
SQL 是否因为字段变化而报错
任务是否超时
Hive / Spark 是否资源不足
StarRocks 导入是否失败
指标口径是否变化
任务失败后是否需要补数
```

所以更准确的理解是：

```text
以前：手动执行脚本
现在：设计和维护自动化调度流程
```

这也是数据开发实习中很重要的能力。

---

## 10. 今天形成的核心认知

今天最重要的沉淀是：

### 10.1 业务链路

```text
申请通过
  ↓
生成资产 Asset
  ↓
生成还款计划 Asset Per
  ↓
到期还款
  ↓
可能延期 Delay
  ↓
结清 / 逾期
```

### 10.2 数据链路

```text
ODS 原始数据
  ↓
DWD 明细清洗
  ↓
DWS 汇总宽表
  ↓
ADS 报表指标
  ↓
BI 展示
```

### 10.3 调度链路

```text
Shell / Hive / Spark / Python / StarRocks 任务
  ↓
DolphinScheduler 工作流编排
  ↓
定时自动执行
  ↓
日志排查和失败重跑
```

---

## 11. 明天可以继续追问的问题

明天可以围绕这几个问题继续学习：

```text
1. Asset Info、Asset Per Info、Delay 表分别来自哪些 ODS 原始表？
2. 这些表每天是由哪个 DolphinScheduler 工作流产出的？
3. 工作流中 ODS、DWD、DWS、ADS 的节点分别叫什么？
4. 如果某一天数据缺失，应该从哪个节点开始补数？
5. Hive 到 StarRocks 是通过 SQL、DataX、Spark，还是其他方式同步？
6. 资金域最终给 BI 提供了哪些核心指标？
```

---

## 12. 今日总结

今天完成了一个比较关键的转变：

> 从只看单张表，开始转向理解业务域、表关系、数据分层和调度链路。

今天掌握的关键词：

```text
资金域
资产域
Asset Info
Asset Per Info
Delay
借据
放款
到期
还款计划
延期
结清
DolphinScheduler
工作流
任务节点
Shell 任务
任务日志
退出码 0
调度依赖
```

后续学习建议：

> 每学一张表，都要问三个问题：  
> 1. 这张表的业务粒度是什么？  
> 2. 它的上游来自哪里？  
> 3. 它的下游服务什么指标或报表？
