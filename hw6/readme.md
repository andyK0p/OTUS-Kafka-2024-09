1. запустил docker-compose (compose1.png)
docker compose up -d
docker compose ps -a

2. проверил статус Kafka Connect и наличие плагинов коннекторов (plugins.png)
curl http://localhost:8083
curl http://localhost:8083/connector-plugins

3. проверил топики kafka (kafka-topics-before.png)
docker exec kafka1 kafka-topics --list --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094

4. подключился к контейнеру postgres, создал тестовую таблицу customers и загрузил данные (pg-create-customers.png)
docker exec -ti postgres psql -U postgres
CREATE TABLE customers (id INT PRIMARY KEY, name TEXT, age INT);
INSERT INTO customers (id, name, age) VALUES (5, 'Fred', 34);
INSERT INTO customers (id, name, age) VALUES (7, 'Sue', 25);
INSERT INTO customers (id, name, age) VALUES (2, 'Bill', 51);
SELECT * FROM customers;

5. создал customers-connector и проверил его доступность в Kafka Connect (new-connector.png)
curl -X POST --data-binary "@customers.json" -H "Content-Type: application/json" http://localhost:8083/connectors
curl http://localhost:8083/connectors
curl http://localhost:8083/connectors/customers-connector/status

6. снова проверил топики кафки и убедился что добавился новый топик postgres.public.customers (kafka-topics-after.png)
docker exec kafka1 kafka-topics --list --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094

6. прочитал топик postgres.public.customers
docker exec kafka1 kafka-console-consumer --topic postgres.public.customers --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 --property print.offset=true --property print.key=true --from-beginning


7. добавил в БД новую запись (pg-insert-one.png)
INSERT INTO customers VALUES (10, 'Andrey', 38);

9. убедился, что новое сообщение попало в топик postgres.public.customers (inserted.png)

