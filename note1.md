# 2025/7/11

周报：

本周:

1.对设备流量消耗进行采样，获取流量消耗参考数值，服务设备流量异常消耗方案执行。

2.处理运维平台IPD中试反馈，充电记录等数据无法写入的问题

# 2025/7/9

AAAAAMWY 898608421024C0947762

剩余流量

| 日期     | 使用流量 |      |
| -------- | -------- | ---- |
| 2025/7/2 | 5.141    |      |
| 2025/7/3 | 5.024    |      |
| 2025/7/4 | 4.787    |      |
| 2025/7/5 | 4.541    |      |
| 2025/7/6 | 4.376    |      |
| 2025/7/7 | 5.01     |      |
| 2025/7/8 | 4.219    |      |



# 2025/7/5

https://vercel.com/

https://bolt.new/

# 2025/7/4

本周：

1.完成运维平台软件详细设计说明书收尾

2.安装java相关环境，运行java版运维平台

下周：

# 2025/7/1

- 以下是为选中函数 `station_distribution()` 生成的详细接口文档，基于您提供的模板格式。

  ------

  ## 接口名称：`station_distribution()`

  ### 接口类型：

  POST

  ### 接口路径：

  ```
  /organization/station/distribution
  ```

  ### 功能说明：

  该接口用于**分配场站权限给指定组织及其子组织或上级组织**。其主要功能包括：

  - 支持向下递归删除多余的场站；
  - 支持向上递归添加新的场站；
  - 根据用户选择的目标场站与当前场站的差异进行增量更新；
  - 记录操作日志。

  ------

  ### 输入参数结构体：

  #### 结构体名：`SelectOrganizationStation`

  | 字段名          | 类型     | 是否必填 | 默认值 | 描述                   |
  | :-------------- | :------- | :------- | :----- | :--------------------- |
  | organization_id | u64      | 是       | -      | 要分配场站的组织 ID    |
  | sta_ids         | Vec<u64> | 是       | -      | 需要分配的场站 ID 列表 |

  示例请求体：

  ```
  json{
    "organization_id": 1001,
    "sta_ids": [2001, 2002, 2003]
  }
  ```

  ------

  ### 输出参数结构体：

  成功响应示例：

  ```
  json{
    "status": 1,
    "msg": "成功",
    "data": "程序运行成功了"
  }
  ```

  失败响应示例：

  ```
  json{
    "status": 0,
    "msg": "操作记录生成异常",
    "data": 0
  }
  ```

  ------

  ### 数据结构定义：

  ##### `WriteOrganizationStation`（组织-场站关系写入结构）

  | 字段名          | 类型 | 描述                         |
  | :-------------- | :--- | :--------------------------- |
  | timestamp       | i64  | 操作时间戳                   |
  | flag_del        | u8   | 删除标志位（0:有效, 1:删除） |
  | organization_id | u64  | 组织唯一标识                 |
  | sta_id          | u64  | 场站唯一标识                 |

  ##### `OrganizationStationList`（返回结果结构）

  | 字段名          | 类型   | 描述         |
  | :-------------- | :----- | :----------- |
  | organization_id | u64    | 组织唯一标识 |
  | sta_id          | u64    | 场站唯一标识 |
  | sta_name        | String | 场站名称     |

  ------

  ### 实现逻辑说明：

  1. **获取组织上下文关系**：
     - 向上递归查找组织层级 (`tool_org_up`)；
     - 向下递归查找组织层级 (`tool_org_down`)；
  2. **处理目标场站和原场站的差异**：
     - 将用户传入的 `sta_ids` 转换为 `HashSet` 用于集合运算；
     - 查询当前组织已有的场站并转换为 `HashSet`；
     - 计算需要删除的场站（原场站 - 目标场站）；
     - 计算需要新增的场站（目标场站 - 原场站）；
  3. **执行数据变更**：
     - 对于需要删除的场站，向下层组织插入 `flag_del=1` 的记录；
     - 对于需要新增的场站，向上层组织插入 `flag_del=0` 的记录；
  4. **记录操作日志**：
     - 调用 `add_operation_record()` 方法记录本次分配操作。

  ------

  ### 异常处理说明：

  | 错误情况         | 返回示例                                                |
  | :--------------- | :------------------------------------------------------ |
  | 操作记录生成异常 | `{ "status": 0, "msg": "操作记录生成异常", "data": 0 }` |
  | 分配失败         | `{ "status": 0, "msg": "增加操作失败", "data": 0 }`     |

  ------

  ### 示例请求：

  #### 请求地址：

  ```
  POST /organization/station/distribution
  ```

  #### 请求体：

  ```
  json{
    "organization_id": 1001,
    "sta_ids": [2001, 2002, 2003]
  }
  ```

  #### 成功响应示例：

  ```
  json{
    "status": 1,
    "msg": "成功",
    "data": "程序运行成功了"
  }
  ```

  #### 失败响应示例：

  ```
  json{
    "status": 0,
    "msg": "操作记录生成异常",
    "data": 0
  }
  ```

  ------

  ### 权限控制说明：

  - 接口调用者需具备对目标组织的操作权限；
  - 通过 `Claims` 和 `AuthSession` 等机制确保只有授权用户可以调用此接口；
  - 所有变更均记录操作日志以供审计。

  ------

  ### 附加说明：

  - **软删除机制**：所有变更都通过 `flag_del` 字段标记是否删除；
  - **递归操作**：支持组织层级的上下级联动操作；
  - **幂等性设计**：重复提交相同场站列表不会产生副作用；
  - **事务保障**：所有数据库操作应尽可能保证原子性，避免部分写入导致数据不一致；
  - **性能优化**：批量写入减少数据库交互次数，提升效率。
  
  以上述接口设计文档为模版，生成选中代码函数对应的详细接口文档

