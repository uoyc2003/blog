---
title: Docker快速搭建开发环境
tags:
  - Docker
  - 开发环境
categories:
  - 编程笔记
headimg: https://pic.zeng.cyou/wide/202409042025521.webp
abbrlink: 28831
date: 2024-09-02 22:11:39
---

## MySQL

### 创建挂载的目录

```shell
mkdir -p /data/mysql/{data,logs,conf}
```

### 创建挂载的配置文件

```shell
vim /data/mysql/conf/my.cnf
```

```conf
[mysqld]
user=mysql
port=3306
bind-address=0.0.0.0

#日志设置
log-error=/var/log/mysql/error.log
log_timestamps=SYSTEM

#慢查询日志设置
slow_query_log=1
slow_query_log_file=/var/log/mysql/slow.log
long_query_time=2

#存储引擎设置
default-storage-engine=InnoDB

#InnoDB设置
innodb_buffer_pool_size=1G
innodb_log_file_size=256M
innodb_flush_log_at_trx_commit=1
innodb_flush_method=O_DIRECT

#字符集和排序规则
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

#严格模式和SQL模式设置
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[mysql]
default-character-set=utf8mb4

[client]
default-character-set=utf8mb4
```

### 创建docker-compose.yml

```shell
vim /data/mysql/docker-compose.yml
```

```yaml
services:
    mysql:
        image: mysql:8.0.34
        restart: unless-stopped
        environment:
            - MYSQL_ROOT_PASSWORD=root
        volumes:
            - /data/mysql/conf:/etc/mysql/conf.d
            - /data/mysql/logs:/var/log/mysql
            - /data/mysql/data:/var/lib/mysql
        ports:
            - 3306:3306
        hostname: mysql
        container_name: mysql
```

### 启动容器

```shell
docker-compose up -d
```

## Redis

### 创建挂载的目录

```shell
mkdir -p /data/redis/{data,conf}
```

### 创建挂载的配置文件

```shell
vim /data/redis/conf/redis.conf
```

```conf
#设置Redis运行的模式(非保护模式)
protected-mode no

#监听的端口号
port 6379

#TCP连接队列的最大长度
tcp-backlog 511

#设置连接密码
requirepass root

#客户端空闲多长时间后关闭连接
timeout 0

#TCP连接的保活时间
tcp-keepalive 300

#是否在后台运行(不建议在生产环境使用)
daemonize no

#监督模式(不启用)
supervised no

#存放进程ID的文件路径
pidfile /var/run/redis_6379.pid

#日志级别
loglevel notice

#日志文件路径(为空表示不写日志文件)
logfile ""

#数据库数量
databases 30

#是否总是显示Redis的Logo
always-show-logo yes

#持久化配置
#在指定时间内,如果指定数量的键被修改,执行一次保存操作
save 900 1
save 300 10
save 60 10000

#如果在执行Bgsave(后台保存)时出错,停止写入操作
stop-writes-on-bgsave-error yes

#是否启用RDB压缩
rdbcompression yes

#是否在RDB文件中添加校验和
rdbchecksum yes

#RDB文件的名称
dbfilename dump.rdb

#RDB文件的存储目录
dir ./

#从节点是否提供过期的键
replica-serve-stale-data yes

#从节点是否只读
replica-read-only yes

#是否启用无盘同步
repl-diskless-sync no

#是否禁用TCP_NODELAY选项
repl-disable-tcp-nodelay no

#从节点优先级
replica-priority 100

#是否启用惰性删除
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

#是否开启AOF持久化
appendonly yes

#AOF文件的名称
appendfilename "appendonly.aof"

#在AOF重写时是否禁用文件系统同步
no-appendfsync-on-rewrite no

#当AOF文件大小达到总数据大小的百分之多少时,执行AOF重写
auto-aof-rewrite-percentage 100

#当AOF文件大小达到多少时,执行AOF重写
auto-aof-rewrite-min-size 64mb

#是否在截断的AOF文件上加载数据
aof-load-truncated yes

#是否使用RDB文件的文件头
aof-use-rdb-preamble yes

#执行Lua脚本的时间限制
lua-time-limit 5000

#慢查询日志的最大长度
slowlog-max-len 128

#订阅通知的事件类型
notify-keyspace-events ""

#Hash类型数据结构的最大ziplist条目数和值大小
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

#List类型数据结构的最大ziplist大小和压缩深度
list-max-ziplist-size -2
list-compress-depth 0

#Set类型数据结构的最大intset条目数
set-max-intset-entries 512

#Zset类型数据结构的最大ziplist条目数和值大小
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

#HyperLogLog数据结构的最大稀疏字节数
hll-sparse-max-bytes 3000

#Stream数据结构的最大节点字节数和最大节点数量
stream-node-max-bytes 4096
stream-node-max-entries 100

#是否启用主动rehash操作
activerehashing yes

#Redis的执行频率(单位:赫兹)
hz 10

#动态调整Redis的执行频率
dynamic-hz yes

#是否在AOF重写时增量地执行文件同步
aof-rewrite-incremental-fsync yes

#是否在RDB保存时增量地执行文件同步
rdb-save-incremental-fsync yes
```

