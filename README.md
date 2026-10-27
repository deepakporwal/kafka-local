# Kafka Local

A local Kafka cluster for development, managed with Docker Compose. The stack
contains three Kafka brokers, ZooKeeper, and Kafka UI.

## Prerequisites

- Docker Desktop (or Docker Engine) with the Docker Compose plugin
- Ports `2181`, `8080`, and `9092`-`9094` available on your machine

## Start the cluster

From the directory containing `docker-compose.yml`, run:

```shell
docker compose up -d
```

Check that the containers are running:

```shell
docker compose ps
```

Kafka may need a little time to become ready after the containers start. If a
Kafka command fails immediately, check the broker logs and try again:

```shell
docker compose logs -f kafka1 kafka2 kafka3
```

Open the Kafka UI at [http://localhost:8080](http://localhost:8080). The
configured cluster is named `local-cluster`.

## Connect to Kafka

Use these bootstrap servers depending on where your client runs:

| Client location | Bootstrap servers |
| --- | --- |
| Your host machine | `localhost:9092,localhost:9093,localhost:9094` |
| A container on the Compose network | `kafka1:29092,kafka2:29093,kafka3:29094` |

The examples below run Kafka's command-line tools inside the `kafka1`
container, so they use `kafka1:29092`.

## Create a topic

Create a topic named `quickstart` with three partitions and a replication
factor of three:

```shell
docker compose exec kafka1 kafka-topics --bootstrap-server kafka1:29092 --create --topic quickstart --partitions 3 --replication-factor 3
```

List topics or inspect the new topic:

```shell
docker compose exec kafka1 kafka-topics --bootstrap-server kafka1:29092 --list
docker compose exec kafka1 kafka-topics --bootstrap-server kafka1:29092 --describe --topic quickstart
```

## Produce and consume messages

Open two terminals in the project directory. In the first terminal, start a
consumer that reads existing messages from the beginning:

```shell
docker compose exec kafka1 kafka-console-consumer --bootstrap-server kafka1:29092 --topic quickstart --from-beginning
```

In the second terminal, start the producer:

```shell
docker compose exec kafka1 kafka-console-producer --bootstrap-server kafka1:29092 --topic quickstart
```

Type a message and press Enter to send it. For example:

```text
Hello from my local Kafka cluster!
```

The message appears in the consumer terminal. Stop either interactive command
with `Ctrl+C`.

### Produce and consume a keyed JSON message

Create a separate topic for the order example:

```shell
docker exec -it kafka1 kafka-topics --bootstrap-server localhost:29092 --create --topic tours-inquiry --partitions 3 --replication-factor 3
```

In one terminal, start a consumer that prints each message key before its
value:

```powershell
docker exec -it kafka1 kafka-console-consumer `
  --bootstrap-server localhost:29092 `
  --topic tours-inquiry `
  --group tours-inquiry-consumer `
  --from-beginning `
  --property print.key=true `
  --property key.separator=" : "
```

In another terminal, start a producer configured to parse the key from each
line:

```powershell
docker exec -it kafka1 kafka-console-producer `
  --bootstrap-server localhost:29092 `
  --topic tours-inquiry `
  --property parse.key=true `
  --property key.separator=" : "
```

Enter this line in the producer and press Enter:

```text
order-101 : {"orderId":101,"customer":"Amit","amount":1500}
```

The consumer displays the key and JSON value:

```text
order-101 : {"orderId":101,"customer":"Amit","amount":1500}
```

## Stop the cluster

Stop and remove the containers while keeping Kafka and ZooKeeper data in their
named volumes:

```shell
docker compose down
```

To also delete the persisted data and start with a clean cluster next time:

```shell
docker compose down --volumes
```

> **Warning:** `docker compose down --volumes` permanently deletes the
> cluster's persisted Kafka and ZooKeeper data.