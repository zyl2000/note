```
Host hw-test
    HostName 114.115.237.39
    Port 22
    User lcg
    ServerAliveInterval 60
    IdentityFile ~/.ssh/id_hecs
```

### 一些账号

```
bisheng
账号：161672238@qq.com
密码：BiSheng0068!
华为云
账号：hid_k-psf-bt9vqvm2d
密码：123456789zzm@
hw-test root HW_tbl_217
阿里云
密码：G7b!xQ2@pL9m$zR4tY
```

## 状态码

``` sql
307 重定向
1.cookie错误
2.账户访问自己没有的权限的接口

```

``` sql
401 Unauthorized
1.账号密码错误
2.数据库连接有问题，
```

``` sql
404 
1.web路由中没有这个url接口，错误原因一般为url前段后端url路径不匹配
```

``` sql
405 method not allowed  
1. http test 请求类型与 代码mod.rs 不相符，导致方法方法找不到
```

``` sql
422 Unprocessable Entity
1.错误原因一般为接受请求的结构体与传入参数实体不一致
```

``` sql
500 
1.传入参数不合法，导致程序出现崩溃。eg：除数为0
2.clickhouse sql 语句出现问题
```

## linux

### 代码从xjj到ali,半自动化脚本

```shell

scp -r xjj:/home/lcg/sems/webpage/dist lcg:/home/lcg/sems/webpage/dist_tmp

scp xjj:/home/lcg/sems/target/release/webserver lcg:/home/lcg/sems/target/release/webserver_tmp 

scp xjj:/home/lcg/sems/target/release/connector lcg:/home/lcg/sems/target/release/connector_tmp
```

```bash
echo "webpage finash"
cp -r /home/lcg/sems/webpage/dist_tmp  /home/lcg/sems/webpage/dist_1.08;
rm -rfv /home/lcg/sems/webpage/dist;
mv /home/lcg/sems/webpage/dist_tmp /home/lcg/sems/webpage/dist;
cd /home/lcg/sems/webpage;
ln -sv ../png ./dist/png;ln -sv ../static/ dist/static;
echo "webserver finash"
sudo systemctl stop sems_webserver;
cp /home/lcg/sems/target/release/webserver_tmp /home/lcg/sems/target/release/webserver_v1.14;
mv /home/lcg/sems/target/release/webserver_tmp /home/lcg/sems/target/release/webserver; 
sudo systemctl start sems_webserver;
echo "connector finash"
sudo systemctl stop sems_connector;
cp /home/lcg/sems/target/release/connector_tmp /home/lcg/sems/target/release/connector_v1.03;
mv /home/lcg/sems/target/release/connector_tmp /home/lcg/sems/target/release/connector; 
sudo systemctl start sems_connector;
```



### 防火墙

```shell
sudo firewall-cmd --list-port
sudo firewall-cmd --zone=public --add-port 1883/tcp --permanent
sudo firewall-cmd --reload
```

### 日志缓存

```shell
history | grep size
sudo journalctl -u sems_webserver
sudo journalctl --vacuum-size=50M
```

### 端口映射

```shell
本地运行，测试运维OTA相关功能
ssh -fCNL 8123:localhost:8123 lcg  #clickhouse
ssh -fCNL 6379:localhost:6379 lcg  #redis
ssh -fCNL 8088:localhost:8088 lcg  #grpc
```

```
端口映射
-l user：指定要登录的用户。
-p port：指定连接到远程主机的端口号，默认是22。
-i identity_file：指定身份验证文件（私钥文件）。
-v：详细模式，可以显示调试信息。
-C：启用压缩。
-N：不执行远程命令，只进行端口转发。
-f：后台运行。
-L local_port:remote_host:remote_port：本地端口转发。
-R remote_port:local_host:local_port：远程端口转发。
-D [bind_address:]port：动态应用程序级端口转发。
```

``` rust
ps -elf |grep ssh //查看进程
```

``` sql
ssh -fCNL 127.0.0.1:9199:192.168.0.8:9000 hw-mq
```

``` sql
sudo systemctl stop sems_webserver;./start.sh
```

