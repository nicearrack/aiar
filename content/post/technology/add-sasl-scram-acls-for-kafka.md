---
title: "Kafka 启用用户鉴权功能"
description: 记录为果奔的kafka加用户鉴权的过程。
date: 2024-01-11T10:29:00+08:00
image: https://s11.ax1x.com/2024/01/11/pF9gP1K.jpg
math: 
license: 
hidden: false
comments: true
draft: false
categories:
    - 技术
tags:
    - Kafka
    - Linux
---

## 目标
将使用 [SASL/SCRAM](https://kafka.apache.org/documentation/#security_sasl_scram) 认证管理协议进行用户管理，密码采用 `SCRAM-SHA-256` 作加密处理。[中文文档](https://www.orchome.com/1946)

通过 [ACLs](https://kafka.apache.org/documentation/#security_authz) 进行用户权限管理。

## Operation

### Backup Config
``` shell
$ cd [KAFKA_HOME]
$ cp -R config config_bak
```

### Start Zookeeper & Kafka If No Start
`-daemon` 为后台守护启动
``` shell
$ ./bin/zookeeper-server-start.sh -daemon ./config/zookeeper.properties
$ ./bin/kafka-server-start.sh -daemon ./config/server.properties
```

### Add Users

``` shell
$ bin/kafka-configs.sh --bootstrap-server localhost:9092 --alter --add-config 'SCRAM-SHA-256=[password=xG0^qO5&]' --entity-type users --entity-name admin
$ bin/kafka-configs.sh --bootstrap-server localhost:9092 --alter --add-config 'SCRAM-SHA-256=[iterations=8192,password=hL7!rV4^]' --entity-type users --entity-name thtf
$ bin/kafka-configs.sh --bootstrap-server localhost:9092 --describe --entity-type users
```

### Configuring Kafka Brokers
```
// kafka-admin-jaas.conf

KafkaServer {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="admin"
    password="123456";
};

KafkaClient {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="admin"
    password="123456";
};
```

### Configure Kafka Server SASL Properties
``` shell
$ vi config/server.properties

# config/server.properties
listeners=SASL_PLAINTEXT://0.0.0.0:9092
advertised.listeners=SASL_PLAINTEXT://192.168.1.237:9092
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
sasl.enabled.mechanisms=SCRAM-SHA-256
authorizer.class.name=kafka.security.authorizer.AclAuthorizer
super.users=User:admin
```

### Configure Kafka Client SASL Properties
``` shell
$ vi config/sasl.conf

# config/sasl.conf
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
```

### [Optional] Create Start Shell for Kafka Service with JAAS
** this way is the same as using .env and original start shell**
``` shell
$ cp ./bin/kafka-server-start.sh ./bin/kafka-jaas-server-start.sh
$ vim ./bin/kafka-jaas-server-start.sh

# ./bin/kafka-jaas-server-start.sh
# to the end of file
export JAAS_CONF="-Djava.security.auth.login.config=$base_dir/../config/kafka-admin-jaas.conf"
exec $base_dir/kafka-run-class.sh $EXTRA_ARGS $JAAS_CONF kafka.Kafka "$@"
```

### Control KAFKA_OPTS by .env
**Notice: Must register KAFKA_OPTS param like below to ENVIRONMENT if you want to use any operation in ./bin **
``` shell
# .env
export KAFKA_OPTS="-Djava.security.auth.login.config=./config/kafka-admin-jaas.conf"
```

### Restart Service
**Notice: Command order can NOT be reversed**
``` shell
$ cd [KAFKA_HOME]
$ ./bin/kafka-server-stop.sh
$ ./bin/zookeeper-server-stop.sh
$ source .env
$ ./bin/zookeeper-server-start.sh -daemon ./config/zookeeper.properties
$ ./bin/kafka-server-start.sh -daemon ./config/server.properties
```

### Add ACLs for Users
**Notice: Must set this AFTER configuring Authentication and restarting kafka service**

producer
``` shell
$ bin/kafka-acls.sh --bootstrap-server localhost:9092 --command-config config/sasl.conf --add --allow-principal User:test-user --producer --topic test-topic
```

consumer
``` shell
$ bin/kafka-acls.sh --bootstrap-server localhost:9092 --command-config config/sasl.conf --add --allow-principal User:test-user --consumer --topic test-topic --group test-group
```