### 创建docker-compose.yml

```shell
vim /data/redis/docker-compose.yml
```

```yaml
services:
    redis:
        image: redis:6.0
        restart: unless-stopped
        command: 'redis-server /etc/redis/redis.conf'
        logging:
            options:
                max-file: 2
                max-size: 100m
        volumes:
            - /data/redis/conf/redis.conf:/etc/redis/redis.conf
            - /data/redis/data:/data
        ports:
            - 6379:6379
        hostname: redis
        container_name: redis
```

### 启动容器

```shell
docker-compose up -d
```

## Nginx

> 访问页面：<http://ip:80>

### 创建挂载的目录

```shell
mkdir -p /data/nginx/{conf,html,logs}
```

### 先启动一次容器

```shell
docker run -d --name nginx nginx
```

### 把容器里面的内容复制出来

```shell
docker cp nginx:/etc/nginx/nginx.conf /data/nginx/conf/nginx.conf
docker cp nginx:/etc/nginx/conf.d /data/nginx/conf/conf.d
docker cp nginx:/usr/share/nginx/html /data/nginx
docker cp nginx:/var/log/nginx/. /data/nginx/logs
```

### 删除容器

```shell
docker rm -f nginx
```

### 创建docker-compose.yml

```shell
vim /data/nginx/docker-compose.yml
```

```yaml
services:
    nginx:
        image: nginx
        restart: unless-stopped
        volumes:
            - /data/nginx/html:/usr/share/nginx/html
            - /data/nginx/logs:/var/log/nginx
            - /data/nginx/conf/conf.d:/etc/nginx/conf.d
            - /data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf
        ports:
            - 80:80
        hostname: nginx
        container_name: nginx
```

### 启动容器

```shell
docker-compose up -d
```

## Nacos

> 访问页面：<http://ip:8848/nacos/index.html>

### 创建挂载的目录

```shell
mkdir -p /data/nacos
```

### 先启动一次容器

```shell
docker run -d --name nacos nacos/nacos-server
```

### 把容器里面的内容复制出来

```shell
docker cp nacos:/home/nacos/logs /data/nacos
docker cp nacos:/home/nacos/conf /data/nacos
```

### 删除容器

```shell
docker rm -f nacos
```

### 创建挂载的配置文件

```shell
vim /data/nacos/conf/application.properties
```

记得修改mysql的连接地址。

```conf
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://mysql:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=30000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user=root
db.password=root
```

### 运行sql文件

```sql
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID,空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额,0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限,单位为字节,0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数,,0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限,单位为字节,0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',
  `src_user` text,
  `src_ip` varchar(20) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `encrypted_data_key` text NOT NULL COMMENT '秘钥',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额,0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限,单位为字节,0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限,单位为字节,0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';

CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE users (
 username varchar(50) NOT NULL PRIMARY KEY,
 password varchar(500) NOT NULL,
 enabled boolean NOT NULL
);

CREATE TABLE roles (
 username varchar(50) NOT NULL,
 role varchar(50) NOT NULL,
 constraint uk_username_role UNIQUE (username,role)
);

CREATE TABLE permissions (
    role varchar(50) NOT NULL,
    resource varchar(512) NOT NULL,
    action varchar(8) NOT NULL,
    constraint uk_role_permission UNIQUE (role,resource,action)
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```

