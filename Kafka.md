## Kafka
- Kafka consumers
  - maybe defined `-Dmicronaut.environments=k8s` in Java env, then it's able to be same consumerGroup
  - define kafka related settings in appConfig.properties, and it is set in java_opts
  - concurrency: the number of consumers in each service
```
@KafkaListener(
    batch = true,
    groupId = "consumerGroup",
    offsetStrategy = OffsetStrategy.DISABLED,
    threadsValue = concurrency
)
public class kafkaListener {
    @Retryable(
        predicate=some.class,
        attempts=1000,
        delay=20s,
        includes=...,
        excludes=...

    )
    @Topic("someTopic")
    public void processMessages(List<ConsumerRecord<String, someDto>> records,
    Acknowledgement acknowledgement) {
        // say set max.poll.records in appConfig.properties
        records.forEach(
            record-> {
                // process each record
                acknowledgement.ack();
            }
        )'
    }
}
```
- Kafka Producer
```
// apache kafka consumer
public class KafkaProducerService {
    private final Producer<String, String> kafkaProducer;
    public KafkaProducerService(
        @KafkaClient(id = "kafka-producer") Producer<String, String> producer
    ) {
        this.kafkaProducer = producer;
    }

    public void send(String topic,
        String key,
        String value,
        ConsumerRecord<String, String> payload
    ) {
        ProducerRecord<String, String> payload = new ProducerRecord<>(topic, key, value);
        for (Header header : payload.headers()) {
            kafkaPayload.headers().add(header);
        }
        kafkaProducer.send(kafkaPayload);
    }
}
```