``` rust
sudo systemctl status sems_webserver //查看应用状态
```

``` rust
sudo systemctl restart sems_webserver //重启应用
```

``` rust
ssh -fCNL 8123:localhost:8123 xjj  //端口映射
ssh -fCNL 6379:localhost:6379 xjj  //端口映射
ssh -fCNL 8123:localhost:8123 hw-test  //端口映射
ssh -fCNL 6379:localhost:6379 hw-test  //端口映射
```

``` rust
ssh -fCNL 8088:localhost:8088 hw-mq  //端口映射
```

``` rust
ssh -fCNL 6379:localhost:6379 hw-mq  //端口映射
ssh -fCNL 6379:192.168.0.8:6379 hw-mq 
ssh -fCNL 8123:hw-db:8123 hw-mq //端口映射到生产库
```

```sql
本地端口转发
ssh -L local_port:remote_host:remote_port user@hostname
ssh -L 8080:localhost:80 test@runoob.com
```

```sql
远程端口转发
ssh -R remote_port:local_host:local_port user@hostname 
ssh -R 8080:localhost:80 test@runoob.com
```

``` rust
netstat -ntlp   //查看当前所有tcp端口
```

``` rust
 echo "avatr#test_668" |md5sum
ln -sv ../png ./dist/png //建立软连接
ln -sv ../static/ dist/static //建立软连接
```

### scp

```shell

scp /home/lcg/sems/target/release/webserver_tmp hw-mq:/home/lcg/sems/target/release/webserver_tmp#从test 传输到mq
scp -r /home/lcg/sems/webpage/dist_tmp hw-mq:/home/lcg/sems/webpage/dist_tmp #从test传输到mq
 scp -r hw-mq:/home/lcg/pro/rust/projects/sems/ admin@47.122.118.170:/home/admin #从mq传输到ali
```

### 回滚代码

``` shell
sudo systemctl stop sems_webserver;cp  target/release/webserver_v1.44 target/release/webserver;sudo systemctl restart sems_webserver
```

### 常规命令

```bash
dpkg -l | grep zip #列出系统中与 zip 相关的已安装软件包
du -h sems_zip #查看文件大小
lsb_release -a #查看当前linxu系统版本
sudo su # 切换到root用户
su - lcg #用户之间的切换，从root用户切换为原来的用户
who am i #查询原始用户名
top # 查看cpu、内存使用情况
lsb_release -a #
sudo journalctl -u sems_webserver.service #查看某个应用日志
```

### 监听设备

```sh
$queue/+/ops/v1/+/+/status
$queue/+/ops/v1/+/+/msg
$queue/+/ops/v1/+/+/cmd_resp/+
```

```bash
/+/ops/v1/+/+/status
/+/ops/v1/+/+/msg
/+/ops/v1/+/+/cmd_resp/+
/+/注册ops/v1
/+/注册ops/v1/注册结果/+
```

### mosquitto相关命令

```shell
history |grep mos
mosquitto_sub   -v -t "/+/ops/v1/TCU/AAAAAARB/#"
history |grep mosq
mosquitto_sub -t /+/ops/v1/TCU/AAAAACFC/# -v
mosquitto_sub -t /+/ops/v1/TCU/# -v
```

### 新建服务

```shell
cd /etc/systemd/system #进入文件夹
#新建service文件
sudo systemctl daemon-reload #重启
```

### 静态编译

```shell
 RUSTFLAGS="-C target-feature=+crt-static" cargo build --target x86_64-unknown-linux-musl -r --bin broker
 
 RUSTFLAGS="-C target-feature=+crt-static" cargo build --target x86_64-unknown-linux-musl -r --bin restart
 
  RUSTFLAGS="-C target-feature=+crt-static" cargo build --target x86_64-unknown-linux-musl -r --bin trade_record_confirm
```

### 内存占用查看

```bash
#1使用 du 命令查看目录占用空间
sudo du -sh /*
#2深入查看特定目录
sudo du -sh /home/*
sudo du -sh /var/*
sudo du -sh /usr/*
#3检查日志文件
sudo du -sh /var/log/*
```



## git相关

#### tag相关

