### **syncClient**

>   syncClient，数据实时同步中间件（阿里canal到kafka、redis、elasticsearch）！

 本项目是打通canal、kafka的桥梁；  
 基本原理：  
 canal解析binlog的数据，由syncClient订阅，然后实时推送到kafka或者redis、elasticsearch；如果kafka、redis服务异常，syncClient会回滚操作；canal、kafka、redis异常退出，都不会影响数据的传输；


---

**目录：**  
bin：已编译二进制项目，可以直接使用；  
src：源代码；  

---

**配置说明：**

#common  
system.debug=1          # 是否开始调试：1未开启，0为关闭（线上运行请关闭）  

#canal  
canal.ip=127.0.0.1      # canal 服务端 ip;  
canal.port=11111        # canal 服务端 端口：默认11111;  
canal.destination=one   # canal 服务端项目（destinations），多个用逗号分隔，如：one,two;  
canal.username=         # canal 用户名：默认为空;   
canal.password=         # canal 密码：默认为空;  
canal.filter=           # canal 同步filter设置，默认空使用canal配置;  

#kafka or redis 服务配置，前缀对应canal.destination的多个destinations;
#redis
sdsw.target_type=redis  # 同步插件类型 kafka or redis  
sdsw.target_ip=         # kafka 服务端 ip;   
sdsw.target_port=       # kafka 端口：默认9092;   

#kafka
epos.target_type=kafka  # kafka 端口：默认9092;   
epos.target_ip=         # kafka 端口：默认9092;   
epos.target_port=       # kafka 端口：默认9092;   

#elasticsearch  
es.target_type=elasticsearch  
es.target_ip=10.5.3.66  
es.target_port=  

---

**使用场景(基于日志增量订阅&消费支持的业务)：**

数据库镜像  
数据库实时备份  
多级索引 (分库索引)  
search build  
业务cache刷新  
数据变化等重要业务消息  

**Kafka：**

Topic规则：数据库的每个表有单独的topic，如数据库admin的user表，对应的kafka主题名为：sync_admin_user  
Topic数据字段：  

	插入数据：
    {
        "head": {
            "binlog_pos": 53036,
            "type": "INSERT",
            "binlog_file": "mysql-bin.000173",
            "db": "sdsw",
            "table": "sys_log"
        },
        "after": [
            {
                "log_id": "1",
            },
            {
                "log_ip": "27.17.47.100",
            },
            {
                "log_addtime": "1494204717",
            }
        ]
    }
	
	修改数据：
    {
        "head": {
            "binlog_pos": 53036,
            "type": "UPDATE",
            "binlog_file": "mysql-bin.000173",
            "db": "sdsw",
            "table": "sys_log"
        },
        "before": [
            {
                "log_id": "1",
            },
            {
                "log_ip": "27.17.47.100",
            },
            {
                "log_addtime": "1494204717",
            }
        ],
        "after": [
            {
                "log_id": "1",
            },
            {
                "log_ip": "27.17.47.1",
            },
            {
                "log_addtime": "1494204717",
            }
        ]
    }
	
	删除数据：
    {
        "head": {
            "binlog_pos": 53036,
            "type": "DELETE",
            "binlog_file": "mysql-bin.000173",
            "db": "sdsw",
            "table": "sys_log"
        },
        "before": [
            {
                "log_id": "1",
            },
            {
                "log_ip": "27.17.47.1",
            },
            {
                "log_addtime": "1494204717",
            }
        ]
    }

head.type 类型：INSERT（插入）、UPDATE（修改）、DELETE（删除）； 

head.db 数据库； 

head.table 数据库表；

head.binlog_pos  日志位置； 

head.binlog_file 日志文件；  

before： UPDATE（修改前）、DELETE（删除前）的数据；  

after：  INSERT（插入后）、UPDATE（修改后）的数据；  


**Redis：**

List规则：数据库的每个表有单独的list，如数据库admin的user表，对应的redis list名为：sync_admin_user  

**Elasticsearch**

规则：同步index = sync，如数据库admin的user表，对应的Elasticsearch type名为：admin_user    
