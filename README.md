# Installing Bushido dev environment with [Docker](http://www.docker.com)

# RabbitMQ

RabbitMQ is a message broker used on Bushido to wire up server, web and mobile components.

Build RabbitMQ image from this repository [rabbitmq](https://github.com/bushidowallet/rabbitmq). Checkout the code, and once in the rabbitmq folder:
```
docker build -t bushido-rabbitmq .
```
Run the container off this image.
```
docker run -t -i -e RABBITMQ_DEFAULT_USER=bushido -e RABBITMQ_DEFAULT_PASS=bushido
-p 15674:15674 -p 5672:5672 bushido-rabbitmq
```
Note we open up 2 ports: 15674 for rabbitmq_web_stomp plugin and 5672 for AMQP.
