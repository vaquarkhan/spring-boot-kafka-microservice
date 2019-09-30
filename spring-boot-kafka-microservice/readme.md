### Spring Kafka sample :
- https://github.com/spring-projects/spring-kafka
- https://github.com/spring-projects/spring-kafka/tree/master/samples
- https://jimolonely.github.io/2019/03/19/java/036-springboot-kafkatemplate/
- https://thepracticaldeveloper.com/2018/11/24/spring-boot-kafka-config/
- https://github.com/singhmanishkumar3007/springkafkaproducerconsumer
- https://www.youtube.com/watch?v=XfUo66E5K-g
- https://spring.io/blog/2017/02/06/springone-platform-2016-replay-spring-for-apache-kafka
---------------------------------------------------

### Download Kafka :
https://docs.confluent.io/current/installation/installing_cp/zip-tar.html?_ga=2.128620876.153162480.1569811889-901763877.1569811889#prod-kafka-cli-install

### Configure Install Kafka
https://docs.confluent.io/current/installation/installing_cp/zip-tar.html?_ga=2.128620876.153162480.1569811889-901763877.1569811889#prod-kafka-cli-install

### SSL:
- https://docs.confluent.io/current/kafka/authentication_ssl.html
- https://www.ibm.com/support/knowledgecenter/en/SSWTQQ_1.2.0/com.ibm.swg.ba.cognos.trade_analytics.1.2.0.doc/t_trd_sslforkafka.html
- https://www.cnblogs.com/felixzh/p/9089508.html

---------------------------------------------------
### How to call kafka using @Async or webflux

 - https://stackoverflow.com/questions/47351435/spring-async-with-completablefuture
 - https://spring.io/guides/gs/async-method/
 - https://howtodoinjava.com/spring-boot2/rest/enableasync-async-controller/
 - https://dzone.com/articles/spring-boot-creating-asynchronous-methods-using-as
 
      

         import org.springframework.scheduling.annotation.Async;
         import org.springframework.stereotype.Component;

					RestTemplate restTemplate;

					@Async
					public CompletableFuture<String> SaveMessage(String inputJson) {

						restTemplate = new RestTemplate();
						//
						saveKafkaMessage(inputJson);
						
						return CompletableFuture.completedFuture(responseEntity.getBody());
					}
					private void saveKafkaMessage(String inputJson) {
						
						HttpHeaders headers = new HttpHeaders();
						
						headers.set(HttpHeaders.AUTHORIZATION, request.getHeader(HttpHeaders.AUTHORIZATION));
						headers.setContentType(MediaType.APPLICATION_JSON);
						headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));
						HttpEntity<String> request = new HttpEntity<String>(datashareRequest, headers);
					
						responseEntity = restTemplate.exchange(URL, HttpMethod.POST, request, String.class);
						log.info("successfully save message asynchronously.");
					}
					
-----------------------------------------------------

### Spring kafka template

- https://dzone.com/articles/magic-of-kafka-with-spring-boot
- https://medium.com/@contactsunny/simple-apache-kafka-producer-and-consumer-using-spring-boot-41be672f4e2b
- https://www.tutorialspoint.com/spring_boot/spring_boot_apache_kafka.htm
- https://www.javainuse.com/spring/spring-boot-apache-kafka-hello-world
- https://www.onlinetutorialspoint.com/spring-boot/sending-spring-boot-kafka-json-message-to-kafka-topic.html


### Config template :

				import java.util.HashMap;
				import java.util.Map;

				import org.apache.kafka.clients.producer.ProducerConfig;
				import org.apache.kafka.common.serialization.StringSerializer;
				import org.springframework.beans.factory.annotation.Qualifier;
				import org.springframework.beans.factory.annotation.Value;
				import org.springframework.context.annotation.Bean;
				import org.springframework.context.annotation.Configuration;
				import org.springframework.kafka.core.DefaultKafkaProducerFactory;
				import org.springframework.kafka.core.KafkaTemplate;
				import org.springframework.kafka.core.ProducerFactory;

				@Configuration
				public class Config {
					
					@Bean
					public ProducerFactory<String, String> producerFactory() {
						Map<String, Object> configs = new HashMap<>();
						configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");//server ip
						configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
						configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,JsonSerializer.class);//      StringSerializer.class
						//SSL config
						configs.put("security.protocol", "securityProtocol");
						configs.put("ssl.key.password", "trust-store-keyPassword");
						configs.put("ssl.keystore.location", "trust-store-keystoreLocation");
						configs.put("ssl.keystore.password", "trust-store-keystorePassword");
						configs.put("ssl.truststore.location", "trust-store-Location");
						configs.put("ssl.truststore.password", "trust-store-Password");
						// others properties
						configs.put("acks", "all");
						configs.put("retries", 0);
						configs.put("batch.size", 16384);
						configs.put("linger.ms", 1);
						configs.put("buffer.memory", 33554432);
						
						
						return new DefaultKafkaProducerFactory<>(configs);
					}
					
					@Bean
					@Qualifier("kafkaTemplate")
					public KafkaTemplate<String, String> kafkaTemplate() {
						return new KafkaTemplate<>(producerFactory());
					}
					
	
				}


### cCalling class
			import java.util.concurrent.ExecutionException;
			import java.util.concurrent.TimeUnit;
			import java.util.concurrent.TimeoutException;
			import org.apache.kafka.common.KafkaException;
			import org.springframework.beans.factory.annotation.Autowired;
			import org.springframework.beans.factory.annotation.Qualifier;
			import org.springframework.kafka.core.KafkaTemplate;
			import org.springframework.kafka.support.SendResult;
			import org.springframework.stereotype.Service;


			@Service
			public class KafkaProducerService {
				
				@Autowired
				@Qualifier("kafkaTemplate")
				private KafkaTemplate<String, String> kafkaTemplate;
				
				
				public KafkaSendResponse sendMessageToKafka(String topic, String message) throws Exception {
					
						result = this.kafkaTemplate.send(topic, message).get(100, TimeUnit.SECONDS);
						return new KafkaResponse(result.getRecordMetadata());
					
				}

			}




