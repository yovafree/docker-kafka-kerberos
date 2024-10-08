# Kafka with Kerberos Authentication

For a tutorial, see https://docs.hazelcast.com/tutorials/stream-from-kafka-kerberos.

## Quickstart

```bash
docker compose rm -f
docker compose up
```

```bash
docker exec -it hazelcast bin/hz-cli sql
```


```sql
CREATE MAPPING trades (
    id BIGINT,
    ticker VARCHAR,
    price DECIMAL,
    amount BIGINT)
TYPE Kafka
OPTIONS (
    'valueFormat' = 'json-flat',
    'bootstrap.servers' = 'kafkabroker:9092',
    'sasl.mechanism' = 'GSSAPI',
    'security.protocol' = 'SASL_PLAINTEXT',
    'sasl.kerberos.service.name' = 'kafka',
    'sasl.jaas.config' = 'com.sun.security.auth.module.Krb5LoginModule required useTicketCache=true useKeyTab=true storeKey=true keyTab="/mnt/jduke.keytab" principal="jduke@KERBEROS.EXAMPLE";'
);

SELECT ticker, ROUND(price * 100) AS price_cents, amount
  FROM trades
  WHERE price * amount > 100;
  
INSERT INTO trades VALUES
  (1, 'ABCD', 5.5, 10),
  (2, 'EFGH', 14, 20); 
```

See https://docs.hazelcast.com/hazelcast/latest/sql/learn-sql

## Test broker-only Kerberos authentication (GSSAPI)

```bash
docker exec -it broker bash

# create topic and test producer+consumer authentication
kafka-topics --bootstrap-server kafkabroker:9093 --create --topic hztest --command-config /etc/kafka/kafka-client.properties
cat /etc/passwd | kafka-console-producer --bootstrap-server kafkabroker:9093 --topic hztest --producer.config /etc/kafka/kafka-client.properties
kafka-console-consumer --bootstrap-server kafkabroker:9093 --topic hztest --consumer.config /etc/kafka/kafka-client.properties --from-beginning 


kafka-topics --bootstrap-server kafkabroker:9093 --create --topic r_8650 --command-config /etc/kafka/kafka-client.properties
cat /etc/passwd | kafka-console-producer --bootstrap-server kafkabroker:9093 --topic r_8650 --producer.config /etc/kafka/kafka-client.properties
```

Commands to convert JKS to crt & key

keytool -importkeystore -srckeystore kafka.consumer.truststore.jks -destkeystore ca_consumer.pkcs12 -deststoretype PKCS12

openssl pkcs12 -in ca_consumer.pkcs12  -nodes -nocerts -out ca_consumer.key -legacy

openssl pkcs12 -in ca_consumer.pkcs12 -nokeys -out ca_consumer.crt -legacy


keytool -importkeystore -srckeystore kafka.consumer.keystore.jks -destkeystore tls_consumer.pkcs12 -deststoretype PKCS12

openssl pkcs12 -in tls_consumer.pkcs12 -nodes -nocerts -out tls_consumer.key -legacy

openssl pkcs12 -in tls_consumer.pkcs12 -nokeys -out tls_consumer.crt -legacy
