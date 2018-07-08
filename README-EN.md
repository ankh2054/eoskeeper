# eoskeeper
EOS BP节点高可用守护进程

[中文链接](https://github.com/eosstore/eoskeeper/blob/master/README.md)

### prompt
If you plan to use this program, read the source code:eoskeeper.py carefully.


### Implemented functionality
* Real-time monitoring node current block height, irreversible block height, current block BP name.
* Real-time monitoring of whether the BP node is coming out of the block. (1 master 2 standby uses the same key to start, you need to filter the log to determine which machine is out of the box)
* Real-time monitoring of the number of p2p links between nodes. (The number of connections is a very important parameter by calling the lsof command)
* Any abnormality in the parameters will be notified by Enterprise WeChat and SMS. (need to use your own WeChat and SMS module)
* When the primary BP fails, the first standby BP automatically issues a block after two rounds to achieve high availability of BP.
* If both the primary BP and the first standby BP fail to be blocked, the second standby BP will start to be out after 7 rounds to achieve high availability of BP.
* Push the running parameters of each node to influxdb in real time, and then display it through grafana.

other:grafana interface,you need get data from influxdb database and zabbix database,and do it for yourself.

### Runtime environment
python version:2.7  

### Install and run
```
Install
$ easy_install -U sh requests
$ mkdir /etc/eoskeeper add config.ini vi /etc/eoskeeper/config.ini （reference config.ini）
$ put eoskeeper.py source code to /usr/local/bin/eoskeeper.
$ chmod +x /usr/local/bin/eoskeeper
$ Modify configuration file,the important is to change the role. (the role is explained below. The main BP role is A, the first standby BP role is B, the second standby BP role is C, and the others are F)

Run
It is recommended to use systemctl service to run eoskeeper,please reference /systemctl/README.md to create service.
you can alse run eoskeeper.
```

### Introduce configuration  
```a
role = "A"
block_producer_name = "eosstorebest"            # bp name
eosio_log_file      = "/data/eosio.log"         # eosio log
eoskeeper_log_file  = "/data/eoskeeper.log"     # eoskeeper 
infulxdb_url        = "http://13.115.200.171:8086/write?db=eosdb"   # influxdb的url
mobiles             = "1821050****,1352131****"     #The phone number of the person who needs to be notified

```

### influxdb的两个表
BP节点表 （共3台机器）
表名 eosbpinfo  
字段名/中文名            属性          示例 
* host/主机名            (字符串)       eos-open-fn-1-1
* hbn/当前块             (整数)         19876
* lib/不可逆块            (整数)        19856
* linkn/连接数量          (整数)        34
* lpbt/上次出块时间     （字符串）        10秒前
* paused                 （字符串）     是
* info/告警信息         （字符串，最长60个字符） 
         

全节点表 （共7台机器）
表名 eosfninfo
* 字段名/中文名            属性          示例 
* host/主机名            (字符串)       eos-open-fn-1-1
* hbn/当前块             (整数)         19876
* lib/不可逆块            (整数)        19856
* linkn/连接数量          (整数)        34
* info/告警信息         （字符串，最长60个字符） 


### 逻辑说明

eoskeeper是一个用于监控eos程序的守护进程，并有报警和推送参数到influxdb的功能。

== 程序原理 ==  
我们的节点分为四种角色：A角色（BP）、B角色（备用BP，第一道防线）、C角色（备用BP，第二道防线）、普通全节点（后面用F角色表示）  
在三个主机中会分别给eoskeeper守护进程配置文件中设置为A、B、C三个角色，以下是eoskeeper根据角色做出相应的动作。  
当eosstorebest在前21名，B主机eosio运行正常，并且，B主机检测到2轮出块循环都没有eosstorebest账户时，B主机的eoskeeper会执行命令，使B出块。  
当eosstorebest在前21名，C主机eosio运行正常，并且，C主机检测到6轮出块循环都没有eosstorebest账户时，C主机的eoskeeper会执行命令，使C出块。    

== 配置相关 ==  
所有eosio需要配置 http-server-address = 127.0.0.1:8888  
为了/v1/producer/* api BP节点的eosio配置文件需增加 plugin = eosio::producer_api_plugin  

== 管理相关 ==  
任何一台主机出现故障时，都需要及时修复。修复后，使各个节点恢复自己的角色。  


### 相关命令
```bash
curl --request POST --url http://127.0.0.1:8888/v1/producer/pause
curl --request POST --url http://127.0.0.1:8888/v1/producer/resume
curl --request POST --url http://127.0.0.1:8888/v1/producer/paused
```




