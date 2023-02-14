Команды выполняем на указанных нодах и тогда можно будет добиться требуемого результата. 2 шарда и каждый реплицирован.

### node1

```bash
CREATE DATABASE shard;
CREATE TABLE shard.test (id Int64, event_time DateTime) Engine=ReplicatedMergeTree('/clickhouse/tables/shard1/test', 'replica_1') PARTITION BY toYYYYMMDD(event_time) ORDER BY id;
CREATE TABLE default.test (id Int64, event_time DateTime) ENGINE = Distributed('company_cluster', '', test, rand());
```

### node2

```bash
CREATE DATABASE replica;
CREATE TABLE replica.test (id Int64, event_time DateTime) Engine=ReplicatedMergeTree('/clickhouse/tables/shard1/test', 'replica_2') PARTITION BY toYYYYMMDD(event_time) ORDER BY id;
```


### node3

```bash
CREATE DATABASE shard;
CREATE TABLE shard.test (id Int64, event_time DateTime) Engine=ReplicatedMergeTree('/clickhouse/tables/shard2/test', 'replica_1') PARTITION BY toYYYYMMDD(event_time) ORDER BY id;
CREATE TABLE default.test (id Int64, event_time DateTime) ENGINE = Distributed('company_cluster', '', test, rand());
```

### node4

```bash
CREATE DATABASE replica;
CREATE TABLE replica.test (id Int64, event_time DateTime) Engine=ReplicatedMergeTree('/clickhouse/tables/shard2/test', 'replica_2') PARTITION BY toYYYYMMDD(event_time) ORDER BY id;
```

Для проверки на node1 выполняем:
```bash
INSERT INTO default.test (id, event_time) VALUES (1, today()), (2, today()), (3, now())
INSERT INTO default.test (id, event_time) VALUES (1, today()), (2, today()), (3, now())
INSERT INTO default.test (id, event_time) VALUES (1, today()), (2, today()), (3, now())
INSERT INTO default.test (id, event_time) VALUES (1, today()), (2, today()), (3, now())
INSERT INTO default.test (id, event_time) VALUES (1, today()), (2, today()), (3, now())
```

Затем на node1 и node3, которые главные в шардах 1 и 2:
```
SELECT * FROM default.test
```

И получаем все записи. 

Или же на них:
```
SELECT * FROM shard.test
```

И получаем записи конкретно на этом шарде.

Так же можно на node2 и node4:
```
SELECT * FROM replica.test
```

И увидим копию shard.test с соответстующих им нод.