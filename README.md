# bushido-doc

Installing Bushido dev environment with Docker.

1. Build and run Bushido RabbitMQ container.
Build [RabbitMQ](https://github.com/bushidowallet/rabbitmq) image: *docker build -t bushido-rabbitmq .*
Run the container: *docker run -t -i -e RABBITMQ_DEFAULT_USER=bushido -e RABBITMQ_DEFAULT_PASS=bushido -p 15674:15674 -p 5672:5672 bushido-rabbitmq*
