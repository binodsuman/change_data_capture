 # Change Data Capture (CDC) with Debezium & MySQL: Real-Time Data Streaming! #

Change Data Capture (CDC) is a set of software design patterns used to determine and track data that has changed so that action can be taken using the changed
data. In essence, it's about capturing and responding to data modifications in real-time. 

Debezium is an open-source distributed platform for change data capture.
It simplifies the process of capturing and streaming database changes. Here's how it helps

# Demo Setup #
1. Install MySql 8.0.11 
https://downloads.mysql.com/archives/community/<br>

Create my.cnf file in<br>
/usr/local/mysql-8.0.11-macos10.13-x86_64/my.cnf<br>
>sudo touch my.cnf<br>
>sudo vi my.cnf <br>
contents:<br>

[mysqld]<br>
server-id = 223344<br>
log_bin = mysql-bin<br>
binlog_format = row<br>
binlog_row_image = FULL<br>
gtid_mode = ON<br>
enforce-gtid-consistency = TRUE<br>
require_secure_transport=OFF<br>

** Create new user in mysql:<br>
mysql> CREATE USER 'debezium'@'%' IDENTIFIED WITH mysql_native_password BY 'MyStrongPass123!';<br>
mysql> GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'debezium'@'%';<br>
mysql> FLUSH PRIVILEGES;<br>
<br>

2. Install Debezium<br>
Create folder demo and keep two files:<br>
  a. debezium-mysql-source-connector.yml<br>
  b. debezium-connector-config.json<br>

<b> Docker Commands </b>
>docker compose -f debezium-mysql-source-connector.json up -d<br>
<br>
Check in Docekr dashboard app. <br>

curl -H "Accept:application/json" localhost:8083/connectors/
Return will be []

curl -s http://localhost:8083/connectors/mysql-connector/status

curl -X POST -H "Accept:application/json" -H "Content-Type:application/json" \
    --data @debezium-connector-config.json http://localhost:8083/connectors

curl -s http://localhost:8083/connectors/mysql-connector/status

If any change in mysql IP address.
curl -X DELETE http://localhost:8083/connectors/mysql-connector

curl -X POST -H "Accept:application/json" -H "Content-Type:application/json" \
    --data @debezium-connector-config.json http://localhost:8083/connectors

If any change in Mysql then restart <br>

sudo /usr/local/mysql/support-files/mysql.server restart  <br>

docker-compose down<br>
docker-compose up -d<br>

Now go to Kafka container and check topic has been created.
> docker exec -it kafka_demo bash<br>
root@96c7a0bc81ec:/# cd /usr/bin/

<br>
root@96c7a0bc81ec:/usr/bin# ./kafka-topics --bootstrap-server localhost:9092 --list<br>
__confluent.support.metrics<br>
__consumer_offsets<br>
my_connect_configs<br>
my_connect_offsets<br>
my_connect_statuses<br>
mysql_school_server<br>
mysql_school_server.schema-changes.school<br>
mysql_school_server.school.students<br>
root@96c7a0bc81ec:/usr/bin#<br>

mysql> insert into students value (2, 'Ishan Suman');<br>

Output on Kafka topic:
{"schema":{"type":"struct","fields":[{"type":"struct","fields":[{"type":"int32","optional":true,"field":"id"},{"type":"string","optional":true,"field":"name"}],"optional":true,"name":"mysql_school_server.school.students.Value","field":"before"},{"type":"struct","fields":[{"type":"int32","optional":true,"field":"id"},{"type":"string","optional":true,"field":"name"}],"optional":true,"name":"mysql_school_server.school.students.Value","field":"after"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"version"},{"type":"string","optional":false,"field":"connector"},{"type":"string","optional":false,"field":"name"},{"type":"int64","optional":false,"field":"ts_ms"},{"type":"string","optional":true,"name":"io.debezium.data.Enum","version":1,"parameters":{"allowed":"true,last,false,incremental"},"default":"false","field":"snapshot"},{"type":"string","optional":false,"field":"db"},{"type":"string","optional":true,"field":"sequence"},{"type":"string","optional":true,"field":"table"},{"type":"int64","optional":false,"field":"server_id"},{"type":"string","optional":true,"field":"gtid"},{"type":"string","optional":false,"field":"file"},{"type":"int64","optional":false,"field":"pos"},{"type":"int32","optional":false,"field":"row"},{"type":"int64","optional":true,"field":"thread"},{"type":"string","optional":true,"field":"query"}],"optional":false,"name":"io.debezium.connector.mysql.Source","field":"source"},{"type":"string","optional":false,"field":"op"},{"type":"int64","optional":true,"field":"ts_ms"},{"type":"struct","fields":[{"type":"string","optional":false,"field":"id"},{"type":"int64","optional":false,"field":"total_order"},{"type":"int64","optional":false,"field":"data_collection_order"}],"optional":true,"field":"transaction"}],"optional":false,"name":"mysql_school_server.school.students.Envelope"},"payload":{"before":null,"after":{"id":1,"name":"Binod Suman"},"source":{"version":"1.9.7.Final","connector":"mysql","name":"mysql_school_server","ts_ms":1742456381000,"snapshot":"last","db":"school","sequence":null,"table":"students","server_id":0,"gtid":null,"file":"mysql-bin.000002","pos":195,"row":0,"thread":null,"query":null},"op":"r","ts_ms":1742456381438,"transaction":null}}
Use json fomratter<br>

To check change in Schema <br>
>docker exec -it kafka_demo bash<br>
root@96c7a0bc81ec:/# cd /usr/bin<br>
./kafka-console-consumer --bootstrap-server localhost:9092 --topic mysql_school_server.schema-changes.school  --from-beginning<br>
<br>
watch this above topic, <br>
mysql> create table demo (id integer, name varchar(100));<br>







  