# 2025/6/20

本周：

1. 运维平台运维工作：运维平台响应慢问题排查与处理、运维平台异常充电桩的处理、充电桩新版本运维模块程序接入平台后的数据监测、欧标桩平台接入问题反馈的处理。
2. 定位充电桩连接运维平台上送充电记录时，vin字段乱码的充电记录重复上送的原因，并初步制定解决方案。
3. IPD关键器件功能与桩对接的部分进行联调，IPD启动失败记录功能入库模块相关代码的开发与对桩联调。关键器件、启动失败列表前后端联调。

下周：1.配合中试进行IPD内容测试。2.对中试发现的问题进行解决

```sql
select sum (switching_cycles) as a, sum(abnormal_switching_cycles) as b from key_component_action_count
```



# 2025/6/19

```sql
select t.timestamp as timestamp,
        t.dt_id as dt_id,
        t.device_id as device_id,
        trade_id,
        device_zone,
        app_time,
        gun_num,
        failed_code,
        charge_way,
        vin,
        start_soc,
        ass_name,
        a.sta_id as sta_id,
        s.sta_name as sta_name
        from boot_failed_record as t 
        inner join (select * from asset final where flag_del=0) as a on t.device_id=a.device_id 
        inner join (SELECT * FROM station FINAL WHERE flag_del=0 and sta_id in(select sta_id from organization_station final where flag_del =0 and organization_id =
            (select organization_id from organization_user final where flag_del =0 and user_id =1)) ) as s on a.sta_id=s.sta_id 
```



# 2025/6/18

```json
/xjdy/ops/v1/TCU/AAAAAJQX/msg 
{"命令":"启动失败原因","reqid":1,"数据分区":"1枪数据","系统时间":"2025-06-18 19:08:15+0800","产生时间":"2025-06-18 19:08:15+0800","交易流水号":"00000000000000002506181906190401","枪号":"1","车辆VIN码":"","充电方式":"密码","失败原因":"0X68:车辆原因-BRM报文接收超时"}

```

```sql
--启动失败原因 数据表
DROP TABLE IF EXISTS `boot_failed_record`;
CREATE TABLE IF NOT EXISTS sems.boot_failed_record (
`timestamp` Int64,
`dt_id` UInt64,
`trade_id` String,--交易流水号
`device_id` UInt64, --设备id
`device_zone` String,--数据分区
`gun_num` UInt8,--枪编号
`app_time` String,--系统时间
`generate_time` Int64,--开始时间
`failed_code` String,--失败原因
`charge_way` String,--充电方式
`start_soc` UInt8,--起始soc
`vin` String,--车辆vin
) ENGINE =  MergeTree  ORDER BY ( timestamp , trade_id ) PRIMARY KEY (timestamp, trade_id);
```



