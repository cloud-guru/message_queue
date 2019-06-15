## Operator with kafka Topics
### List existing topics
 `bin/kafka-topics.sh --zookeeper localhost:2181 --list`

### Describe a topic
  `bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic mytopic `
### Purge a topic
 `bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic mytopic --config retention.ms=1000`
 
... wait a minute ...

 `bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic mytopic --delete-config retention.ms`
 
### Delete a topic
 `bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic mytopic`
 
### Get number of messages in a topic
 `bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -1 --offsets 1 | awk -F  ":" '{sum += $3} END {print sum}'`
 
### Get the earliest offset still in a topic
`bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -2`

### Get the latest offset still in a topic
`bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic mytopic --time -1`

### Consume messages with the console consumer
`bin/kafka-console-consumer.sh --new-consumer --bootstrap-server localhost:9092 --topic mytopic --from-beginning`

## Get the consumer offsets for a topic
`bin/kafka-consumer-offset-checker.sh --zookeeper=localhost:2181 --topic=mytopic --group=my_consumer_group`

### Read from __consumer_offsets

Add the following property to `config/consumer.properties`:
`exclude.internal.topics=false`

`bin/kafka-console-consumer.sh --consumer.config config/consumer.properties --from-beginning --topic __consumer_offsets --zookeeper localhost:2181 --formatter "kafka.coordinator.GroupMetadataManager\$OffsetsMessageFormatter"`

## Kafka Consumer Groups

### List the consumer groups known to Kafka
`bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --list`  (old api)

`bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --list` (new api)

### View the details of a consumer group 
`bin/kafka-consumer-groups.sh --zookeeper localhost:2181 --describe --group <group name>`

## kafkacat

### Getting the last five message of a topic
`kafkacat -C -b localhost:9092 -t mytopic -p 0 -o -5 -e`

## Zookeeper

### Starting the Zookeeper Shell

`bin/zookeeper-shell.sh localhost:2181`

basic command cli zookeeper : http://www.corejavaguru.com/bigdata/zookeeper/cli

## Reassign partition to broker

Sử dụng kafka-reassign-partitions.sh tool với 3 tùy chọn:

* generate: Đưa vào list topic cần chuyển paritition, trả về danh sách gợi ý (plan) nên đưa partition nào vào broker nào
* execute:  Thực hiện reassignment dưa trên plan
* verify:   Đưa vào plan, check xem quá trình reassign đã thành công hay thất bại thế nào

Bước 1: Tạo danh sách topic:

```
cat topics-to-move.json
{"topics": [{"topic": "foo1"},
            {"topic": "foo2"}],
"version":1
}
```
Bước 2: tạo plan:

```
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --topics-to-move-json-file topics-to-move.json --broker-list "5,6" --generate
Current partition replica assignment
 
{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[1,2]},
              {"topic":"foo1","partition":0,"replicas":[3,4]},
              {"topic":"foo2","partition":2,"replicas":[1,2]},
              {"topic":"foo2","partition":0,"replicas":[3,4]},
              {"topic":"foo1","partition":1,"replicas":[2,3]},
              {"topic":"foo2","partition":1,"replicas":[2,3]}]
}
 
Proposed partition reassignment configuration
 
{"version":1,
"partitions":[{"topic":"foo1","partition":2,"replicas":[5,6]},
              {"topic":"foo1","partition":0,"replicas":[5,6]},
              {"topic":"foo2","partition":2,"replicas":[5,6]},
              {"topic":"foo2","partition":0,"replicas":[5,6]},
              {"topic":"foo1","partition":1,"replicas":[5,6]},
              {"topic":"foo2","partition":1,"replicas":[5,6]}]
}
```

Lưu thông tin từ sau dòng ``Proposed...`` vào file  expand-cluster-reassignment.json

Bước 3: Thực hiện:

```
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file expand-cluster-reassignment.json --execute
```

Bước 4: kiểm tra:

```
bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file expand-cluster-reassignment.json --verify
Status of partition reassignment:
Reassignment of partition [foo1,0] completed successfully
Reassignment of partition [foo1,1] is in progress
Reassignment of partition [foo1,2] is in progress
Reassignment of partition [foo2,0] completed successfully
Reassignment of partition [foo2,1] completed successfully
Reassignment of partition [foo2,2] completed successfully
```

Tương tự, thao tác tăng số replicate của một topic cũng làm như các bước trên.

```
cat increase-replication-factor.json
{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5,6,7]}]}

bin/kafka-reassign-partitions.sh --zookeeper localhost:2181 --reassignment-json-file increase-replication-factor.json --execute

Current partition replica assignment
{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5]}]}
 
Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions
{"version":1,
"partitions":[{"topic":"foo","partition":0,"replicas":[5,6,7]}]}
```

## Ref


https://www.ibm.com/support/knowledgecenter/en/SSCVHB_1.2.0/admin/tnpi_reassign_partitions.html

https://blog.imaginea.com/how-to-rebalance-topics-in-kafka-cluster/

https://cwiki.apache.org/confluence/display/KAFKA/Replication+tools

https://kafka.apache.org/documentation/#basic_ops

https://gist.github.com/sonhmai/5b2b4455162c808c091b661aeb675625