![FIWARE Banner](https://fiware.github.io/tutorials.Historic-Context/img/fiware.png)

This tutorial is an introduction to [FIWARE Cygnus](http://fiware-cygnus.readthedocs.io/en/latest/) - a generic enabler which is used to persist context data into third-party databases creating a historical view of the context.

The tutorial uses [cUrl](https://ec.haxx.se/) commands throughout, but is also available as [Postman documentation](http://fiware.github.io/tutorials.Historic-Context/)

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/2150531e68299d46f937)


# Contents

...etc

# Data Persistence

> "History will be kind to me for I intend to write it."
>
> — Winston Churchill


Previous tutorials have introduced a set of IoT Sensors (providing measurements of the
state of the real world), and two FIWARE Components - the **Orion Context Broker** and an **IoT Agent**. 
This tutorial will introduct a new data persistance component - FIWARE **Cygnus**.

The system so far has been built up to handle the current context, in other words it holds the data entities
defining the state of the real-world objects at a given moment in time.

From this definition you can see - context is only interested in the **current** state of the system.
It is not the responsibility of any of the existing components to report on the historical state of the system.

In order to do this, we will need to extend the existing architecture to persist changes of state into a database whenever 
the context is updated.

Persisting historical context data is useful for big data analysis - it can be used to discover trends, or data 
can be sampled and aggregated to remove the influence of outlying data measurement. However within each Smart Solution,
the significance of each entity type will differ and entities and attributes may need to be sampled at different rates.

Since the business requirements for using context data differ from application to appliation, there is no one standard use 
case  for historical data persistence - each situation is unique - it is not the case that one size fits all.
Therefore rather than overloading the context broker with the job of historical context data persistence, this role has been
separated out into a separate, highly configurable component - **Cygnus**.

As you would expect, **Cygnus**, as part of an Open Source platform, is technology agnostic regarding the database 
to be used for data persistance. The database you choose to use will depend upon your own business needs. 

However there is a cost to offering this flexibility - each part of the system must be separately configured and
notifications must be set up to only pass the minimal data required as necessary.




#### Device Monitor

For the purpose of this tutorial, a series of dummy IoT devices have been created, which will be attached to the context broker.
The state of each device can be seen on the UltraLight device monitor web-page found at: `http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.Historic-Context/img/device-monitor.png)




# Architecture

This application builds on the components and dummy IoT devices created in 
[previous tutorials](https://github.com/Fiware/tutorials.IoT-Agent/). It will make use of three FIWARE components - 
the [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/), the
[IoT Agent for UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/) and introduce the
[Cygnus Generic Enabler](http://fiware-cygnus.readthedocs.io/en/latest/) for persisting context data to a database.
Additional databases are now involved - both the Orion Context Broker and the IoT Agent rely on [MongoDB](https://www.mongodb.com/) technology to keep persistence of the information they hold, and we will be persisting our historical context data another database - either **MySQL** , **PostgreSQL**  or **Mongo-DB** database.


Therefore the overall architecture will consist of the following elements:

* Three **FIWARE Generic Enablers**:
  * The FIWARE [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
  * The FIWARE [IoT Agent for UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/) which will receive southbound requests
    using [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) and convert them to 
    [UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) commands for 
    the devices
  * FIWARE [Cygnus](http://fiware-cygnus.readthedocs.io/en/latest/) which will subscribe to context changes and persist them into a database (**MySQL** , **PostgreSQL**  or **Mongo-DB**)
* One, two or three of the following **Databases**:
  * The underlying [MongoDB](https://www.mongodb.com/) database :
    + Used by the **Orion Context Broker** to hold context data information such as data entities, subscriptions and registrations
    + Used by the **IoT Agent** to hold device information such as device URLs and Keys
    + Potentially used as a data sink to hold historical context data.
  * An additional **POSTGRES** database :
    + Potentially used as a data sink to hold historical context data.
   * An additional **MySQL** database :
    + Potentially used as a data sink to hold historical context data.
* Three **Context Providers**:
  * The **Stock Management Frontend**  is used to do the following:
    + Display store information and allow users to interact with the dummy IoT devices
    + Show which products can be bought at each store
    + Allow users to "buy" products and reduce the stock count.
  * A webserver acting as set of [dummy IoT devices](https://github.com/Fiware/tutorials.IoT-Sensors) using the [UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) protocol running over HTTP.
  * The **Context Provider NGSI** proxy is not used in this tutorial. It does the following:
    + receive requests using [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
    + makes requests to publicly available data sources using their own APIs in a proprietory format 
    + returns context data back to the Orion Context Broker in [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) format.

Since all interactions between the elements are initiated by HTTP requests, the entities can be containerized and run from exposed ports. 

The specific architecture of each section of the tutorial is discussed below.



# Persisting Context Data into a Mongo DB Database

Persisting historic context data using MongoDB technology is relatively simple to configure since
we are already using a MongoDB instance to hold data related to the Orion Context Broker and the
IoT Agent. The MongoDB instance is listening on the standard `27017` port and the overall architecture
can be seen below:

![](https://fiware.github.io/tutorials.Historic-Context/img/cygnus-mongo.png)

## Mongo DB Server Configuration

```yaml
  mongo-db:
    image: mongo:3.6
    hostname: mongo-db
    container_name: db-mongo
    ports:
        - "27017:27017"
    networks:
        - default
    command: --bind_ip_all --smallfiles
```

## Cygnus Configuration to connect to Mongo DB

```yaml
  cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: fiware-cygnus
    depends_on:
        - mongo-db
    networks:
        - default
    expose:
        - "5080"
    ports:
        - "5050:5050"
        - "5080:5080"
    environment:
        - "CYGNUS_MONGO_HOSTS=mongo-db:27017"
        - "CYGNUS_LOG_LEVEL=DEBUG"
        - "CYGNUS_SERVICE_PORT=5050"
        - "CYGNUS_API_PORT=5080"
```


The `cygnus` container is listening on two ports: 

* The service will be listening on port `5050` for notifications from the Orion context broker
* Port `5080` is exposed purely for tutorial access - so that cUrl or Postman can make provisioning commands
  without being part of the same network.


The `cygnus` container is driven by environment variables as shown:

| Key                           |Value         |Description|
|-------------------------------|--------------|-----------|
|CYGNUS_MONGO_HOSTS         |`mongo-db:27017` |  Comma separated list of Mongo-DB servers which Cygnus will contact to persist historical context data |
|CYGNUS_LOG_LEVEL               |`DEBUG`       | The logging level for Cygnus |
|CYGNUS_SERVICE_PORT            |`5050`        | Notification Port that Cygnus listens when subcribing to context data changes|
|CYGNUS_API_PORT                |`5080`        | Port that Cygnus listens on for operational reasons |


### Heartbeat

### Subscribe


#### Device Monitor
The device monitor can be found at: `http://localhost:3000/device/monitor`


### Reading Data from a Mongo-DB database

To read mongo-db data from the command line, we will need access to the `mongo` tool run an interactive instance
of the `mongo` image as shown to obtain a command line prompt:

```console
docker run -it --network fiware_default  --entrypoint /bin/bash mongo
```

You can then log into to the running `mongo-db` database by using the command line as shown:

```bash
mongo --host mongo-db
```

To read the data within a table, run the select statements as shown:

#### Query:

```
show dbs
use sth_openiot
show collections
db["sth_/_Door:001_Door"].find()
```





# Persisting Context Data into a PostgreSQL Database

To persist historic context data into an alternative database such as **PostgreSQL**, we will
need an additional container which hosts the PostgreSQL server - the default Docker image for this
data can be used. The PostgreSQL instance is listening on the standard `5432` port and the overall architecture
can be seen below:

![](https://fiware.github.io/tutorials.Historic-Context/img/cygnus-postgres.png)

We now have a system with two databases, since the MongoDB container is still required 
to hold data relaed to the Orion Context Broker and the IoT Agent. 


## PostgreSQL Server Configuration

```yaml
  postgres-db:
      image: postgres:latest
      hostname: postgres-db
      container_name: db-postgres
      expose:
        - "5432"
      ports:
        - "5432:5432"
      networks:
        - default
      environment:
        - "POSTGRES_PASSWORD=password"
        - "POSTGRES_USER=postgres"
        - "POSTGRES_DB=postgres"

```

The `postgres-db` container is listening on a single port: 

* Port `5432` is the default port for a PostgreSQL server. It has been exposed so you can also run the `pgAdmin4` tool to display database data if you wish

The `postgres-db` container is driven by environment variables as shown:

| Key             |Value.    |Description                    |
|-----------------|----------|-------------------------------|
|POSTGRES_PASSWORD|`password`| Password for the PostgreSQL database user|
|POSTGRES_USER    |`postgres`| Username for the PostgreSQL database user|
|POSTGRES_DB      |`postgres`| The name of the PostgreSQL database      | 



## Cygnus Configuration to connect to PostgreSQL

```yaml
  cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: fiware-cygnus
    networks:
        - default
    depends_on:
        - postgres-db
    expose:
        - "5080"
    ports:
        - "5050:5050"
        - "5080:5080"
    environment:
        - "CYGNUS_POSTGRESQL_HOST=postgres-db"
        - "CYGNUS_POSTGRESQL_PORT=5432"
        - "CYGNUS_POSTGRESQL_USER=postgres" 
        - "CYGNUS_POSTGRESQL_PASS=password" 
        - "CYGNUS_LOG_LEVEL=DEBUG"
        - "CYGNUS_SERVICE_PORT=5050"
        - "CYGNUS_API_PORT=5080"
        - "CYGNUS_POSTGRESQL_ENABLE_CACHE=true"
```

The `cygnus` container is listening on two ports: 

* The service will be listening on port `5050` for notifications from the Orion context broker
* Port `5080` is exposed purely for tutorial access - so that cUrl or Postman can make provisioning commands
  without being part of the same network.


The `cygnus` container is driven by environment variables as shown:

| Key                           |Value         |Description|
|-------------------------------|--------------|-----------|
|CYGNUS_POSTGRESQL_HOST         |`postgres-db` | Hostname of the PostgreSQL server used to persist historical context data |
|CYGNUS_POSTGRESQL_PORT         |`5432`        | Port that the PostgreSQL server uses to listen to commands |
|CYGNUS_POSTGRESQL_USER         |`postgres`    | Username for the PostgreSQL database user | 
|CYGNUS_POSTGRESQL_PASS         |`password`    | Password for the PostgreSQL database user |
|CYGNUS_LOG_LEVEL               |`DEBUG`       | The logging level for Cygnus |
|CYGNUS_SERVICE_PORT            |`5050`        | Notification Port that Cygnus listens when subcribing to context data changes|
|CYGNUS_API_PORT                |`5080`        | Port that Cygnus listens on for operational reasons |
|CYGNUS_POSTGRESQL_ENABLE_CACHE |`true`        | Switch to enable caching within the PostgreSQL configuration |



### Heartbeat

### Subscribe

#### Device Monitor
The device monitor can be found at: `http://localhost:3000/device/monitor`


### Reading Data from a PostgreSQL database

To read PostgreSQL data from the command line, we will need access to the `postgres` client, to do this, run an
interactive instance of the `postgresql-client` image supplying the connection string as shown to obtain a command 
line prompt:

```console
docker run -it --rm  --network fiware_default jbergknoff/postgresql-client postgresql://postgres:password@postgres-db:5432/postgres
```

Once running a docker container within the network, it is possible to obtain information about the running
database.


#### Query:

```sql
SELECT table_schema,table_name
FROM information_schema.tables
WHERE table_schema ='openiot'
ORDER BY table_schema,table_name;
```

#### Result:

```
 table_schema |    table_name     
--------------+-------------------
 openiot      | door_001_door
 openiot      | door_002_door
 openiot      | door_003_door
 openiot      | door_004_door
 openiot      | lamp_001_lamp
 openiot      | motion_001_motion
 openiot      | motion_002_motion
 openiot      | motion_003_motion
 openiot      | motion_004_motion
(9 rows)
```

The `table_schema` matches the `fiware-service` header supplied with the context data:

To read the data within a table, run a select statement as shown:

#### Query:

```sql
SELECT * FROM openiot.door_001_door limit 10;
```

#### Result:

```
  recvtimets   |         recvtime         | fiwareservicepath | entityid | entitytype |   attrname   |   attrtype    |        attrvalue         |                                    attrmd                                    
---------------+--------------------------+-------------------+----------+------------+--------------+---------------+--------------------------+------
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | TimeInstant  | ISO8601       | 2018-06-05T12:34:06.642Z | []
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | close_info   | commandResult |                          | []
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | close_status | commandStatus | UNKNOWN                  | []
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | lock_info    | commandResult |                          | []
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | lock_status  | commandStatus | UNKNOWN                  | []
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | open_info    | commandResult |  open OK                 | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:33:06.591Z"}]
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | open_status  | commandStatus | OK                       | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:33:06.591Z"}]
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | refStore     | Relationship  | Store:001                | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:34:06.642Z"}]
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | state        | Text          | OPEN                     | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:34:06.642Z"}]
 1528202046707 | 2018-06-05T12:34:06.707Z | /                 | Door:001 | Door       | unlock_info  | commandResult |                          | []
(10 rows)
```

#### Query:

```sql
SELECT * FROM openiot.motion_001_motion limit 10;
```

#### Result:

```
  recvtimets   |         recvtime         | fiwareservicepath |  entityid  | entitytype |  attrname   |   attrtype   |        attrvalue         |                                    attrmd                                    
---------------+--------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+-----
 1528202064795 | 2018-06-05T12:34:24.795Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-05T12:34:24.736Z | []
 1528202064795 | 2018-06-05T12:34:24.795Z | /                 | Motion:001 | Motion     | count       | Integer      | 7                        | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:34:24.736Z"}]
 1528202064795 | 2018-06-05T12:34:24.795Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:34:24.736Z"}]
 1528202082820 | 2018-06-05T12:34:42.820Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-05T12:34:42.782Z | []
 1528202082820 | 2018-06-05T12:34:42.820Z | /                 | Motion:001 | Motion     | count       | Integer      | 9                        | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:34:42.782Z"}]
 1528202082820 | 2018-06-05T12:34:42.820Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:34:42.782Z"}]
 1528202112977 | 2018-06-05T12:35:12.977Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-05T12:35:12.930Z | []
 1528202112977 | 2018-06-05T12:35:12.977Z | /                 | Motion:001 | Motion     | count       | Integer      | 10                       | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:35:12.930Z"}]
 1528202112977 | 2018-06-05T12:35:12.977Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | 
 [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-05T12:35:12.930Z"}]
 1528202130992 | 2018-06-05T12:35:30.992Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-05T12:35:30.955Z | []
```

To leave the Postgres client and leave interactive mode, run the following:

```sql
\q
```
 You will then return to the commmand line.


# Persisting Context Data into a MySQL Database

Similarl to persisting historic context data into **MySQL**, we will
need an additional container which hosts the MySQL server, once again the default Docker image for this
data can be used. The MySQL instance is listening on the standard `3306` port and the overall architecture
can be seen below:

![](https://fiware.github.io/tutorials.Historic-Context/img/cygnus-mysql.png)

We now have a system with two databases, since the MongoDB container is still required 
to hold data relaed to the Orion Context Broker and the IoT Agent. 


## MySQL Server Configuration

```yaml
  mysql-db:
      restart: always
      image: mysql:5.7
      hostname: mysql-db
      container_name: db-mysql
      expose:
        - "3306"
      ports:
        - "3306:3306"
      networks:
        - default
      environment:
        - "MYSQL_ROOT_PASSWORD=123"
        - "MYSQL_ROOT_HOST=%"
```

The `mysql-db` container is listening on a single port: 

* Port `3306` is the default port for a MySQL server. It has been exposed so you can also run other database tools to display data if you wish

The `mysql-db` container is driven by environment variables as shown:

| Key               |Value.    |Description                               |
|-------------------|----------|------------------------------------------|
|MYSQL_ROOT_PASSWORD|`123`.    | specifies a password that is set for the MySQL `root` account.|
|MYSQL_ROOT_HOST    |`postgres`| By default, MySQL creates the `root'@'localhost` account. This account can only be connected to from inside the container. Setting this environment variable allows root connections from other hosts | 


## Cygnus Configuration to connect to MySQL

```yaml
  cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: fiware-cygnus
    networks:
        - default
    depends_on:
        - mysql-db
    expose:
        - "5080"
    ports:
        - "5080:5080"
    environment:
        - "CYGNUS_MYSQL_HOST=mysql-db"
        - "CYGNUS_MYSQL_PORT=3306"
        - "CYGNUS_MYSQL_USER=root" 
        - "CYGNUS_MYSQL_PASS=123" 
        - "CYGNUS_LOG_LEVEL=DEBUG"
        - "CYGNUS_SERVICE_PORT=5050"
        - "CYGNUS_API_PORT=5080"
```



The `cygnus` container is listening on two ports: 

* The service will be listening on port `5050` for notifications from the Orion context broker
* Port `5080` is exposed purely for tutorial access - so that cUrl or Postman can make provisioning commands
  without being part of the same network.


The `cygnus` container is driven by environment variables as shown:

| Key                           |Value         |Description|
|-------------------------------|--------------|-----------|
|CYGNUS_MYSQL_HOST              |`mysql-db`    | Hostname of the MySQL server used to persist historical context data |
|CYGNUS_MYSQL_PORT              |`3306`        | Port that the MySQL server uses to listen to commands |
|CYGNUS_MYSQL_USER              |`root`        | Username for the MySQL database user | 
|CYGNUS_MYSQL_PASS              |`123`         | Password for the MySQL database user |
|CYGNUS_LOG_LEVEL               |`DEBUG`       | The logging level for Cygnus |
|CYGNUS_SERVICE_PORT            |`5050`        | Notification Port that Cygnus listens when subcribing to context data changes|
|CYGNUS_API_PORT                |`5080`        | Port that Cygnus listens on for operational reasons |


### Reading Data from a MySQL database

To read MySQL data from the command line, we will need access to the `mysql` client, to do this, run an
interactive instance of the `mysql` image supplying the connection string as shown to obtain a command 
line prompt:

```console
docker run -it --rm  --network fiware_default mysql mysql -h mysql-db -P 3306  -u root -p123
```

To read the data within a table, run a select statement as shown:

#### Query:

```sql
SELECT * FROM openiot.door_001_door limit 10;
```




# Persisting Context Data into a multiple Databases

It is also possible to configure Cygnus to populate multiple databases simultaneously. We can combine
the architecture from the three previous examples and configure cygnus to listen on multiple ports


![](https://fiware.github.io/tutorials.Historic-Context/img/cygnus-all-three.png)

We now have a system with three databases, since the MongoDB container is still required 
to hold data relaed to the Orion Context Broker and the IoT Agent. 

## Cygnus Configuration for Multiple Agents

```yaml
  cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: fiware-cygnus
    depends_on:
      - mongo-db
      - mysql-db
      - postgres-db
    networks:
      - default
    expose:
      - "5080"
      - "5081"
      - "5084"
    ports:
      - "5050:5050"
      - "5051:5051"
      - "5054:5054"
      - "5080:5080"
      - "5081:5081"
      - "5084:5084"
    environment:
      - "CYGNUS_MULTIAGENT=true"
      - "CYGNUS_POSTGRESQL_HOST=postgres-sb"
      - "CYGNUS_POSTGRESQL_PORT=5432"
      - "CYGNUS_POSTGRESQL_USER=postgres" 
      - "CYGNUS_POSTGRESQL_PASS=password" 
      - "CYGNUS_POSTGRESQL_ENABLE_CACHE=true"
      - "CYGNUS_MYSQL_HOST=mysql-db"
      - "CYGNUS_MYSQL_PORT=3306"
      - "CYGNUS_MYSQL_USER=root" 
      - "CYGNUS_MYSQL_PASS=123" 
      - "CYGNUS_LOG_LEVEL=DEBUG"
```



In multi-agent mode, the `cygnus` container is listening on multiple ports: 

* The service will be listening on ports `5050-5055` for notifications from the Orion context broker
* Ports `5080-5085` are exposed purely for tutorial access - so that cUrl or Postman can make provisioning commands
  without being part of the same network.

The default port mapping can be seen below:

| sink       | port | admin_port |
|-----------:|-----:|-----------:|
| mysql      | 5050 | 5080       |
| mongo      | 5051 | 5081       |
| ckan       | 5052 | 5082       |
| hdfs       | 5053 | 5083       |
| postgresql | 5054 | 5084       |
| cartodb    | 5055 | 5085       |

Since we are not persisting CKAN, HDFS or CartoDB data, there is no need to open those ports.




The `cygnus` container is driven by environment variables as shown:

| Key                           |Value         |Description|
|-------------------------------|--------------|-----------|
|CYGNUS_MULTIAGENT              |`true`        | Whether to persist data into multiple databases. |
|CYGNUS_MONGO_HOSTS             |`mongo-db:27017` |  Comma separated list of Mongo-DB servers which Cygnus will contact to persist historical context data |
|CYGNUS_POSTGRESQL_HOST         |`postgres-db` | Hostname of the PostgreSQL server used to persist historical context data |
|CYGNUS_POSTGRESQL_PORT         |`5432`        | Port that the PostgreSQL server uses to listen to commands |
|CYGNUS_POSTGRESQL_USER         |`postgres`    | Username for the PostgreSQL database user | 
|CYGNUS_POSTGRESQL_PASS         |`password`    | Password for the PostgreSQL database user |
|CYGNUS_MYSQL_HOST              |`mysql-db`    | Hostname of the MySQL server used to persist historical context data |
|CYGNUS_MYSQL_PORT              |`3306`        | Port that the MySQL server uses to listen to commands |
|CYGNUS_MYSQL_USER              |`root`        | Username for the MySQL database user | 
|CYGNUS_MYSQL_PASS              |`123`         | Password for the MySQL database user |
|CYGNUS_LOG_LEVEL               |`DEBUG`       | The logging level for Cygnus |


### Heartbeat

### Subscribe

### Read Data


# Next Steps

Want to learn how to add more complexity to your application by adding advanced features?
You can find out by reading the other tutorials in this series:

&nbsp; 101. [Getting Started](https://github.com/Fiware/tutorials.Getting-Started)<br/>
&nbsp; 102. [Entity Relationships](https://github.com/Fiware/tutorials.Entity-Relationships/)<br/>
&nbsp; 103. [CRUD Operations](https://github.com/Fiware/tutorials.CRUD-Operations/)<br/>
&nbsp; 104. [Context Providers](https://github.com/Fiware/tutorials.Context-Providers/)<br/>
&nbsp; 105. [Altering the Context Programmatically](https://github.com/Fiware/tutorials.Accessing-Context/)<br/> 
&nbsp; 106. [Subscribing to Changes in Context](https://github.com/Fiware/tutorials.Subscriptions/)<br/>

&nbsp; 201. [Introduction to IoT Sensors](https://github.com/Fiware/tutorials.IoT-Sensors/)<br/>
&nbsp; 202. [Provisioning an IoT Agent](https://github.com/Fiware/tutorials.IoT-Agent/)<br/>

&nbsp; 301. [Persisting Context Data](https://github.com/Fiware/tutorials.Historic-Context/)<br/>
