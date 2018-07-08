# eoskeeper

[中文链接](https://github.com/eosstore/eoskeeper/blob/master/README.md)

### Prompt
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

### Two list of influxdb 
BP list （3 hosts）
list-name: eosbpinfo  
Field name/Chinese name       attribute      sample 
* host/host name              (string)      eos-open-fn-1-1
* hbn/current block           (int)         19876
* lib/irreversible block      (int)         19856
* linkn/link number           (int)         34
* lpbt/last produce time     （string）      10秒前
* paused                     （string）      yes
* info/alarm information （string，No more than 60 characters） 
         

fullnode list （7 hosts）
list-name: eosfninfo
* Field name/Chinese name       attribute      sample 
* host/host name               (string)       eos-open-fn-1-1
* hbn/current block             (int)         19876
* lib/irreversible block        (int)         19856
* linkn/link number             (int)         34
* info/alarm information      （string，No more than 60 characters） 


### Logic instructions

eoskeeper is a monitor the daemon of the eos program,with function of alarm and push message to influxdb。

== Principle of procedure ==  
Our have 4 roles:A(BP),B(backup BP,the first line of defense),C(backup BP,the second line of defense),fullnode(role F)
The three hosts will be set to the roles of A, B, and C in the configuration file of the eoskeeper daemon. The following is the action taken by eoskeeper according to the role. 
When eosstorebest before the 21st,eosio in host B running normal,and when hostB check eosstorebest isn't produce blocks with 2 round,eoskeeper of host C could perform command,let node B produce.
When eosstorebest before the 21st,eosio in host C running normal,and when hostC check eosstorebest isn't produce blocks with 2 round,eoskeeper of host C could perform command,let node C produce.

== about configuration ==  
all eosio need to be configured http-server-address = 127.0.0.1:8888  
In order to /v1/producer/* api, eosio-configuration of bp node need add plugin = eosio::producer_api_plugin  

== about admin ==  
When each host fails, it needs to be repaired in time. After the fix, restore each node to its own role.


### Relevant command
```bash
curl --request POST --url http://127.0.0.1:8888/v1/producer/pause
curl --request POST --url http://127.0.0.1:8888/v1/producer/resume
curl --request POST --url http://127.0.0.1:8888/v1/producer/paused
```