```shell
git tag 查看本地tag
git ls-remote --tags origin 查看远端tag
git tag v2.14 新建tag
git push origin --tags 推送本地tag到远程仓库
```

## clickhouse

### 日志删除

```bash
#1切换至root用户
sudo su root
#2先进入目标目录再执行命令：
cd /var/log/clickhouse-server/
sudo ls -l *.gz
#3删除
sudo rm *.gz
```



### trace_log相关命令及代码

```sql
--要在 ClickHouse 中查看某个特定数据库占用的磁盘空间
SELECT 
    database,
    table,
    formatReadableSize(sum(data_compressed_bytes)) AS compressed,
    formatReadableSize(sum(data_uncompressed_bytes)) AS uncompressed,
    sum(rows) AS rows
FROM system.parts 
WHERE active = 1 AND database = 'system'
GROUP BY database, table
ORDER BY sum(data_compressed_bytes) DESC;
```

```sql
--运行以下命令删除 ClickHouse 的系统日志表（它们会自动重建）
TRUNCATE TABLE system.query_log;
TRUNCATE TABLE system.trace_log;
```

```sql
--ClickHouse 为了防止误删大表，设置了保护机制。当你尝试删除一个 大于 max_table_size_to_drop 的表或分区时，会触发这个限制。
--通过 SQL 设置临时限制（适合单次操作）
--你可以在执行删除语句时传入查询参数来覆盖限制：
SET allow_drop_large_table = 1;
SET max_table_size_to_drop = '200Gi';
TRUNCATE TABLE system.trace_log;
```

```sql
--通过 TTL（Time To Live）机制设置 trace_log 表在 ClickHouse 中每 12 小时自动清除一次旧数据，你可以使用 ALTER TABLE ... MODIFY TTL 命令
--1检查现有的 trace_log 表结构
DESC system.trace_log;
--2如果需要，添加时间戳字段
--如果你发现没有合适的时间戳字段，可以通过如下命令添加（注意：这可能会影响现有数据的处理方式，谨慎操作）：
ALTER TABLE system.trace_log ADD COLUMN IF NOT EXISTS event_time DateTime DEFAULT now();
--3修改表的 TTL 设置
--使用 ALTER TABLE ... MODIFY TTL 命令来设置 TTL 策略，使得数据在 event_time 超过 12 小时时被删除
ALTER TABLE system.trace_log
    MODIFY TTL event_time + INTERVAL 12 HOUR;
--4验证设置是否成功
 DESC system.trace_log;
```



### 删除表中全部数据

```
truncate table sems.user;
truncate table sems.organization;
truncate table sems.organization_user;
truncate table sems.organization_station;
truncate table sems.template;
truncate table sems.template_permission;、
```

```shell
ssh -L 6379:C:6379 userB@B_server_ip
ssh -L 6379:hw-db:6379 hw-test
```

### 一些关键操作

```sql
1.以sql文件批量执行sql语句时，语句不能存在空格，不然会影响文件执行
2.clickhouse-client --password -d sems -n < warn_level.sql //数据库执行文件
3.SELECT * FROM organization INTO OUTFILE 'dump_test.sql' FORMAT SQLInsert --//导出sql语句
4.truncate table default.ck_table01;--清空表数据
```

### 时间相关

``` sql
 select toDateTime64(timestamp / 1000, 8) from station order by timestamp desc limit 1;
```

```sql
with now() as v select value from device_data where  
    (toDateTime(timestamp/1000,8)>toStartOfDay(dateAdd(Day,-?,v))) 
    and (toDateTime(timestamp/1000,8)<toStartOfDay(dateAdd(Day,-?,v))) and (var ='total_power')
```

```sql
select dateDiff('second',toDateTime(toStartOfDay(now())),toDateTime(now()))--计算时间差值，second可以换成hour、year、month等
```

```sql
with now() as v select toStartOfDay(dateAdd(Day,-1,v))
```

### 插入

```sql
insert into station (timestamp,flag_del,sta_id,sta_name,sta_loc,sta_type,mno,con_num,lat,lng) values(123114432,0,1,'许继测试站点','许继电源有限公司',1,'运营商','10086',45.3242,123.1123)
```

#### 查询结果创建新表

``` sql
create table company_weibo_new 
engine=MergeTree() 
order by id 
as
( SELECT id, cid, company_name, name, ico, info, tags, count, fans, follow_count, source_url, create_time, update_time, deleted, row_type, chuli_date
FROM economic_brain_temp.company_weibo where chuli_date = '2021-11-16 00:00:00')
```

####  查询结果insert现有表

``` sql
INSERT INTO company_weibo_old 
(ID, CID, COMPANY_NAME, NAME, ICO, INFO, TAGS, COUNT, FANS, FOLLOW_COUNT, SOURCE_URL, CREATE_TIME, UPDATE_TIME, DELETED, ROW_TYPE, CHULI_DATE)
 SELECT id, cid, company_name, name, ico, info, tags, count, fans, follow_count, source_url, create_time, update_time, deleted, row_type, chuli_date
FROM company_weibo where chuli_date = '2021-11-16 00:00:00'
```



### 迁移

``` sql
 insert into online_record select *from remote('127.0.0.1:9099','sems','device_data','default','') where var='online'
 insert into asset_old select a.timestamp as timestamp, a.ass_name as ass_name,  d.name as name  from asset final as a inner join (select * from device final) as d on a.device_id = d.device_id
 
     insert into asset_old (
     select ass.timestamp as timestamp , ass.flag_del as flag_del, ass.ass_name as ass_name, dev.name as name, dev.soft_version as soft_version from asset_old as ass inner join (select * from device) as dev on ass.device_id = dev.device_id limit 10)
    	insert into station(timestamp,flag_del,sta_id,sta_name,sta_loc,sta_type,mno,con_num,lat,lng,operation_type,oam_type,oam,sta_status,commissioning_date,sta_loc_s,create_time,creator,photo,ems)
SELECT timestamp,flag_del,sta_id,sta_name,sta_loc,sta_type,mno,con_num,lat,lng,operation_type,oam_type,oam,sta_status,commissioning_date,sta_loc_s,create_time,creator,photo,ems FROM station_bak_1
```

### 更改表结构

#### 重命名表

``` sql
RENAME TABLE table_A TO table_A_bak, table_B TO table_B_bak;
```

#### 增加列

``` sql
ALTER TABLE asset_old ADD COLUMN brand String;
ALTER TABLE trade_record ADD COLUMN tip_j Float64;
ALTER TABLE trade_record ADD COLUMN tip_f Float64;
ALTER TABLE trade_record ADD COLUMN tip_p Float64;
ALTER TABLE trade_record ADD COLUMN tip_g Float64;
```

#### 删除列

```  sql
ALTER TABLE visits DROP COLUMN browser
```

#### 清空列

``` sql
ALTER TABLE visits CLEAR COLUMN browser IN PARTITION tuple()
```

#### 修改列

``` sql
ALTER TABLE visits MODIFY COLUMN browser Array(String);
ALTER TABLE db_name.table_name MODIFY COLUMN endTimestamp Int64;//修改数据类型
ALTER TABLE db_name.table_name RENAME COLUMN regionId to region;//修改列名称
```

### 查询相关

#### 两表联查

``` sql
SELECT device_id,device.dt_id dt_id,device_type.type_name type_name,name,producer,ip_port,com,protocol
FROM device final inner join device_type FINAL ON device.dt_id = device_type.dt_id where device.flag_del=0 order by device_id limit ?,?
```

#### 子查询

``` sql
SELECT device_id,d.dt_id dt_id,t.type_name type_name,name,producer,ip_port,com,protocol
            FROM device as d final
            inner join device_type as t FINAL
            ON device.dt_id = device_type.dt_id
            where device.flag_del=0
            and d.device_id NOT IN (select a.device_id from asset as a final where flag_del=0)
            order by device_id
```

#### 查询语句 传入参数的方法

``` sql
let mut cursor = client
        .lock()
        .await
        .query(&format!(
            "SELECT * from device FINAL where flag_del=0 and dt_id={} ",
            xjtcu_dt_id
        ))
        .fetch::<WriteRegDevice>()
        .unwrap();
```

