1. запустил zookeeper (zoo.png)
zookeeper-server-start.bat C:/kafka/config/zookeeper.properties 

2. запустил	брокер (server.png)
kafka-server-start.bat C:/kafka/config/server.properties

3. создал топик тест
kafka-topics.bat --create --topic test --bootstrap-server localhost:9092

4. проверил создание топика командой describe (topic.png)
kafka-topics.bat --describe --bootstrap-server localhost:9092

5. записал сообщения в топик с ключами и сепаратором :(producer.png)
kafka-console-producer.bat --topic test --bootstrap-server localhost:9092 --property parse.key=true --property key.separator=:

6. прочитал сообщения из топика test (consumer.png)
kafka-console-consumer.bat --topic test --bootstrap-server localhost:9092 --from-beginning --property print.key=true --property print.offset=true