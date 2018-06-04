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
Two databases are now involved - both the Orion Context Broker and the IoT Agent rely on [MongoDB](https://www.mongodb.com/) technology to keep persistence of the information they hold, and we will be persisting our historical context data into a **POSTGRES** database.


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




# Persisting Context Data

To follow the tutorial correctly please ensure you have the device monitor page available in your browser and click on the page to enable audio before you enter any cUrl commands. The device monitor displays the current state of an array of dummy devices using Ultralight 2.0 syntax

#### Device Monitor
The device monitor can be found at: `http://localhost:3000/device/monitor`

#### Stores

The stores can be found at:

* Store 1 -  `http://localhost:3000/app/store/urn:ngsi-ld:Store:001`
* Store 2 -  `http://localhost:3000/app/store/urn:ngsi-ld:Store:002`
* ...etc







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