### 创建docker-compose.yml

```shell
vim /data/nacos/docker-compose.yml
```

```yaml
services:
    nacos-server:
        image: nacos/nacos-server
        restart: unless-stopped
        privileged: true
        environment:
            - MODE=standalone
            - JVM_XMX=256m
            - JVM_XMS=256m
        volumes:
            - /data/nacos/conf:/data/nacos/conf
            - /data/nacos/logs:/data/nacos/logs
        ports:
            - 9849:9849
            - 9848:9848
            - 8848:8848
        hostname: nacos
        container_name: nacos
```

### 启动容器

```shell
docker-compose up -d
```

## Minio

> 访问页面：<http://ip:9090>

### 创建挂载的目录

```shell
mkdir -p /data/minio/{{data,conf},data/data-{1..4}}
```

### 创建挂载的配置文件

```shell
vim /data/minio/conf/config.env
```

```conf
#账号和密码(至少8位)
MINIO_ROOT_USER=minioroot
MINIO_ROOT_PASSWORD=minioroot

#存储卷路径
MINIO_VOLUMES="/data-{1...4}"
```

### 创建docker-compose.yml

```shell
vim /data/minio/docker-compose.yml
```

```yaml
services:
    minio:
        image: quay.io/minio/minio
        restart: unless-stopped
        command: 'server --console-address ":9090"'
        environment:
            - MINIO_CONFIG_ENV_FILE=/etc/config.env
        volumes:
            - /data/minio/conf/config.env:/etc/config.env
            - /data/minio/data/data-4:/data-4
            - /data/minio/data/data-3:/data-3
            - /data/minio/data/data-2:/data-2
            - /data/minio/data/data-1:/data-1
        ports:
            - 9090:9090
            - 9000:9000
        hostname: minio
        container_name: minio
```

### 启动容器

```shell
docker-compose up -d
```

## Xxl-Job

> 访问页面：<http://ip:7397/xxl-job-admin>

### 创建挂载的目录

```shell
mkdir -p /data/xxl-job/logs
```

### 运行sql文件

