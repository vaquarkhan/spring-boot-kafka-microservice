### 1.Producer
	1.request.required.acks=[0,1,all/-1]  0 no acknowledgement but ver fast, 1 acknowledged after leader commits, all acknowledged after replicated
	
	2.use Async producer	- use callback for the acknowledgement, using property  producer.type=1
	3.Batching data - send multiple messages together.
		batch.num.messages
		queue.buffer.max.ms
	4.Compression for Large files - gzip, snappy supported
		very large files can be stored in shared location and just the file path can be logged by the kafka producer.
		
	5.timeouts/retry settings - defaults may be longer, so change based on use case - using property request.timout.ms
	
### 2.Brokers:
	1. choose high partitions - as cannot have more consumers than partitions.
	2. try 1 partition per physical disc, to avoid IO bottleneck.
	3. load balance partitions - use tool - 
		kafka-reassign-partitions.sh --generate (the plan) --execute --verify.
	some parameters to tune:
		num.io.thread - at least as many threads as the disks.
		log.flush.interval - higher interval will increase the speed, but risk data loss of server crashes.
		
### 3.Consumers:
	1.have as many consumers in a group as there are partitions.
	2.keep up with the number of producers
	3.adding more consumers to a group will enhance performance but not adding a consumer group.
	4.checkpoint interval 
		replica.high.watermark.checkpoint.interval.ms - high value will increase performance, as checkpointing is done infrequently - slight data risk possibility
		
### 4.pipeline performance:
	The other systems in the pipeline, for example data is written to HDFS, should also perform as good as Kafka.



### extra:	
kafka vs other messaging system:
	1.Decoupling of producer & consumer	- data is persisted when consumers go down, work periodically like ETL
	2.Data is not stored per consumer like in a queue - data saved once - any number of consumer can independently read with their own offset values.
	3.Replication is by default - not in just specialized cases with complicated configuration.
	
		
- https://github.com/apache/metron/blob/master/metron-platform/Performance-tuning-guide.md
- https://github.com/DataDog/the-monitor/blob/master/kafka/monitoring-kafka-performance-metrics.md
- https://github.com/jaceklaskowski/kafka-workshop
- https://github.com/jaceklaskowski/kafka-notebook

------------------------------------------------------

### Producer

### Setup
bin/kafka-topics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test-rep-one --partitions 6 --replication-factor 1
bin/kafka-topics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test --partitions 6 --replication-factor 3

### Single thread, no replication

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test7 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

### Single-thread, async 3x replication

bin/kafktopics.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --create --topic test --partitions 6 --replication-factor 3
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test6 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

### Single-thread, sync 3x replication

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=-1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=64000

### Three Producers, 3x async replication

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

### Throughput Versus Stored Data

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

### Effect of message size

for i in 10 100 1000 10000 100000;
do
echo ""
echo $i
bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test $((1000*1024*1024/$i)) $i -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=128000
done;

### Consumer
Consumer throughput

bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1

3 Consumers

On three servers, run:
bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1

### End-to-end Latency

bin/kafka-run-class.sh kafka.tools.TestEndToEndLatency esv4-hcl198.grid.linkedin.com:9092 esv4-hcl197.grid.linkedin.com:2181 test 5000

### Producer and consumer

bin/kafka-run-class.sh org.apache.kafka.clients.tools.ProducerPerformance test 50000000 100 -1 acks=1 bootstrap.servers=esv4-hcl198.grid.linkedin.com:9092 buffer.memory=67108864 batch.size=8196

bin/kafka-consumer-perf-test.sh --zookeeper esv4-hcl197.grid.linkedin.com:2181 --messages 50000000 --topic test --threads 1

