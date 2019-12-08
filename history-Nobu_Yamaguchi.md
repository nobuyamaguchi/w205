
Copy yml file and spin up the cluster
(see docker-compose.yml to see the version after I edited the file)
```
cp ~/w205/course-content/13-Understanding-Data/docker-compose.yml .
docker-compose up -d
```

check the status
```
docker-compose ps
```
```
              Name                             Command               State                   Ports                 
-------------------------------------------------------------------------------------------------------------------
project3nobuyamaguchi_cloudera_1    /usr/bin/docker-entrypoint ...   Up      10000/tcp, 50070/tcp, 8020/tcp,       
                                                                             8888/tcp, 9083/tcp                    
project3nobuyamaguchi_kafka_1       /etc/confluent/docker/run        Up      29092/tcp, 9092/tcp                   
project3nobuyamaguchi_mids_1        /bin/bash                        Up      0.0.0.0:5000->5000/tcp, 8888/tcp      
project3nobuyamaguchi_presto_1      /usr/bin/docker-entrypoint ...   Up      8080/tcp                              
project3nobuyamaguchi_spark_1       docker-entrypoint.sh bash        Up      0.0.0.0:8888->8888/tcp                
project3nobuyamaguchi_zookeeper_1   /etc/confluent/docker/run        Up      2181/tcp, 2888/tcp, 32181/tcp,        
                                                                             3888/tcp   
```
```
docker ps -a
```
```
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS              PORTS                                                                                NAMES
36e787318171        confluentinc/cp-kafka:latest       "/etc/confluent/dock…"   3 minutes ago       Up 3 minutes        9092/tcp, 29092/tcp                                                                  project3nobuyamaguchi_kafka_1
becebddc31e3        midsw205/spark-python:0.0.5        "docker-entrypoint.s…"   3 minutes ago       Up 3 minutes        0.0.0.0:8888->8888/tcp                                                               project3nobuyamaguchi_spark_1
3056bfd9065c        confluentinc/cp-zookeeper:latest   "/etc/confluent/dock…"   3 minutes ago       Up 3 minutes        2181/tcp, 2888/tcp, 3888/tcp, 32181/tcp                                              project3nobuyamaguchi_zookeeper_1
1a1e14781b82        midsw205/base:0.1.9                "/bin/bash"              3 minutes ago       Up 3 minutes        0.0.0.0:5000->5000/tcp, 8888/tcp                                                     project3nobuyamaguchi_mids_1
3c91e47c01bb        midsw205/cdh-minimal:latest        "cdh_startup_script.…"   3 minutes ago       Up 3 minutes        8020/tcp, 8088/tcp, 8888/tcp, 9090/tcp, 11000/tcp, 11443/tcp, 19888/tcp, 50070/tcp   project3nobuyamaguchi_cloudera_1
```
showing vitual console for the cloudera hadoop hdfs containe
```
docker-compose logs -f cloudera
```
create a topic
```
docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
```
```
Created topic events.
```
copy game_api_with_extended_json_events.py file to modify
```
cp ~/w205/course-content/10-Transforming-Streaming-Data/game_api_with_extended_json_events.py .
```

run flask
```
docker-compose exec mids env FLASK_APP=/w205/project-3-nobuyamaguchi/game_api_with_extended_json_events.py flask run --host 0.0.0.0
```
game_api_with_extended_json_events.py
```python
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
    purchase_knife_event = {'event_type': 'purchase_knife',
                            'description': 'very sharp knife'}
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

@app.route("/join_a_guild")
def join_a_guild():
    join_guild_event = {'event_type': 'join_guild'}
    log_to_kafka('events', join_guild_event)
    return "Guild Joined!\n"



```

set up to watch kafka
```
docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning
```

Apache Bench to generate data
```
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_knife
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_frog
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/ride_a_horse
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/climb_a_mountain
docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/join_a_guild
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_sword
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_knife
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/purchase_a_frog
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/ride_a_horse
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/climb_a_mountain
docker-compose exec mids ab -n 10 -H "Host: user2.att.com" http://localhost:5000/join_a_guild
```

Curl version
```
docker-compose exec mids curl http://localhost:5000/
docker-compose exec mids curl http://localhost:5000/purchase_a_sword
docker-compose exec mids curl http://localhost:5000/purchase_a_frog
docker-compose exec mids curl http://localhost:5000/purchase_a_knife
docker-compose exec mids curl http://localhost:5000/ride_a_horse
```

```
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_knife", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "purchase_frog", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "ride_horse", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "climb_mountain", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
% Reached end of topic events [0] at offset 6: exiting
```