# 2025/6/17

```json
//目前桩上送格式
{"命令":"关键器件动作应答","系统时间":"2025-06-17 16:22:35+0800","关键器件名称":
 [
 {"数据分区":"1枪数据","器件类型":"直流输出接触器","正常次数":"1","异常次数":"0",
 "数据分区":"1枪数据","器件类型":"交流接触器","正常次数":"1","异常次数":"0"},
 {"数据分区":"2枪数据","器件类型":"直流输出接触器","正常次数":"1","异常次数":"0",
 "数据分区":"2枪数据","器件类型":"交流接触器","正常次数":"1","异常次数":"0"}]}
//实际应该上送格式
{"命令":"关键器件动作应答","系统时间":"2025-06-17 16:22:35+0800","关键器件名称":
 [
 {"数据分区":"1枪数据","器件类型":"直流输出接触器","正常次数":"1","异常次数":"0"}，
 {"数据分区":"1枪数据","器件类型":"交流接触器","正常次数":"1","异常次数":"0"},
 {"数据分区":"2枪数据","器件类型":"直流输出接触器","正常次数":"1","异常次数":"0"},
 {"数据分区":"2枪数据","器件类型":"交流接触器","正常次数":"1","异常次数":"0"}
]}
```



# 2025/6/10

关键器件逻辑进行调整，由平台定时下发，充电桩回复，平台收到回复后，二次与桩交互回复信息确认帧。

关键器件数据分区进行整合，同一条命令下发，只回复一条数据。

# 2025/6/9

```bash
scp xjj:/home/lcg/zyl_personal_file/warn_class_def.csv /c/Users/16167/Desktop
scp xjj:/home/lcg/zyl_personal_file/warn_level.csv /c/Users/16167/Desktop
scp xjj:/home/lcg/zyl_personal_file/warn_level_class.csv /c/Users/16167/Desktop
scp xjj:/home/lcg/zyl_personal_file/warn_level_def.csv /c/Users/16167/Desktop
scp xjj:/home/lcg/zyl_personal_file/warn_name_table.csv /c/Users/16167/Desktop
```





/home/lcg/zyl_personal_file

warn_class_def

warn_level

warn_level_class

warn_level_def

warn_name_table

```shell
# 导出 warn_class_def 表
clickhouse-client --query="SELECT * FROM sems.warn_class_def FORMAT CSV" > /home/lcg/zyl_personal_file/warn_class_def.csv

# 导出 warn_level 表
clickhouse-client --query="SELECT * FROM sems.warn_level FORMAT CSV" > /home/lcg/zyl_personal_file/warn_level.csv

# 导出 warn_level_class 表
clickhouse-client --query="SELECT * FROM sems.warn_level_class FORMAT CSV" > /home/lcg/zyl_personal_file/warn_level_class.csv

# 导出 warn_level_def 表
clickhouse-client --query="SELECT * FROM sems.warn_level_def FORMAT CSV" > /home/lcg/zyl_personal_file/warn_level_def.csv

# 导出 warn_name_table 表
clickhouse-client --query="SELECT * FROM sems.warn_name_table FORMAT CSV" > /home/lcg/zyl_personal_file/warn_name_table.csv
```

本周

1. 完成后端代码和充电桩、前端页面的参数配置、sim卡召唤、OTA等功能的联调，并根据联调情况进行协议和代码修改。
2. 完成关键器件动作次数相关接口的编写。

下周

1. 关键器件动作次数相关接口的联调
2. 桩禁用、关闭运维功能等其他OTA的联调

# 2025/6/6

工作情况

1、6月5日，运维运维平台测试服务器，为中试提供测试环境。--张延龙

2、6月6日，完成关键器件功能块，需求分析、业务梳理，数据库设计，接口文档编写。--马占宾、张延龙

3、6月6日，初步完成家充桩app对桩的页面设计及数据交互调试。--王江波

4、6月5日，前后端联调固件、升级、重启、参数配置等相关逻辑。--吕舟、马占宾、张延龙、丁孟超