#### distinct去重

``` sql
 select distinct on (device_id, device_zone, var_id, value) * from device_data where var in ('online') order by timestamp desc
```

``` sql
 select distinct on (device_id, device_zone, warn_desc) * from warn_active order by timestamp desc
```

####  数据迁移

```sql
CREATE TABLE IF NOT EXISTS sems.user_station_new(
`timestamp` Int64,
`flag_del` UInt8,
`user_id` UInt64, 
`sta_id` UInt64  
)ENGINE =  ReplacingMergeTree(timestamp)  ORDER BY ( user_id,sta_id );

insert into user_station_new select *from remote('127.0.0.1','sems','user_station','default','');
rename table sems.user_station to sems.user_station_old;
rename table sems.user_station_new to sems.user_station;
```

```sql
Cookie:   id=KZ_WjArX7eI97-RaCzepoA; HttpOnly; SameSite=Strict; Path=/; Max-Age=86399
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOjEsInN1YiI6ImFkbWluIiwiYXVkIjoxLCJleHAiOjE3MTMyMzU0NTQsImlhdCI6MTcxMzE0OTA1NH0.ALpuUuMKemmtMNlTAgRjwUQNWoBzJ0476Q1IqEmQJPk
scp /home/lcg/.ssh/id_ed25519 C:\Users\16167\Desktop
```

#### inner join 

``` sql
 SELECT
 user.timestamp,user.flag_del,user.user_id,user.user_id,
 role.role_name,user.user_name,user.password,user.phone,user.email,user.note
 FROM user final inner join (select * from role final)  as role on user.role_id = role.role_id
 where user.flag_del =0 and user.role_id in
 (SELECT role_id FROM role final where role.flag_del=0 and role.user_id =1)
 
 SELECT
  asset.timestamp as timestamp,asset.flag_del as flag_del,asset.device_id as device_id,asset.sta_id as sta_id,asset.ass_name as ass_name,asset.dt_id as dt_id ,asset.ass_statu as statu, asset.mf_num as mf_num,asset.brand as brand ,asset.model as model,asset.contract_num as contract_num,asset.port_num as port_num,asset.charge_type as charge_type,asset.asset_type as asset_type,asset.rated_power as rated_power,asset.max_v as max_v,asset.max_i as max_i,asset.manufacture_date as manufacture_date,asset.mvno as mvno,mvno_code,sta.sta_name as sta_name,sim,audit_status,maintecance_status,guarantee_period,guarantee_start,guarantee_end,maintecance_organization,responsible,contact_information,dev.name as name
 FROM asset final 
 inner join (select * from device final where flag_del=0) as dev on asset.device_id=dev.device_id
 inner join (select * from station final where flag_del=0) as sta on asset.sta_id=sta.sta_id
 where asset.flag_del =0 
 {}{}{}{}{}{}{} order by device_id desc

```

#### select role

``` sql
select timestamp,flag_del,role_id,user_id,role_name from role final where flag_del =0
```

## 版本情况

hw-mq

```
1.85	--稳定版本，没有跟新操作记录和账号定期修改密码
1.87 	--稳定版本，增加操作记录和账号定期修改密码功能
1.88	--稳定版本，增加场站名唯一限制，修改场站删除逻辑 (有一条数据显示两遍的小bug)
1.89	--稳定版本，解决1.88的bug
1.90	--1.增加密码传输rsa加密解密，密码存储MD5+加盐处理
		  2.告警搜索bug修复
1.91	--1.修改告警故障事件下拉框与之前不兼容的问题 // dist 同步更新至1.80
		  2.告警故障事件列表增加模糊搜索
```





## websocat

``` sql
 websocat ws://114.115.237.39:4000/ws
 {"subscribe":"/ops/tcu/AAAAAAQU/event"}
```

``` sql
 while let rlt_dbg = cursor1.next().await {
        if ! rlt_dbg.is_ok() { println!("#{}:{}, {:?}", file!(),line!(), rlt_dbg) ;};
        if let Ok(Some(row)) = rlt_dbg {
            let mut cursor2 = client
                .lock()
                .await
                .query("SELECT * FROM device final where flag_del =0 and device_id =?")
                .bind(row.device_id)
                .fetch::<WriteRegDevice>()
                .unwrap();
            let version = if let Ok(Some(row)) = cursor2.next().await {
                row.soft_version
            } else {
                "".to_string()
            };
```