```
docker-compose exec spark bash
ln -s /w205 w205
exit
```
spin up a notebook
```
docker-compose exec spark env PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS='notebook --no-browser --port 8888 --ip 0.0.0.0 --allow-root' pyspark
```
ckeck hdfs
```
docker-compose exec cloudera hadoop fs -ls /tmp/
```
```
Found 5 items
drwxr-xr-x   - root   supergroup          0 2019-11-10 05:47 /tmp/default_hits
drwxr-xr-x   - root   supergroup          0 2019-11-10 05:47 /tmp/extracted_events
drwxrwxrwt   - mapred mapred              0 2018-02-06 18:27 /tmp/hadoop-yarn
drwx-wx-wx   - root   supergroup          0 2019-11-10 04:56 /tmp/hive
drwxr-xr-x   - root   supergroup          0 2019-11-10 05:47 /tmp/sword_purchases
```
```
docker-compose exec cloudera hadoop fs -ls /tmp/sword_purchases
```
```
Found 2 items
-rw-r--r--   1 root supergroup          0 2019-11-10 05:47 /tmp/sword_purchases/_SUCCESS
-rw-r--r--   1 root supergroup       1548 2019-11-10 05:47 /tmp/sword_purchases/part-00000-264f825a-86b7-4e79-ab0d-c809af9b7889-c000.snappy.parquet
```
```
docker-compose exec cloudera hadoop fs -ls /tmp/default_hits
```
```
Found 2 items
-rw-r--r--   1 root supergroup          0 2019-11-10 05:47 /tmp/default_hits/_SUCCESS
-rw-r--r--   1 root supergroup       1511 2019-11-10 05:47 /tmp/default_hits/part-00000-3d7e0d02-0468-458d-81aa-d031f019549f-c000.snappy.parquet
```
```
docker-compose exec cloudera hadoop fs -ls /tmp/extracted_events
```
```
Found 2 items
-rw-r--r--   1 root supergroup          0 2019-11-10 05:47 /tmp/extracted_events/_SUCCESS
-rw-r--r--   1 root supergroup       1823 2019-11-10 05:47 /tmp/extracted_events/part-00000-22150a3a-4b8b-4cda-bffb-6a13b1ee3fd6-c000.snappy.parquet
```

Query with Presto
```
docker-compose exec presto presto --server presto:8080 --catalog hive --schema default
```
check the tables
```
show tables;
```

```
      Table      
-----------------
 join_events     
 joins           
 purchase_events 
 purchases       
(4 rows)
```

I showed description of two tables below.
```
describe purchase_events;
```
```
   Column   |  Type   | Comment 
------------+---------+---------
 accept     | varchar |         
 host       | varchar |         
 user-agent | varchar |         
 event_type | varchar |         
 timestamp  | varchar |         
(5 rows)
```

```
describe purchases;
```
```
   Column   |  Type   | Comment 
------------+---------+---------
 accept     | varchar |         
 host       | varchar |         
 user-agent | varchar |         
 event_type | varchar |         
 timestamp  | varchar |         
(5 rows)
```
Query purchases table
```sql
select count(*) from purchases;
```
```
 _col0 
-------
    20 
(1 row)
```
```sql
select * from purchases;
```
```
 accept |       host        |   user-agent    |   event_type   |        timestamp        
--------+-------------------+-----------------+----------------+-------------------------
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.214 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.228 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.232 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.235 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.239 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.246 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.252 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.261 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.266 
 */*    | user1.comcast.com | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:43:02.279 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:12.977 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:12.98  
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:12.985 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:12.988 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:12.995 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:13.002 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:13.01  
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:13.016 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:13.023 
 */*    | user2.att.com     | ApacheBench/2.3 | purchase_sword | 2019-12-06 04:44:13.034 
(20 rows)
```
There are 20 rows in 'purchases' table.



```sql
select * from joins where host = 'user2.att.com';
```
```
 accept |     host      |   user-agent    | event_type |        timestamp        
--------+---------------+-----------------+------------+-------------------------
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.155 
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.159 
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.162 
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.171 
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.174 
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.177 
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.18  
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.182 
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.185 
 */*    | user2.att.com | ApacheBench/2.3 | join_guild | 2019-12-07 23:11:34.188 
(10 rows)
```
There are 10 rows which have 'user2.att.com' in the column 'host' in 'joins' table.

Write from a stream
```
while true; do docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword; done
```
```
while true; do docker-compose exec mids ab -n 10 -H "Host: user1.comcast.com" http://localhost:5000/purchase_a_sword; done
```

```
docker-compose exec cloudera hadoop fs -ls /tmp/sword_purchases
```
```
Found 3 items
drwxr-xr-x   - root supergroup          0 2019-12-06 05:27 /tmp/sword_purchases/_spark_metadata
-rw-r--r--   1 root supergroup        688 2019-12-06 05:27 /tmp/sword_purchases/part-00000-ad0bdd9f-fb91-4f41-a067-27c4b736e019-c000.snappy.parquet
-rw-r--r--   1 root supergroup       2459 2019-12-06 05:27 /tmp/sword_purchases/part-00000-ccedace1-506b-4560-bb92-3bf6b7b23868-c000.snappy.parquet
```



down
```
docker-compose down
```
```
docker-compose ps
```
```
docker ps -a
```