5、6月6日，开发站信息总览页面，未进行接口对接。--马占宾

6、6月6日，工单模块前后端联调，前端调试工单暂存、查询、提交、删除、搜索等接口及其他工单相关业务逻辑、后端端根据联调情况修正接口。--王江波、吕舟

7、6月5日，充电产品三个指标页面开发完成。--丁孟超

计划

1、6月13日，完成站监控页面开发，运维平台自测、配合桩调试。--马占宾

2、6月13日，联调工单的审核、接单、拒单、完成、验收、子工单、设备更换、维护记录等功能。--吕舟

3、6月13日，完成关键器件接口开发任务收尾工作，开发站监控相关接口。--张延龙

4、6月13日，完成家充桩app开发调试的收尾工作，根据情况补充运维平台前端开发或后端开发工作余量。--王江波

# 2025/65

test服环境安装

1. git远程登录
2. emqx
3. clickhouse
4. redis
5. ftp
6. webserver、ops_tcu、connector、webpage
7. 防火墙端口开放
8. clickhouse数据迁移

# 2025/6/3

关键器件数据库表设计

+ 关于工单调用关键器件记录-增接口，字段：数量，应该如何处理？
+ 关于关键器件记录，返厂确认是否存在按钮，如果存在需要增加新的返厂确认接口?
+ 关于关键器件记录，关键器件的处理方式字段意义不太情绪，需要做二次讨论?
+ 工单状态为结束时调用关键器件相关逻辑吗？还是分为不同状态调用，因为需要至少调用两个不用的接口，关键器件信息接口、关键器件更换记录接口。

表1:关键器件信息表 

```sql
DROP TABLE IF EXISTS `key_component`;--关键器件表
CREATE TABLE IF NOT EXISTS sems.key_component(
`timestamp` Int64, -- 时间戳
`flag_del` UInt8, -- 删除标志 (0: 未删除, 1: 已删除)
`device_id` UInt64,-- 设备id
`data_sources` String, --数据来源
`workorder_id` UInt64, --工单号,*和新版工单数据库对齐
`key_component_id` UInt64, --关键器件唯一标识
`attribute` String, --属性->关键器件/组成器件
`key_component_name` String,--名称->桩内同类型关键器件唯一性校验
`key_component_type` String,--类型->直流接触器、断路器、熔断器
`sap` String,--sap该数据为非必填
`supplier` String,--厂家
`key_component_model` String,--器件型号是否与其他数据有关联
`version_batch` String,--版本批次
`key_parameter` String,--关键参数，
`count` UInt64,--数量，默认为1
`commissioning_date` Int64,--投运时间
`state` String, --为后续增加关键器件健康度预警做预留
`duration_thresholds` Int64, --使用时长阈值
`time_thresholds` UInt64, --动作次数阈值
`abnormal_time_thresholds` UInt64 --异常动作次数阈值
) ENGINE = ReplacingMergeTree(timestamp) ORDER BY (key_component_id) PRIMARY KEY (key_component_id);
```

表2 关键器件动作次数、异常动作次数表

````sql
DROP TABLE IF EXISTS `key_component_action_count`;--动作次数表
CREATE TABLE IF NOT EXISTS sems.key_component_action_count(
`timestamp` Int64, -- 时间戳
`device_id` UInt64,-- 设备id
`device_zone` String,-- 数据分区
`key_component_name` String,--关键器件唯  一识别号
`switching_cycles` UInt64,--正常动作次数
`abnormal_switching_cycles` UInt64 --异常动作次数
) ENGINE =  MergeTree  ORDER BY ( timestamp , device_id ) PRIMARY KEY (timestamp, device_id);
````

表3 关键器件更换记录表