``` sql
        四个表、两个直流桩、八个交流桩、一个光伏、一个储能、
```

![image-20240805111203409](C:\Users\16167\AppData\Roaming\Typora\typora-user-images\image-20240805111203409.png)

```sql
       and toDateTime((timestamp+86400000)/1000,8)<toDateTime(now())
        and value = '故障'
```

```sql
select
    w.warn_desc as warn_desc,
    count()
    from warn_active as w
    inner join (select * from asset final)  as a on a.device_id = w.device_id
    inner join (select * from station final where flag_del=0 ) as s on a.sta_id = s.sta_id
    where a.flag_del = 0 and w.true_false = true
    and w.warn_desc in(select warn_desc from warn_level_class where class_id=3)
    {}
    {}
    group by warn_desc order by count() desc limit 20 
```

```sql
and sta_id in(select sta_id from organization_station final where flag_del =0 
    and organization_id =(select organization_id from organization_user final where flag_del =0 and user_id ={}))",claim.iss);
```

```sql
  format!("and sta_id in(select sta_id from organization_station final where flag_del =0 and organization_id =
            (select organization_id from organization_user final where flag_del =0 and user_id ={}))",claim.iss)
```

```
select sta_id from organization_station final where flag_del =0 and organization_id =(select organization_id from organization_user final where flag_del =0 and user_id ={})
```



```
工作目标拆分为步骤：
测试组织架构约束：
1.新建所有标准模版
2.依照从模版到人员的模式进行新建测试
3.进行人员可见性测试
4.进行人员树状图新建站可见性测试
```

```xml
需求：新建单位接口，选定负责人列表，返回所有不是负责人的，权限内的人员
1.不是负责人
2.权限内
方案1：（大减小）
1.总的人员列表A减去负责人列表B得到C
2.C减去C权限意外的列表D得到E
3.E为最终要求答案
方案2：（从小往大相加）
1.所有游客组织除负责人外的人员列表a
2.所有新建单位接口操作者的权限内的负责人意外的人员列表b
3.a+b得到最终要求答案c
方案评估：
1.
```

## 杂七杂八

### redis rdb数据库数据迁移

```shell
chmod 660 ~/dump.rdb
sudo chown redis:redis ~/dump.rdb
sudo cp ~/dump.rdb   /var/lib/redis/
sudo ls -lsa /var/lib/redis/
sudo systemctl restart redis
```

### 交易记录数据导出

-----sql

```sql
select count()
        from sems.trade_record as t 
        inner join (select * from sems.asset final where flag_del=0) as a on t.device_id=a.device_id 
        inner join (SELECT * FROM sems.station FINAL WHERE flag_del=0 ) as s on a.sta_id=s.sta_id 
where t.stop_code not in('用户远程停止','正常停机','充满自动停止','app停止充电','用户金额不足','充电中车辆充满',
        '用户按钮停止','用户金额用完','车辆充满','充满停止','触控屏手动停止','后台停止充电','达到设置充电时长停止','达到设置充电电量停止',
        '达到设置充电金额停止','达到 SOC 终止条件停止','扫码停止','车端S2主动断开','BST报文停机','达到预设充满条件',
        '刷卡停机','收到 BST(总电压满足)','收到 BST(单体电压满足)','收到 BST(soc已满足)') and t.timestamp>1734056742000
```

