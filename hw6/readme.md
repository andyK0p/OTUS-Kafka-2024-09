1. запустил docker-compose (compose.png)
docker compose up -d
docker compose ps -a

2. проверил топики kafka (kafka-topics.png)
docker exec kafka1 kafka-topics --list --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094

3. подключился к контейнеру postgres, создал тестовую таблицу clients и загрузил данные из файла /data/Data.csv, потом проверил первые 5 строк (pg-create.png)
docker exec -ti postgres psql -U postgres
CREATE TABLE clients (id int PRIMARY KEY, first_name text, last_name text, gender text, card_number text, bill numeric(7,2), created_date timestamp, modified_date timestamp);
COPY clients FROM '/data/Demo.csv' WITH (FORMAT csv, HEADER true);
SELECT * FROM clients LIMIT 5;

4. создал clients-connector и проверил его доступность в Kafka Connect (connect.png)
curl -X POST --data-binary "@clients.json" -H "Content-Type: application/json" http://localhost:8083/connectors
curl http://localhost:8083/connectors
curl http://localhost:8083/connectors/clients-connector/status

5. снова проверил топики кафки и убедился что добавился новый топик postgres.clients (kafka-topics1.png)
docker exec kafka1 kafka-topics --list --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094

6. прочитал топик postgres.clients с начала и вывел 1000 сообщений (consumer.png)
docker exec kafka1 kafka-console-consumer --topic postgres.clients --bootstrap-server kafka1:19092,kafka2:19093,kafka3:19094 --from-beginning --property print.offset=true

7. сделал UPDATE записи с id 262 в БД clients, изменил bill = 5000 и обновил modified_date, чтобы запись попала в топик postgres.clients (updated262.png)

8. добавил в БД новую запись с id 1001
INSERT INTO clients VALUES (1001, 'Andrey', 'Koptev', 'Male', '1231313133131', 13500.0, current_timestamp(0), current_timestamp(0));

9. убедился, что новое сообщение попало в топик postgres.clients (inserted1001.png)