```sql
DROP TABLE IF EXISTS `component_record`;--关键器件记录
CREATE TABLE IF NOT EXISTS sems.key_component(
`timestamp` Int64, -- 时间戳
`flag_del` UInt8, -- 删除标志 (0: 未删除, 1: 已删除)
`device_id` UInt8,-- 设备id
`workorder_id` String, --工单号,*和新版工单数据库对齐
`key_component_id_old` String,--更换下来的原来的关键器件唯一标识
`attribute_old` String, --关键器件/组成器件
`key_component_name_old` String,--名称
`key_component_type_old` String,--直流接触器、断路器、熔断器
`SAP_old` String,--sap
`supplier_old` String,--厂家
`key_component_model_old` String,--器件型号是否与其他数据有关联
`version_batch_old` String,--版本批次
`key_parameter_Old` String,--关键参数，
`commissioning_date_old` String,--投运时间
`key_component_id_new` String, --更换新的关键器件唯一标识
`attribute_new` String, --关键器件/组成器件
`key_component_name_new` String,--名称
`key_component_type_new` String,--直流接触器、断路器、熔断器
`SAP_new` String,--sap
`supplier_new` String,--厂家
`key_component_model_new` String,--器件型号是否与其他数据有关联
`version_batch_new` String,--版本批次
`key_parameter_new` String,--关键参数，
`commissioning_date_new` String,--投运时间
`disposal_method` String, --自行处理、返厂处理
`disposal_instructions` String, --详细解释自行处理的方式 
`disposal_verify` String,--返厂确认
) ENGINE = ReplacingMergeTree(timestamp) ORDER BY (device_id,key_component_id_old) PRIMARY KEY (device_id,key_component_id_old);
```

# 周二2025/5/27工作计划制定

{  "status": 1,  "msg": "成功",  "data": [WriteKeyComponent] }

{  "status": 0,  "msg": "错误信息",  "data": [] }

1.接口 ：启动失败记录√

2.接口：重启相关接口（修改）√

3.ota升级接口、参数配置接口逻辑梳理 √

4.协议里拓展功能相关的所有接口

+ 更新ip√
+ 无感自愈√
+ 桩禁用√
+ 软件版本查询√

## 运维平台讨论信息

1.运行数据，桩数据，需要回复运营平台信息

2.关键器件先做基础版本，充电桩主动上送，运维平台不应答。后续考虑运维平台应答关键器件报文帧上送，充电桩根据运维平台回复情况做相应逻辑判断。

## 制定运维协议、需要想兵强确认事宜

1.充电数据变位上送，是否使用过“运行数据”这个命令进行数据上送，或者说是否从始至终所有程序使用的都是“数据变位”命令进行数据上送。

2.目前充电桩有“事件上送”这条命令

##  关于IPD需要确认的一些细节

1.交易记录去掉金额，是在通过前端隐藏金额字段，还是修改当前交易记录接口，去掉金额字段。--个人意见通过前端隐藏

## 备品备件

### 数据库设计

```sql
--物料管理
DROP TABLE IF EXISTS `material_management`;--物料管理||库存列表
CREATE TABLE IF NOT EXISTS sems.material_management (
`timestamp` Int64, -- 时间戳
`flag_del` UInt8, -- 删除标志 (0: 未删除, 1: 已删除)
`material_id` UInt64, -- 物料id
`sap` UInt64, -- SAP
`material_type` String, -- 器件类型->物料类型
`supplier` String, -- 供应厂家
`material_model` String, -- 型号
`prewarning_value` String, -- 预警值
`amount` String, -- 数量
) ENGINE = ReplacingMergeTree(timestamp) ORDER BY (material_id) PRIMARY KEY (material_id);
--入库记录
DROP TABLE IF EXISTS `spare_part`;--备品备件
CREATE TABLE IF NOT EXISTS sems.spart_part (
`timestamp` Int64, -- 时间戳
`flag_del` UInt8, -- 删除标志 (0: 未删除, 1: 已删除)
`inbound_num` String, -- 入库单号
`material_id` String, --物料id
`material_source` String, -- 物料来源
`bath` String, -- 版本批次
`key_parameter` String, -- 关键参数
`count` String, -- 数量
`state` String, -- 状态
) ENGINE = ReplacingMergeTree(timestamp) ORDER BY (inbound_num) PRIMARY KEY (inbound_num);
--物料申请
DROP TABLE IF EXISTS `pile_component `;--物料申请
CREATE TABLE IF NOT EXISTS sems.pile_component (
`timestamp` Int64, -- 时间戳
`flag_del` UInt8, -- 删除标志 (0: 未删除, 1: 已删除)
`component_id` UInt64, -- 申请物料号
`quantity` UInt32 -- 申请改物料号的平台账户
`quantity` UInt32 -- 期望收货时间
`contract_num` String, -- 收货人	
`model` String, -- 收货人联系方式
`component_name` String, -- 收获地址省市区
`component_type` String, -- 收获地址详细地址
`component_model` String, -- 站名称
`supplier` String, -- 合同号
`material_number` String, -- 申领原因
`version_batch` String, -- 
`key_parameters` String, -- 关键参数
`quantity` UInt32 -- 数量 (假设为整数)
) ENGINE = ReplacingMergeTree(timestamp) ORDER BY (component_id) PRIMARY KEY (component_id);
```