```sql
select t.timestamp as timestamp,
        t.dt_id as dt_id,t.device_id as device_id,trade_id,
        device_zone,app_time,pile_num,gun_num,
        toDateTime64(start_time / 1000, 8) as start_time,
        toDateTime64(end_time / 1000, 8) as end_time,
        start_soc,end_soc,price_j,kwh_j,loss_kwh_j,money_j,
        price_f,kwh_f,loss_kwh_f,money_f,price_p,kwh_p,
        loss_kwh_p,money_p,price_g,kwh_g,
        loss_kwh_g,money_g,start_meter_kwh,
        end_meter_kwh,totoal_kwh as total_kwh,loss_kwh,money,
        vin,charge_way,physic_card_num,stop_code,
        ass_name,a.sta_id as sta_id,s.sta_name as sta_name,
       dateDiff('hour', start_time, end_time) as charge_duration, 
        tips_j, tips_f, tips_p, tips_g, tips
            from sems.trade_record as t 
        inner join (select * from sems.asset final where flag_del=0) as a on t.device_id=a.device_id 
        inner join (SELECT * FROM sems.station FINAL WHERE flag_del=0 ) as s on a.sta_id=s.sta_id 
where t.stop_code not in('用户远程停止','正常停机','充满自动停止','app停止充电','用户金额不足','充电中车辆充满',
        '用户按钮停止','用户金额用完','车辆充满','充满停止','触控屏手动停止','后台停止充电','达到设置充电时长停止','达到设置充电电量停止',
        '达到设置充电金额停止','达到 SOC 终止条件停止','扫码停止','车端S2主动断开','BST报文停机','达到预设充满条件',
        '刷卡停机','收到 BST(总电压满足)','收到 BST(单体电压满足)','收到 BST(soc已满足)') and t.timestamp>1734056742000
            
```

------shell 

```shell
clickhouse-client --user default --password sems_echarge --query "select t.timestamp as timestamp,
        t.dt_id as dt_id,t.device_id as device_id,trade_id,
        device_zone,app_time,pile_num,gun_num,
        start_time,
        end_time,
        start_soc,end_soc,price_j,kwh_j,loss_kwh_j,money_j,
        price_f,kwh_f,loss_kwh_f,money_f,price_p,kwh_p,
        loss_kwh_p,money_p,price_g,kwh_g,
        loss_kwh_g,money_g,start_meter_kwh,
        end_meter_kwh,totoal_kwh as total_kwh,loss_kwh,money,
        vin,charge_way,physic_card_num,stop_code,
        ass_name,a.sta_id as sta_id,s.sta_name as sta_name,
        (end_time - start_time) as charge_duration, 
        tips_j, tips_f, tips_p, tips_g, tips
            from sems.trade_record as t 
        inner join (select * from sems.asset final where flag_del=0) as a on t.device_id=a.device_id 
        inner join (SELECT * FROM sems.station FINAL WHERE flag_del=0 ) as s on a.sta_id=s.sta_id 
where t.stop_code not in('用户远程停止','正常停机','充满自动停止','app停止充电','用户金额不足','充电中车辆充满',
        '用户按钮停止','用户金额用完','车辆充满','充满停止','触控屏手动停止','后台停止充电','达到设置充电时长停止','达到设置充电电量停止',
        '达到设置充电金额停止','达到 SOC 终止条件停止','扫码停止','车端S2主动断开','BST报文停机','达到预设充满条件',
        '刷卡停机','收到 BST(总电压满足)','收到 BST(单体电压满足)','收到 BST(soc已满足)') and t.timestamp>1734056742000
             FORMAT CSV" > trade_record2.csv

```

### 回滚connect模块

```
sudo systemctl stop sems_connector;
cp /home/lcg/sems/target/release/connector_v1.36.1  /home/lcg/sems/target/release/connector;
sudo systemctl start sems_connector
```

```sql
INSERT INTO device_data (
    device_id, 
    name, 
    bag_type, 
    bag_name, 
    timestamp, 
    result, 
    cksm, 
    update_status, 
    reboot_status, 
    operator
) VALUES (
    2706, 
    'AAAAAJHE', 
    'app', 
    '欧标yinhi1.0420250104', 
    1736837283442, 
    '', 
    '0bfe4d692b5385b9e3795810d4276ec7', 
    '下载成功允许重启', 
    '', 
    'zhaixueming'
);
```

## 软件安装

### mosquitto

```shell
sudo apt update --更新软件包列表
sudo apt install mosquitto mosquitto-clients --安装mosquitto
sudo systemctl start mosquitto --启动服务
sudo systemctl enable mosquitto --系统启动时自动运行
```

