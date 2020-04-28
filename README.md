# Kafka Adapter

Адаптер предоставляет возможность работать с нужными топиками кафка на чтение и запись сообщений с их подтверждением.

#### Important:
If ConsumerGroupID is empty, then each message would be auto-acked, 
and acquiring (msg.Ack()/msg.Nack()) would return "unavailable when GroupID is not set" error.

#### Example:
```
import (
    queue "github.com/iddqdeika/kafka-adapter"
    "log"
 )
 
 func example() {
    topic := "my_topic"
    consumerGroup := "my_group"
    broker := "my-kafka-host.lan:9092"
    messageToSend := []byte("some message")
    
    cfg := queue.KafkaCfg{
      Concurrency:       100,
      QueueToReadNames:  []string{topic},
      QueueToWriteNames: []string{topic},
      Brokers:           []string{broker},
      ConsumerGroupID:   consumerGroup,
   }

   log.Printf("starting adapter")
   q, err := queue.FromStruct(cfg, queue.DefaultLogger)
   if err != nil{
      log.Fatalf("cant init kafka adapter: %v", err)
   }
   defer q.Close()
   log.Printf("adapter started")

   log.Printf("putting message")
   err = q.Put(topic, messageToSend)
   if err != nil{
      log.Fatalf("cant put message in topic: %v", err)
   }
   log.Printf("message put")

   log.Printf("getting message")
   msg, err := q.Get(topic)
   if err != nil{
      log.Fatalf("cant get message from topic: %v", err)
   }
   message := string(msg.Data())
   log.Printf("message got: %v", message)

   log.Printf("acking message")
   err = msg.Ack()
   if err != nil{
      log.Fatalf("cant ack message: %v", err)
   }
   log.Printf("message acked")
 }
```

### Additional:
FromConfig(cfg Config, logger Logger) - constructor, which uses interface Config
LoadJsonConfig(filename string) - Config implementation, loading data from json file
DefaultLogger - default implementation of logger, just forwards all errors to fmt.Pringf method