## 日志召唤命令

```
{"命令":"日志召唤","reqid":234,"服务器地址":"114.116.81.164","端口":21,"用户名":"anonymous","密码":"","FTP路径":"/ops/AAAAALDJ/upload/log/BillingGun1.0.log","文件名":"BillingGun1.0.log"}
```

## ip切换命令

````
{"命令":"更新主站","reqid":0,"服务器IP":"114.116.81.164","端口":1882,"用户名":"","密码":""}
````

### ops_tcu 代码分析

main.rs

```
WorkerManager的handle{
	
}
```

tcu.rs

```
TcuServer的handle{
	MsgDeviceStatusInit
	MsgPxyCmd
	MsgRespMsg
	MsgConfigRequest
	MsgConfigStatus
	MsgTcuLoopQuery
	MsgActiveMsg
	MsgCmdResp
	StopWorker
}
```

worker.rs

```
WorkerManager的handle{
    MsgConfirmAllLogin
    MsgConfirmThisLogin
    InPubTopicPayload
    StopWorker
    MsgPxyCmd
    MsgRespMsg
    MsgConfigRequest
    MsgConfigStatus
}
RedisPxyer的handle{
	MsgGetAllCompany
	MsgRedisHset
}
MqttEventPoller的handle{
	MsgEventLoopStart
}
MqttClienter的handle{
	MsgMqtt
	MsgMqttSubs
	MsgMqttUnsubs
}
DbInserter的handle{
	MsgDbInsert
}
```

### 会议模版

```
会议通知：4月份流量超标解决方案评审
会议时间：2025年4月25日 下午16:30—18:30
会议地点：研发会议小桌
参加人员：马占宾、吕舟、丁孟超、张延龙、刘兵强、刘雪刚敬请准时参加。
会议操作议程：
1. 介绍当前流量超标的解决方案
2. 确定开服时间
3. 讨论并形成后续防止流量超标方案
```

## 需要编写接口的梳理

桩内器件：基础增加、批量导入、删除、查询

不涉及基础修改，出现错误情况，就删除后重新 增加/导入。

1.基础增加

2.基础删除

3.查询，查询需要考虑两种情况：1.桩内器件导入后列表。2.桩详情关键器件信息；以上两种情况通过一个接口，不同参数控制效果实现。

4.批量导入

## 备品备件问题

1.备品备件出库入库数据是否唯一性问题

2.库存告警是否有必要

3.备品备件消耗，数据与，备品备件出库区别

4.入库5，出库4，库存应该如何展示

## 信息接入

资产表(基础信息表)asset 主键为device_id

场站表station 主键为station_id

设备型号表device_model 主键为model_id

//桩内器件表component 主键为component_id

## 数据库设计

### 桩内器件

```sql
DROP TABLE IF EXISTS `asset`;--基本信息表
CREATE TABLE IF NOT EXISTS sems.asset( 
`timestamp` Int64,
`flag_del` UInt8,
`device_id` UInt64,--设备id
`sta_id` UInt64, --场站id
`dt_id` UInt64,--设备类型id
`model` String,--产品型号
`contract_num` String,--合同号
`manufacture_date` Int64,--出厂日期
`mvno` String,--运营商
`mvno_code` String,--运营商编码
`sim` String,--sim卡号
`audit_status` String,--审核状态
`maintecance_status` String,--维保状态
`guarantee_period` String,--质保期
`guarantee_start` Int64,--质保开始时间
`guarantee_end` Int64,--质保结束时间
`maintecance_organization` String,--维保单位
`responsible` String,--责任人
`contact_information` String,--联系方式
`name` String,--uuid
`commissioning_date` Int64--投运时间
) ENGINE = ReplacingMergeTree(timestamp)  ORDER BY ( device_id ) PRIMARY KEY (device_id);
DROP TABLE IF EXISTS `device_model`;--产品型号
CREATE TABLE IF NOT EXISTS sems.device_model ( 
`timestamp` Int64,
`flag_del` UInt8,
`model_id` UInt64,
`model` String,--产品型号
) ENGINE = ReplacingMergeTree(timestamp)  ORDER BY (model_id) PRIMARY KEY (model_id);

```