```sql
CREATE database if NOT EXISTS `xxl_job` default character set utf8mb4 collate utf8mb4_unicode_ci;
use `xxl_job`;

SET NAMES utf8mb4;

CREATE TABLE `xxl_job_info` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `job_group` int(11) NOT NULL COMMENT '执行器主键ID',
  `job_desc` varchar(255) NOT NULL,
  `add_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  `author` varchar(64) DEFAULT NULL COMMENT '作者',
  `alarm_email` varchar(255) DEFAULT NULL COMMENT '报警邮件',
  `schedule_type` varchar(50) NOT NULL DEFAULT 'NONE' COMMENT '调度类型',
  `schedule_conf` varchar(128) DEFAULT NULL COMMENT '调度配置,值含义取决于调度类型',
  `misfire_strategy` varchar(50) NOT NULL DEFAULT 'DO_NOTHING' COMMENT '调度过期策略',
  `executor_route_strategy` varchar(50) DEFAULT NULL COMMENT '执行器路由策略',
  `executor_handler` varchar(255) DEFAULT NULL COMMENT '执行器任务handler',
  `executor_param` varchar(512) DEFAULT NULL COMMENT '执行器任务参数',
  `executor_block_strategy` varchar(50) DEFAULT NULL COMMENT '阻塞处理策略',
  `executor_timeout` int(11) NOT NULL DEFAULT '0' COMMENT '任务执行超时时间,单位秒',
  `executor_fail_retry_count` int(11) NOT NULL DEFAULT '0' COMMENT '失败重试次数',
  `glue_type` varchar(50) NOT NULL COMMENT 'GLUE类型',
  `glue_source` mediumtext COMMENT 'GLUE源代码',
  `glue_remark` varchar(128) DEFAULT NULL COMMENT 'GLUE备注',
  `glue_updatetime` datetime DEFAULT NULL COMMENT 'GLUE更新时间',
  `child_jobid` varchar(255) DEFAULT NULL COMMENT '子任务ID,多个逗号分隔',
  `trigger_status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '调度状态:0-停止,1-运行',
  `trigger_last_time` bigint(13) NOT NULL DEFAULT '0' COMMENT '上次调度时间',
  `trigger_next_time` bigint(13) NOT NULL DEFAULT '0' COMMENT '下次调度时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `job_group` int(11) NOT NULL COMMENT '执行器主键ID',
  `job_id` int(11) NOT NULL COMMENT '任务,主键ID',
  `executor_address` varchar(255) DEFAULT NULL COMMENT '执行器地址,本次执行的地址',
  `executor_handler` varchar(255) DEFAULT NULL COMMENT '执行器任务handler',
  `executor_param` varchar(512) DEFAULT NULL COMMENT '执行器任务参数',
  `executor_sharding_param` varchar(20) DEFAULT NULL COMMENT '执行器任务分片参数,格式如 1/2',
  `executor_fail_retry_count` int(11) NOT NULL DEFAULT '0' COMMENT '失败重试次数',
  `trigger_time` datetime DEFAULT NULL COMMENT '调度-时间',
  `trigger_code` int(11) NOT NULL COMMENT '调度-结果',
  `trigger_msg` text COMMENT '调度-日志',
  `handle_time` datetime DEFAULT NULL COMMENT '执行-时间',
  `handle_code` int(11) NOT NULL COMMENT '执行-状态',
  `handle_msg` text COMMENT '执行-日志',
  `alarm_status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '告警状态:0-默认、1-无需告警、2-告警成功、3-告警失败',
  PRIMARY KEY (`id`),
  KEY `I_trigger_time` (`trigger_time`),
  KEY `I_handle_code` (`handle_code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_log_report` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `trigger_day` datetime DEFAULT NULL COMMENT '调度-时间',
  `running_count` int(11) NOT NULL DEFAULT '0' COMMENT '运行中-日志数量',
  `suc_count` int(11) NOT NULL DEFAULT '0' COMMENT '执行成功-日志数量',
  `fail_count` int(11) NOT NULL DEFAULT '0' COMMENT '执行失败-日志数量',
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `i_trigger_day` (`trigger_day`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_logglue` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `job_id` int(11) NOT NULL COMMENT '任务,主键ID',
  `glue_type` varchar(50) DEFAULT NULL COMMENT 'GLUE类型',
  `glue_source` mediumtext COMMENT 'GLUE源代码',
  `glue_remark` varchar(128) NOT NULL COMMENT 'GLUE备注',
  `add_time` datetime DEFAULT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_registry` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `registry_group` varchar(50) NOT NULL,
  `registry_key` varchar(255) NOT NULL,
  `registry_value` varchar(255) NOT NULL,
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `i_g_k_v` (`registry_group`,`registry_key`,`registry_value`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_group` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `app_name` varchar(64) NOT NULL COMMENT '执行器AppName',
  `title` varchar(12) NOT NULL COMMENT '执行器名称',
  `address_type` tinyint(4) NOT NULL DEFAULT '0' COMMENT '执行器地址类型:0=自动注册、1=手动录入',
  `address_list` text COMMENT '执行器地址列表,多地址逗号分隔',
  `update_time` datetime DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL COMMENT '账号',
  `password` varchar(50) NOT NULL COMMENT '密码',
  `role` tinyint(4) NOT NULL COMMENT '角色:0-普通用户、1-管理员',
  `permission` varchar(255) DEFAULT NULL COMMENT '权限:执行器ID列表,多个逗号分割',
  PRIMARY KEY (`id`),
  UNIQUE KEY `i_username` (`username`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `xxl_job_lock` (
  `lock_name` varchar(50) NOT NULL COMMENT '锁名称',
  PRIMARY KEY (`lock_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO `xxl_job_group`(`id`, `app_name`, `title`, `address_type`, `address_list`, `update_time`) VALUES (1, 'xxl-job-executor-sample', '示例执行器', 0, NULL, '2018-11-03 22:21:31' );
INSERT INTO `xxl_job_info`(`id`, `job_group`, `job_desc`, `add_time`, `update_time`, `author`, `alarm_email`, `schedule_type`, `schedule_conf`, `misfire_strategy`, `executor_route_strategy`, `executor_handler`, `executor_param`, `executor_block_strategy`, `executor_timeout`, `executor_fail_retry_count`, `glue_type`, `glue_source`, `glue_remark`, `glue_updatetime`, `child_jobid`) VALUES (1, 1, '测试任务1', '2018-11-03 22:21:31', '2018-11-03 22:21:31', 'XXL', '', 'CRON', '0 0 0 * * ? *', 'DO_NOTHING', 'FIRST', 'demoJobHandler', '', 'SERIAL_EXECUTION', 0, 0, 'BEAN', '', 'GLUE代码初始化', '2018-11-03 22:21:31', '');
INSERT INTO `xxl_job_user`(`id`, `username`, `password`, `role`, `permission`) VALUES (1, 'admin', 'e10adc3949ba59abbe56e057f20f883e', 1, NULL);
INSERT INTO `xxl_job_lock` ( `lock_name`) VALUES ( 'schedule_lock');

commit;
```

### 创建docker-compose.yml

```shell
vim /data/xxl-job/docker-compose.yml
```

记得修改mysql的连接地址，accessToken的默认值就是default_token，当然也可以对其进行修改。

```yaml
services:
    xxl-job-admin:
        image: xuxueli/xxl-job-admin:2.4.0
        restart: unless-stopped
        environment:
            - PARAMS=--spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver --spring.datasource.url=jdbc:mysql://ip:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai --spring.datasource.username=root --spring.datasource.password=root --xxl.job.accessToken=default_token
        volumes:
            - /data/xxl-job/logs:/data/applogs
        ports:
            - 7397:8080
        hostname: xxl-job-admin
        container_name: xxl-job-admin
```

### 启动容器

```shell
docker-compose up -d
```

## RabbitMQ

> 访问页面：<http://ip:15672>

### 创建挂载的目录

```shell
mkdir -p /data/rabbitmq/plugins
```

### 先启动一次容器

```shell
docker run -d --name rabbitmq rabbitmq:3.12.12-management
```

### 把容器里面的内容复制出来

```shell
docker cp rabbitmq:/plugins/. /data/rabbitmq/plugins/
```

### 删除容器

```shell
docker rm -f rabbitmq
```

### 创建docker-compose.yml

```shell
vim /data/rabbitmq/docker-compose.yml
```

```yaml
services:
    rabbitmq:
        image: rabbitmq:3.12.12-management
        restart: unless-stopped
        environment:
            - RABBITMQ_DEFAULT_PASS=root
            - RABBITMQ_DEFAULT_USER=root
        volumes:
            - /data/rabbitmq/plugins:/plugins
        ports:
            - 5672:5672
            - 15672:15672
        networks:
            - rabbitmq_net
        hostname: rabbitmq
        container_name: rabbitmq

networks:
    rabbitmq_net: 
        driver: bridge
```

### 启动容器

```shell
docker-compose up -d
```

## RocketMQ

> 访问页面：<http://ip:8180>

### 创建挂载的目录

```shell
mkdir -p /data/rocketmq/namesrv
mkdir -p /data/rocketmq/console/data
mkdir -p /data/rocketmq/broker/{conf,lib}
```

### 创建broker配置文件

```shell
vim /data/rocketmq/broker/conf/broker.conf
```

记得修改brokerIP1为宿主机IP。

```conf
#所属集群名字
brokerClusterName=DefaultCluster

#broker名字,注意此处不同的配置文件填写的不一样,如果在broker-a.properties使用:broker-a,在broker-b.properties使用:broker-b
brokerName=broker-a

#0:表示Master >0:表示Slave
brokerId=0

#nameServer地址,分号分割
#namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
namesrvAddr=rocketmq-namesrv:9876

#启动IP,如果docker报com.alibaba.rocketmq.remoting.exception.RemotingConnectException: connect to <192.168.0.120:10909> failed
#解决方式1:加上一句producer.setVipChannelEnabled(false);
#解决方式2:brokerIP1设置宿主机IP,不要使用docker内部IP
brokerIP1=ip

#在发送消息时,自动创建服务器不存在的topic,默认创建的队列数
defaultTopicQueueNums=4

#是否允许Broker自动创建Topic,建议线下开启,线上关闭
autoCreateTopicEnable=true

#是否允许Broker自动创建订阅组,建议线下开启,线上关闭
autoCreateSubscriptionGroup=true

#Broker对外服务的监听端口
listenPort=10911

#此参数控制是否开启密码,不开启可设置false
aclEnable=true

#删除文件时间点,默认凌晨4点
deleteWhen=04

#文件保留时间,默认48小时
fileReservedTime=120

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824

#ConsumeQueue每个文件默认存30W条,根据业务情况调整
mapedFileSizeConsumeQueue=300000

#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
#storePathRootDir=/home/ztztdata/rocketmq-all-4.1.0-incubating/store
#commitLog 存储路径
#storePathCommitLog=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/commitlog
#消费队列存储
#storePathConsumeQueue=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/consumequeue
#消息索引存储路径
#storePathIndex=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/index
#checkpoint 文件存储路径
#storeCheckpoint=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/checkpoint
#abort 文件存储路径
#abortFile=/home/ztztdata/rocketmq-all-4.1.0-incubating/store/abort
#限制的消息大小
maxMessageSize=65536

#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER

#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

### 创建acl配置文件

```shell
vim /data/rocketmq/broker/conf/plain_acl.yml
```

accessKey和secretKey需要大于6位。

 ```yaml
#全局白名单,如果配置了则不需要走acl校验,慎重配置
globalWhiteRemoteAddresses:
# - 47.100.93.*
# - 156.254.120.*
 
accounts:
  - accessKey: rocketmquser
    secretKey: 12345678
    whiteRemoteAddress:
    admin: false
    defaultTopicPerm: DENY
    defaultGroupPerm: SUB
    topicPerms:
      - topicA=DENY
      - topicB=PUB|SUB
      - topicC=SUB
    groupPerms:
      - groupA=DENY
      - groupB=PUB|SUB
      - groupC=SUB

  - accessKey: rocketmqadmin
    secretKey: 12345678
    whiteRemoteAddress: 
    admin: true
 ```

### 创建console配置文件

```shell
vim /data/rocketmq/console/data/users.properties
```

```conf
#用户名和密码规则「用户名=密码,权限」1:管理员 0:普通用户
admin=123456,1
```

### 创建docker-compose.yml

```shell
vim /data/rocketmq/docker-compose.yml
```

```yaml
services:
    rocketmq-namesrv:
        image: foxiswho/rocketmq:4.8.0
        restart: unless-stopped
        command: [ "sh", "mqnamesrv" ]
        environment:
            JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -Xms128M -Xmx128M -Xmn128m"
        volumes:
            - /data/rocketmq/namesrv/logs:/home/rocketmq/logs
            - /data/rocketmq/namesrv/store:/home/rocketmq/store
        ports:
            - 9876:9876
        networks:
            rocketmq_net:
                aliases:
                    - rocketmq-namesrv
        hostname: rocketmq-namesrv
        container_name: rocketmq-namesrv

    rocketmq-broker:
        image: foxiswho/rocketmq:4.8.0
        restart: unless-stopped
        command: [ "sh", "mqbroker", "-c", "/etc/rocketmq/broker.conf" ]
        environment:
            JAVA_OPT_EXT: "-Duser.home=/home/rocketmq -Xms128m -Xmx128m -Xmn128m"
        volumes:
            - /data/rocketmq/broker/logs:/home/rocketmq/logs
            - /data/rocketmq/broker/store:/home/rocketmq/store
            - /data/rocketmq/broker/conf/plain_acl.yml:/home/rocketmq/rocketmq-4.8.0/conf/plain_acl.yml
            - /data/rocketmq/broker/conf/broker.conf:/etc/rocketmq/broker.conf
        ports:
            - 10909:10909
            - 10911:10911
        depends_on:
            - rocketmq-namesrv
        networks:
            rocketmq_net:
                aliases:
                    - rocketmq-broker
        hostname: rocketmq-broker
        container_name: rocketmq-broker

    rocketmq-console:
        image: iamverygood/rocketmq-console:4.7.1
        restart: unless-stopped
        environment:
            JAVA_OPTS: "-Drocketmq.namesrv.addr=rocketmq-namesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false -Drocketmq.config.loginRequired=true -Drocketmq.config.aclEnabled=true -Duser.timezone='Asia/Shanghai' -Drocketmq.config.accessKey=rocketmqadmin -Drocketmq.config.secretKey=12345678"
        volumes:
            - /data/rocketmq/console/data:/tmp/rocketmq-console/data
            - /etc/localtime:/etc/localtime
        ports:
            - 8180:8080
        depends_on:
            - rocketmq-namesrv
        networks:
            rocketmq_net:
                aliases:
                    - rocketmq-console
        hostname: rocketmq-console
        container_name: rocketmq-console

networks:
    rocketmq_net:
        name: rocketmq_net
        driver: bridge
```

如果你的acl密码改了，记得把yml里的console帐号密码也一同更改。

### 授予目录权限

```shell
chmod -R 777 /data/rocketmq/namesrv/
chmod -R 777 /data/rocketmq/broker/
chmod -R 777 /data/rocketmq/console/
```

### 启动容器

```shell
docker-compose up -d
```

注意，如果第一次启动不成功，是因为broker需要创建一堆文件，没有权限，再执行一遍权限命令。

```shell
chmod -R 777 /data/rocketmq/namesrv/
chmod -R 777 /data/rocketmq/broker/
chmod -R 777 /data/rocketmq/console/

docker-compose up --force-recreate -d
```

## Kafka

> 访问页面：<http://ip:9001>

### 创建挂载的目录

```shell
mkdir -p /data/zookeeper/{data,conf,logs}
mkdir -p /data/kafka/kafka-map/data
```

### 创建挂载的配置文件

```shell
vim /data/zookeeper/conf/zoo.cfg
```

```conf
#数据目录
dataDir=/data

#事务日志目录
dataLogDir=/datalog

#控制超时和计时
tickTime=2000

#初始化连接限制时间
initLimit=5

#同步限制时间
syncLimit=2

#自动清理快照文件的保留数量
autopurge.snapRetainCount=3

#自动清理快照文件的时间间隔(设置为0表示禁用自动清理)
autopurge.purgeInterval=0

#允许的最大客户端连接数
maxClientCnxns=60

#单机版模式开启
standaloneEnabled=true

#启用Zookeeper服务器的管理界面
admin.enableServer=true

#Zookeeper服务器的配置
server.1=localhost:2888:3888;2181
```

### 创建docker-compose.yml

```shell
vim /data/kafka/docker-compose.yml
```

记得修改kafka-server为设置宿主机IP。

```yaml
services:
    zookeeper:
        image: zookeeper
        restart: unless-stopped
        volumes:
            - /data/zookeeper/data:/data
            - /data/zookeeper/conf:/conf
            - /data/zookeeper/logs:/datalog
        ports:
            - 2181:2181
        networks:
            - kafka_net
        hostname: zookeeper
        container_name: zookeeper

    kafka-server:
        image: bitnami/kafka
        restart: unless-stopped
        environment:
            - ALLOW_PLAINTEXT_LISTENER=yes
            - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
            - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://ip:9092
        ports:
            - 9092:9092
        networks:
            - kafka_net
        hostname: kafka-server
        container_name: kafka-server

    kafka-map:
        image: dushixiang/kafka-map
        restart: unless-stopped
        environment:
            - DEFAULT_USERNAME=root
            - DEFAULT_PASSWORD=root
        volumes:
            - /data/kafka/kafka-map/data:/usr/local/kafka-map/data
        ports:
            - 9001:8080
        networks:
            - kafka_net
        hostname: kafka-map
        container_name: kafka-map

networks:
    kafka_net:
        driver: bridge
```

### 启动容器

```shell
docker-compose up -d
```
