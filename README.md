## Real-Time-Stock-Analysis-System

Prerequisites for backend.

- Python
- Docker Toolbox 
- zookeeper
- Kafka
- Redis
- Spark

Prerequisites for frontend.

- socket.io
- express
- minimist
- smoothie

#### Create a docker machine called "bigdata"
#### Name of my docker-machine : bigdata

Check the list of docker machine

```
$ docker-machine ls
```

Start docker machine 

```
$ docker-machine start bigdata
```

Steps 1: Fetch data from stock (terminal 1)

```
$ eval $(docker-machine env bigdata)
$ docker rm -f $(docker ps -a -q)
$ docker ps
$ ./local-setup.sh bigdata
$ python simple-data-producer.py AAPL stock-analyzer 192.168.99.100:9092

```

Step 2: Start streaming processor (terminal 2)

```
$ spark-submit --jars spark-streaming-kafka-0-8-assembly_2.11-2.0.0.jar stream-processing.py stock-analyzer average-stock-price 192.168.99.100:9092
```

Step 3: Start redis producer (terminal 3)

```
$ python redis-publisher.py average-stock-price 192.168.99.100:9092 average-stock-price 192.168.99.100 6379
```

Step 4: Node.js & Visualization (terminal 4)

```
$ node index.js --port=3000 --redis_host=192.168.99.100 --redis_port=6379 --subscribe_topic=average-stock-price
```

### Other Information 

Start a Zookeeper Container:

```
$ docker run -d -p 2181:2181 -p 2888:2888 -p 3888:3888 --name zookeeper confluent/zookeeper
```

Start a Kafka Container

```
$ docker run -d -p 9092:9092 -e KAFKA_ADVERTISED_HOST_NAME=`docker-machine ip bigdata` -e KAFKA_ADVERTISED_PORT=9092 --name kafka --link zookeeper:zookeeper confluent/kafka
```

Start a Redis Container

```
$ docker run -d -p 6379:6379 --name redis redis:alpine
```