```sql
DROP TABLE IF EXISTS `pile_component `;--桩内器件
CREATE TABLE IF NOT EXISTS sems.key_component (
`timestamp` Int64, -- 时间戳
`flag_del` UInt8, -- 删除标志 (0: 未删除, 1: 已删除)
`contract_num` String, -- 合同号
`model_id` UInt64, -- 设备型号 ID
`component_id` UInt64, -- 器件 ID
`component_name` String, -- 器件名称
`component_type` String, -- 期间类型
`supplier` String, -- 供应厂家
`material_number` String, -- 物料号
`version_batch` String, -- 版本批次
`key_parameters` String, -- 关键参数
`quantity` UInt32 -- 数量 (假设为整数)
) ENGINE = ReplacingMergeTree(timestamp) -- 使用时间戳去重
ORDER BY (contract_num, model_id) -- 按合同号和设备型号排序
PRIMARY KEY (contract_num, model_id); -- 主键为合同号和设备型号组合
```



## 流量分析

```
001 1.48%
002 1.93%
004 0.32%
```

## 工作计划

```
1.充电桩对接运维平台2.0协议的开发与完善
2.emqx集群的部署
3.通信模块性能评估
4.通信模块架构改进，目标支持20w并发量。
5.通信模块ipd相关功能的开发。
6.桩对平台连接模式，消息发送规则的制定
```

# 2025/2/27

1.内存爆炸，清理的是哪些的内存。运行日志内存的位置，如何清理。

# 协议相关

1.注册完整逻辑，注册申请和注册结果回复？

2.更新id命令在实际中的应用？

# emqx相关

1.监听器监听的几个端口都是什么端口？

![image-20250226102602105](C:\Users\16167\AppData\Roaming\Typora\typora-user-images\image-20250226102602105.png)

2.众多random客户端都是什么客户端？

![image-20250226102944056](C:\Users\16167\AppData\Roaming\Typora\typora-user-images\image-20250226102944056.png)

3.mosquitto客户端 是怎么配置的？

4.图片中的主题是什么情况，为什么会有数字？![image-20250226141422451](C:\Users\16167\AppData\Roaming\Typora\typora-user-images\image-20250226141422451.png)

5.充电记录

![image-20250226141611890](C:\Users\16167\AppData\Roaming\Typora\typora-user-images\image-20250226141611890.png)

# 相关要点

1.tcu初次登录平台注册，redis key uuid，计数。

2.设备注册登录程序是一个独立的程序，程序路径在/home/lcg/ops/broker,服务配置ops_register,执行脚本为start_register.service.

3.uuid在设备进行手动更改可能造成uuid重复。出现问题，删除uuid进行重新分配。

4.充电记录确认，现在是ops_tcu模块收到后直接回复确认。

# 一些报文



```rust
    #[test]
    // {"命令":"运行数据","reqid":3,"数据分区":"1枪数据","系统时间":"2025-03-12 19:16:46+0800",{"名称":"工作状态","值":"空闲"},{"名称":"插枪状态","值":"已插枪"},{"名称":"输出电压","值":"239.977"},{"名称":"输出电流","值":"0.002"},{"名称":"已充时长","值":"0"},{"名称":"SOC","值":"0"},{"名称":"电表度数_度","值":"19.754"}}
    fn device_data_test() {
        // let payload = r#" {"命令":"运行数据","reqid":0,"数据分区":"1枪数据","系统时间":"2025-03-12 18:59:01+0800","工作状态":"空闲","车辆连接状态":"未插枪","电压_V":"239.746","电流_A":"0.001","已充电量":"0","已充时间":"0","SOC":"0","电表读数_度":"19.726"}"#;
        let payload = r#"{"命令":"运行数据","reqid":3,"数据分区":"1枪数据","系统时间":"2025-03-12 19:16:46+0800","状态数据":[{"名称":"工作状态","值":"空闲"},{"名称":"车辆连接状态","值":"已插枪"},{"名称":"电压_V","值":"239.977"},{"名称":"电流_A","值":"0.002"},{"名称":"已充电量","值":"0.002"},{"名称":"已充时间","值":"0"},{"名称":"SOC","值":"0"},{"名称":"电表读数_度","值":"19.754"}]}"#;
   
        let msg_trade_record_v2 = serde_json::from_str::<MsgRunDataV3>(&payload);
        println!("2312 {:?}", msg_trade_record_v2);
        assert_eq!(msg_trade_record_v2.unwrap().数据分区, "数据分区".to_string());
        // assert_eq!(c.end_soc, 100_u8);
    }
```

```sql
这是一个数据库详细清单表格，表格的格式为 序号	字段中文名	字段英文名	类型、宽度、 精度	取值约束	空否	默认值	主键/外键	索引否，根据sql语句，将相关内容填入对应字段下，取值约束不填，空否为否，取值约束为无，主键/外键为否，索引否为否。sql如下：
DROP TABLE IF EXISTS `organization_user`;--组织部门-->人员对应关系 表
CREATE TABLE IF NOT EXISTS sems.organization_user(
`timestamp` Int64,
`flag_del` UInt8,
`organization_id` UInt64,
`user_id` UInt64
)ENGINE =  ReplacingMergeTree(timestamp)  ORDER BY ( organization_id,user_id );
请生成表格
```

```shell
mosquitto_pub -t "/xjdy/ops/v1/TCU/AAAAALFW/cmd" -h 114.116.81.164 -m '{"命令":"下载","reqid":2,"文件类型":"程序包","服务器地址":"114.116.81.164","端口":21,"用户名":"anonymous","密码":"","FTP路径":"/ops/AAAAALFW/download/actest107.bin","字节数":583760,"MD5":"2a8d1a212c7f713cdacb5cb6df55353a","数字签名":"","版本号":"","子设备编号":[]}'
```

```json
{"命令":"更新程序并重启","reqid":3,"配置列表":[],"子设备编号":[]}
```

```shell
mosquitto_sub -t "/xjdy/ops/v1/TCU/+/#" -h 114.116.81.164
```

| 工作内容                                 | 完成率 | 需协调 |
| ---------------------------------------- | ------ | ------ |
| 对接交流桩                               | 80%    | 无     |
| 编写运维平台精简、校对版本               | 100%   | 无     |
| 修改运维平台connector模块代码            | 100%   | 无     |
| 运维平台升级部分代码对照协议升级方案编写 | 80%    |        |

```rust
fn create_and_print<T>() where T: From<i32> + Display {
fn create_and_print<T>() where T: From<i32> + Display {
```

代码逻辑分析

# end

### 如何计划制定

+ 明确和目标拆解
+ 了解自己的工作模式
+ 设定优先级
+ 使用“5分钟起步法”
+ 设定清晰的行动步骤
+ 环境与资源准备
+ 记录与评估

### 随便写写

#### 2025/7/4

最近的一些思考和感悟，很多自己达不到的事情，有一层说不清道不明的东西包裹，我称之为暗规则。区分明规则与暗规则，暗规则未满足，应对暗规则的技法，应对明规则的技法。



#### 以前不知道什么时候写的

关于职业的一些思考

首先，如果要对职业进行分别分类，应该如何分。

从“从业者的收益从哪里来”：

1.有编制老师、医生、警察、公务员：将自己的劳动力售卖给政府换取收益。2.酒店的服务员、小区的保安、教辅机构老师：在市场中为消费者提供服务获取收益。3.小摊小贩：对初级食品或商品加工，在市场中与消费者进行交易获取收益。4.商人：通过信息差、认知差将商品进行低买高卖，获取差价收益。5、矿场老板：通过出售基础资源获取收益。6.程序员、电气设计师：售卖自己有一定技术门槛的劳动力获取收益。