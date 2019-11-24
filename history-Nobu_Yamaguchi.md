cp ~/w205/course-content/11-Storing-Data-III/docker-compose.yml .
cp ~/w205/course-content/12-Querying-Data-II/docker-compose.yml .
docker-compose up -d

docker-compose ps
```
             Name                          Command            State               Ports             
----------------------------------------------------------------------------------------------------
project3nobuyamaguchi_cloudera_   cdh_startup_script.sh       Up      11000/tcp, 11443/tcp,         
1                                                                     19888/tcp, 50070/tcp,         
                                                                      8020/tcp, 8088/tcp, 8888/tcp, 
                                                                      9090/tcp                      
project3nobuyamaguchi_kafka_1     /etc/confluent/docker/run   Up      29092/tcp, 9092/tcp           
project3nobuyamaguchi_mids_1      /bin/bash                   Up      0.0.0.0:5000->5000/tcp,       
                                                                      8888/tcp                      
project3nobuyamaguchi_spark_1     docker-entrypoint.sh bash   Up      0.0.0.0:8888->8888/tcp        
project3nobuyamaguchi_zookeeper   /etc/confluent/docker/run   Up      2181/tcp, 2888/tcp, 32181/tcp,
_1                                                                    3888/tcp   
```

docker ps -a
```
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS                                                                                NAMES
36e787318171        confluentinc/cp-kafka:latest       "/etc/confluent/dock…"   3 minutes ago       Up 3 minutes        9092/tcp, 29092/tcp                                                                  project3nobuyamaguchi_kafka_1
becebddc31e3        midsw205/spark-python:0.0.5        "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        0.0.0.0:8888->8888/tcp                                                               project3nobuyamaguchi_spark_1
3056bfd9065c        confluentinc/cp-zookeeper:latest   "/etc/confluent/dock…"   3 minutes ago       Up 3 minutes        2181/tcp, 2888/tcp, 3888/tcp, 32181/tcp                                              project3nobuyamaguchi_zookeeper_1
1a1e14781b82        midsw205/base:0.1.9                "/bin/bash"              3 minutes ago       Up 3 minutes        0.0.0.0:5000->5000/tcp, 8888/tcp                                                     project3nobuyamaguchi_mids_1
3c91e47c01bb        midsw205/cdh-minimal:latest        "cdh_startup_script.…"   3 minutes ago       Up 3 minutes        8020/tcp, 8088/tcp, 8888/tcp, 9090/tcp, 11000/tcp, 11443/tcp, 19888/tcp, 50070/tcp   project3nobuyamaguchi_cloudera_1
```

docker-compose logs -f cloudera
```
Found 2 items
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2019-11-10 04:56 /tmp/hive
```

docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181

```
Created topic events.
```

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

docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning

docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_knife
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_frog
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/ride_a_horse
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/climb_a_mountain
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_knife
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_frog
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/ride_a_horse
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/climb_a_mountain




docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
docker-compose exec mids curl http://localhost:5000/purchase_a_knife
docker-compose exec mids curl http://localhost:5000/ride_a_horse

docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
```
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_knife", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_frog", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "ride_horse", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "climb_mountain", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
% Reached end of topic events [0] at offset 6: exiting
```
docker-compose exec spark bash
ln -s /w205 w205
exit

docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark

docker-compose exec cloudera hadoop fs -ls /tmp/
```
Found 5 items
drwxr-xr-x   - root   supergroup          0 2019-11-10 05:47 /tmp/default_hits
drwxr-xr-x   - root   supergroup          0 2019-11-10 05:47 /tmp/extracted_events
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2019-11-10 04:56 /tmp/hive
drwxr-xr-x   - root   supergroup          0 2019-11-10 05:47 /tmp/sword_purchases
```
docker-compose exec cloudera hadoop fs -ls /tmp/sword_purchases
```
Found 2 items
-rw-r--r--   1 root supergroup          0 2019-11-10 05:47 /tmp/sword_purchases/_SUCCESS
-rw-r--r--   1 root supergroup       1548 2019-11-10 05:47 /tmp/sword_purchases/part-00000-264f825a-86b7-4e79-ab0d-c809af9b7889-c000.snappy.parquet
```
docker-compose exec cloudera hadoop fs -ls /tmp/default_hits
```
Found 2 items
-rw-r--r--   1 root supergroup          0 2019-11-10 05:47 /tmp/default_hits/_SUCCESS
-rw-r--r--   1 root supergroup       1511 2019-11-10 05:47 /tmp/default_hits/part-00000-3d7e0d02-0468-458d-81aa-d031f019549f-c000.snappy.parquet
```

docker-compose exec cloudera hadoop fs -ls /tmp/extracted_events
```
Found 2 items
-rw-r--r--   1 root supergroup          0 2019-11-10 05:47 /tmp/extracted_events/_SUCCESS
-rw-r--r--   1 root supergroup       1823 2019-11-10 05:47 /tmp/extracted_events/part-00000-22150a3a-4b8b-4cda-bffb-6a13b1ee3fd6-c000.snappy.parquet
```

docker-compose down

docker-compose ps

docker ps -a

docker-compose exec spark bash

root@45c1709d5211:/spark-2.2.0-bin-hadoop2.6# ln -s /w205 w205
root@45c1709d5211:/spark-2.2.0-bin-hadoop2.6# exit

docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark



