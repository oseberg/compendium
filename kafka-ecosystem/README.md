### resources
- [kafka](https://kafka.apache.org/intro)
- [kafka connect](https://kafka.apache.org/documentation/#connect)
- [akka streams](https://doc.akka.io/docs/akka/2.5/stream/index.html)
- [debezium](http://debezium.io/)
- [logstash kafka in](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-kafka.html)
- [logstash kafka out](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-kafka.html)

#### reading messages from an existing kafka topic

```
 docker run --rm confluentinc/cp-kafka \
  kafka-console-consumer \
  --bootstrap-server <kafka server> \
  --topic <topic> \
  --from-beginning \
  --max-messages 42
```
