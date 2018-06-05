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



#### Device Monitor

For the purpose of this tutorial, a series of dummy IoT devices have been created, which will be attached to the context broker.
The state of each device can be seen on the UltraLight device monitor web-page found at: `http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.Historic-Context/img/device-monitor.png)

#### Store 002

Store002 can be found at: `http://localhost:3000/app/store/urn:ngsi-ld:Store:002`

![Store](https://fiware.github.io/tutorials.Historic-Context/img/store2.png)


# Architecture

This application builds on the components and dummy IoT devices created in 
[previous tutorials](https://github.com/Fiware/tutorials.IoT-Agent/). It will make use of three FIWARE components - 
the [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/), the
[IoT Agent for UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/) and introduce the
[Cygnus Generic Enabler](http://fiware-cygnus.readthedocs.io/en/latest/) for persisting context data to a database.
Two databases are now involved - both the Orion Context Broker and the IoT Agent rely on [MongoDB](https://www.mongodb.com/) technology to keep persistence of the information they hold, and we will be persisting our historical context data into a **PostgreSQL** database.


Therefore the overall architecture will consist of the following elements:

* Three **FIWARE Generic Enablers**:
  * The FIWARE [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
  * The FIWARE [IoT Agent for UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/) which will receive southbound requests
    using [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) and convert them to 
    [UltraLight 2.0](http://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual) commands for 
    the devices
  * FIWARE [Cygnus](http://fiware-cygnus.readthedocs.io/en/latest/) which will subscribe to context changes and persist them into a
    **POSTGRES** database
* Two **Databases**:
  * The underlying [MongoDB](https://www.mongodb.com/) database :
    + Used by the **Orion Context Broker** to hold context data information such as data entities, subscriptions and registrations
    + Used by the **IoT Agent** to hold device information such as device URLs and Keys
  * An additional **POSTGRES** database :
    + Used as a data sink to hold historical context data.
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

![](https://fiware.github.io/tutorials.Historic-Context/img/architecture.png)



## PostgreSQL Server Configuration

```yaml
  postgres:
    image: postgres:latest
    hostname: historic-db
    container_name: historic-db
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

The `postgres` container is listening on a single port: 

* Port `5432` is the default port for a PostgreSQL server. It has been exposed so you can also run the `pgAdmin4` tool to display database data if you wish

The `postgres` container is driven by environment variables as shown:

| Key             |Value.    |Description                    |
|-----------------|----------|-------------------------------|
|POSTGRES_PASSWORD|`password`| Password for the PostgreSQL database user|
|POSTGRES_USER    |`postgres`| Username for the PostgreSQL database user|
|POSTGRES_DB      |`postgres`| The name of the PostgreSQL database      | 



## Cygnus  Configuration

```yaml
  cygnus:
    image: fiware/cygnus-ngsi:latest
    hostname: cygnus
    container_name: cygnus
    networks:
        - default
    expose:
        - "5080"
    ports:
        - "5080:5080"
    environment:
        - "CYGNUS_POSTGRESQL_HOST=historic-db"
        - "CYGNUS_POSTGRESQL_PORT=5432"
        - "CYGNUS_POSTGRESQL_USER=postgres" 
        - "CYGNUS_POSTGRESQL_PASS=password" 
        - "CYGNUS_LOG_LEVEL=DEBUG"
        - "CYGNUS_SERVICE_PORT=5050"
        - "CYGNUS_API_PORT=5080"
        - "CYGNUS_POSTGRESQL_ENABLE_CACHE=true"
```

The `cygnus` container is listening on two ports: 

* Port `5050` is exposed ....
* Port `5080` is exposed purely for tutorial access - so that cUrl or Postman can make provisioning commands
  without being part of the same network.


The `cygnus` container is driven by environment variables as shown:

| Key                           |Value         |Description|
|-------------------------------|--------------|-----------|
|CYGNUS_POSTGRESQL_HOST         |`historic-db` | Hostname of the PostgreSQL server used to persist historical context data |
|CYGNUS_POSTGRESQL_PORT         |`5432`        | Port that the PostgreSQL server uses to listen to commands |
|CYGNUS_POSTGRESQL_USER         |`postgres`    | Username for the PostgreSQL database user | 
|CYGNUS_POSTGRESQL_PASS         |`password`    | Password for the PostgreSQL database user |
|CYGNUS_LOG_LEVEL               |`DEBUG`       | The logging level for Cygnus |
|CYGNUS_SERVICE_PORT            |`5050`        | Notification Port that Cygnus listens when subcribing to context data changes|
|CYGNUS_API_PORT                |`5080`        | Port that Cygnus listens on for operational reasons |
|CYGNUS_POSTGRESQL_ENABLE_CACHE |`true`        | Switch to enable caching within the PostgreSQL configuration |






# Persisting Context Data

To follow the tutorial correctly please ensure you have the device monitor page available in your browser and click on the page to enable audio before you enter any cUrl commands. The device monitor displays the current state of an array of dummy devices using Ultralight 2.0 syntax

#### Device Monitor
The device monitor can be found at: `http://localhost:3000/device/monitor`

#### Stores

The stores can be found at:

* Store 1 -  `http://localhost:3000/app/store/urn:ngsi-ld:Store:001`
* Store 2 -  `http://localhost:3000/app/store/urn:ngsi-ld:Store:002`
* ...etc




```console
docker run -it --rm  --network fiware_default jbergknoff/postgresql-client postgresql://postgres:password@historic-db:5432/postgres
```

Once running a docker container within the network, it is possible to 


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
