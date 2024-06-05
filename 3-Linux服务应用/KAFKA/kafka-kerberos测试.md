# 创建kafka_client_jass.conf

- 在 root 目录下编写 `kafka_client_jass.conf`

```shell
cat > kafka_client_jass.conf <<EOF
KafkaClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useTicketCache=true
  renewTicket=true
  serviceName="kafka"
  useKeyTab=true
  keyTab="/etc/security/keytabs/kafka.service.keytab"  #修改为keytab的绝对路径#注释需要去掉
  principal="kafka/bigdata-1@ZETAPROD.COM";  #klist -kt /etc/security/keytabs/kafka.service.keytab查看principal#注释需要去掉
};
EOF
```

- 在 root 目录下编写 `kafka.peoperties`（内容不需要修改）

```shell
cat > kafka.properties <<EOF
security.protocol=SASL_PLAINTEXT
sasl.mechanism=GSSAPI
sasl.kerberos.service.name=kafka
EOF
```

- 执行 export 命令，在同一次会话窗口内生效，不同窗口都需要执行

```shell
export KAFKA_OPTS="-Djava.security.auth.login.config=/root/kafka_client_jass.conf"
```

- 生产命令：topic 为要使用的 topic

```shell
/usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list bigdata-1:6667 --topic demo --producer.config /root/kafka.properties
```

- 消费命令：topic 为要使用的 topic

```shell
/usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server bigdata-1:6667 --from-beginning --topic demo --consumer.config /root/kafka.properties
```

