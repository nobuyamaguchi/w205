cp ~/w205/course-content/10-Transforming-Streaming-Data/docker-compose.yml .

docker-compose up -d

docker-compose ps
```
              Name                           Command            State                    Ports                 
---------------------------------------------------------------------------------------------------------------
project3nobuyamaguchi_kafka_1       /etc/confluent/docker/run   Up      29092/tcp, 9092/tcp                    
project3nobuyamaguchi_mids_1        /bin/bash                   Up      0.0.0.0:5000->5000/tcp, 8888/tcp       
project3nobuyamaguchi_spark_1       docker-entrypoint.sh bash   Up      0.0.0.0:8888->8888/tcp                 
project3nobuyamaguchi_zookeeper_1   /etc/confluent/docker/run   Up      2181/tcp, 2888/tcp, 32181/tcp, 3888/tcp
```

docker ps -a
```
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS                                     NAMES
b908037a1bad        confluentinc/cp-kafka:latest       "/etc/confluent/dock…"   9 minutes ago       Up 9 minutes        9092/tcp, 29092/tcp                       project3nobuyamaguchi_kafka_1
4f50c793c5a6        midsw205/base:0.1.8                "/bin/bash"              9 minutes ago       Up 9 minutes        0.0.0.0:5000->5000/tcp, 8888/tcp          project3nobuyamaguchi_mids_1
0f52404b80dd        confluentinc/cp-zookeeper:latest   "/etc/confluent/dock…"   9 minutes ago       Up 9 minutes        2181/tcp, 2888/tcp, 3888/tcp, 32181/tcp   project3nobuyamaguchi_zookeeper_1
45c1709d5211        midsw205/spark-python:0.0.5        "docker-entrypoint.s…"   9 minutes ago       Up 9 minutes        0.0.0.0:8888->8888/tcp                    project3nobuyamaguchi_spark_1
```

docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181

cp ~/w205/course-content/10-Transforming-Streaming-Data/game_api_with_extended_json_events.py .

docker-compose exec mids env FLASK_APP=/w205/project-3-nobuyamaguchi/game_api_with_extended_json_events.py flask run --host 0.0.0.0

game_api_with_extended_json_events.py
```
!/usr/bin/env python
import json
from kafka import KafkaProducer
from flask import Flask, request

app = Flask(__name__)
producer = KafkaProducer(bootstrap_servers='kafka:29092')


def log_to_kafka(topic, event):
    event.update(request.headers)
    producer.send(topic, json.dumps(event).encode())


@app.route("/")
def default_response():
    default_event = {'event_type': 'default'}
    log_to_kafka('events', default_event)
    return "This is the default response!\n"


@app.route("/purchase_a_sword")
def purchase_a_sword():
    purchase_sword_event = {'event_type': 'purchase_sword'}
    log_to_kafka('events', purchase_sword_event)
    return "Sword Purchased!\n"


@app.route("/purchase_a_frog")
def purchase_a_frog():
    purchase_frog_event = {'event_type': 'purchase_frog'}
    log_to_kafka('events', purchase_frog_event)
    return "Frog Purchased!\n"

@app.route("/purchase_a_knife")
def purchase_a_knife():
    purchase_knife_event = {'event_type': 'purchase_knife'}
    log_to_kafka('events', purchase_knife_event)
    return "Knife Purchased!\n"

@app.route("/ride_a_horse")
def ride_a_horse():
    ride_horse_event = {'event_type': 'ride_horse'}
    log_to_kafka('events', ride_horse_event)
    return "Horse Rided!\n"

@app.route("/climb_a_mountain")
def climb_a_mountain():
    climb_mountain_event = {'event_type', 'climb_mountain'}
    log_to_kafka('events', climb_mountain_event)
    return "Mountain Climbed!\n"
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
docker-compose exec mids curl http://localhost:5000/purchase_a_knife
docker-compose exec mids curl http://localhost:5000/ride_a_horse

docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_frog", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_knife", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "ride_horse", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
% Reached end of topic events [0] at offset 5: exiting

```
docker-compose down

docker-compose ps

docker ps -a

docker-compose exec spark bash

root@45c1709d5211:/spark-2.2.0-bin-hadoop2.6# ln -s /w205 w205
root@45c1709d5211:/spark-2.2.0-bin-hadoop2.6# exit

docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark



