# Installing Bushido development environment with [Docker](http://www.docker.com)

# RabbitMQ

RabbitMQ is a message broker used on Bushido to wire up server, web and mobile components.

Build RabbitMQ image from this repository [rabbitmq](https://github.com/bushidowallet/rabbitmq). Checkout the code, and once in the rabbitmq folder:
```
docker build -t bushido-rabbitmq .
```
Run the RabbitMQ container off this image.
```
docker run -t -i -e RABBITMQ_DEFAULT_USER=bushido -e RABBITMQ_DEFAULT_PASS=bushido
-p 15674:15674 -p 5672:5672 bushido-rabbitmq
```
Note we open up 2 ports: 15674 for rabbitmq_web_stomp plugin and 5672 for AMQP.

# MongoDB

MongoDB is a document-oriented, noSQL database used on Bushido as a data and metadata persistance layer.

Run the MongoDB container.
```
docker run --name bushido-mongo -d mongo
```
In a separate console window, launch a MongoDB client to test if its working fine and find out the IP address of MongoDB container.
```
docker run -it --link bushido-mongo:mongo --rm mongo 
sh -c "exec mongo $MONGO_PORT_27017_TCP_ADDR:$MONGO_PORT_27017_TCP_PORT/test"
```
Make a note of the IP address assigned, typically its 172.17.0.x 

# bushido-java-service

Bushido Java Service provides J2EE applications required to run Bushido - including the APIs.

Clone [bushido-java-service](https://github.com/bushidowallet/bushido-java-service). If you are on Windows, clone to a folder in your user's Documents (eg. C:\Users\JohnDoe\Documents\bushido). It is required by Docker on this OS.
```
git clone https://github.com/bushidowallet/bushido-java-service.git
```

