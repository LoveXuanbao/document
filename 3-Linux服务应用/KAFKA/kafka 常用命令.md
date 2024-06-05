### 使用证书查看topic

```
docker run \
--rm -it --entrypoint=/bin/bash \
harbor.service.moebius.com/tarzan/library/wurstmeister/kafka:2.8.1


cd /opt/kafka && vi saal.properties


sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
username="wmy" \
password="wmytest";
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN


/opt/kafka/bin/kafka-topics.sh --bootstrap-server 11.11.184.185:9092,11.11.184.186:9092,11.11.184.187:9092 --list --command-config saal.properties


/opt/kafka/bin/kafka-topics.sh --bootstrap-server 11.11.127.101:9092 --create --topic wmytest1 --partitions 6 --replication-factor 3 --command-config saal.properties


/opt/kafka/bin/kafka-topics.sh --bootstrap-server 11.11.127.101:9092 --describe --topic wmytest1 --command-config saal.properties


/opt/kafka/bin/kafka-console-producer.sh --bootstrap-server 11.11.127.101:9092 --producer.config saal.properties --topic wmytest1


/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server 11.11.127.101:9092 --consumer.config saal.properties --topic wmytest1 --from-beginning



/opt/kafka/bin/kafka-topics.sh --describe --bootstrap-server 11.11.127.101:9092,11.11.127.102:9092,11.11.127.103:9092 --topic jldc-packagespro-httpevent-avro --command-config saal.properties

```