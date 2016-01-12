# Setting up Bushido development environment

You will need the following software to be installed:

- [Docker](http://www.docker.com)
- [Git](https://git-scm.com/)
- [Maven](https://maven.apache.org/)
- [NodeJS](https://nodejs.org/en/)
- [Android Studio](http://developer.android.com/sdk/index.html)

# RabbitMQ

RabbitMQ is a message broker used on Bushido to wire up server, web and mobile components.

Build RabbitMQ image from this repository [rabbitmq](https://github.com/bushidowallet/rabbitmq). Checkout the code, and once in the rabbitmq folder:
```
docker build -t bushido-rabbitmq .
```
Run the RabbitMQ container off this image.
```
docker run -t -i -e RABBITMQ_DEFAULT_USER=bushido -e RABBITMQ_DEFAULT_PASS=bushido -p 15674:15674 -p 5672:5672 bushido-rabbitmq
```
Note we open up 2 ports: **15674** for rabbitmq_web_stomp plugin and **5672** for AMQP.

# MongoDB

MongoDB is a document-oriented, noSQL database used on Bushido as a data and metadata persistance layer.

Run the MongoDB container.
```
docker run --name bushido-mongo -d mongo
```
In a separate console window, launch a MongoDB client to test if its working fine and find out the IP address of MongoDB container.
```
docker run -it --link bushido-mongo:mongo --rm mongo sh -c "exec mongo $MONGO_PORT_27017_TCP_ADDR:$MONGO_PORT_27017_TCP_PORT/test"
```
Make a note of the IP address assigned, typically its **172.17.0.x** 

# Bushido Java Service

Bushido Java Service provides J2EE applications required to run Bushido - including the APIs.

Clone [bushido-java-service](https://github.com/bushidowallet/bushido-java-service). If you are on Windows, clone to a folder in your user's Documents (eg. C:\Users\JohnDoe\Documents\bushido). It is required by Docker on this OS.
```
git clone https://github.com/bushidowallet/bushido-java-service.git
```
Once you have the code, locate *application.dev.properties* file in */bushido-java-service/bushido-wallet-service/src/main/resources* and change the following entries:
```
app.rabbit.host=192.168.99.100 //your Docker Machine's IP
app.mongo.host=172.17.0.2 //MongoDB container's IP
```
You are ready to build it with Maven.
```
mvn install
```
Once its built, run the docker container off the Tomcat 8 - JRE 7 image defined in this repo [tomcat](https://github.com/bushidowallet/tomcat/tree/bushido/8-jre7)
```
docker run -it --rm -e JAVA_OPTS="-Dspring.profiles.active=dev" -e CATALINA_OPTS="-Xms512M" -p 8080:8080 -v //c/Users/JohnDoe/Documents/bushido/bushido-java-service/bushido-wallet-service/target/bushido-wallet-service-1.0.3:/usr/local/tomcat/webapps/walletapi tomcat:8.0
```
# Bushido Web Application 

Bushido Web Application is a HTML/JS front-end with wallet, registration, widgets and other modules.

Clone [bushido-web-app](https://github.com/bushidowallet/bushido-web-app) to bushido folder, follow the instructions from the code repo's readme file.

# Nginx

Nginx is a HTTP and Reverse Proxy server used on Bushido to serve Bushido Web Application, and reverse proxy API calls and socket connections upstream to Tomcat and RabbitMQ.

Clone [docker-nginx](https://github.com/bushidowallet/docker-nginx) to bushido folder.
```
git clone https://github.com/bushidowallet/docker-nginx.git
```
This repo comes with /conf folder, where you can find Nginx configuaration files and some self-signed SSL certificates. Review the [bushido.conf](https://github.com/bushidowallet/docker-nginx/blob/bushido/conf/bushido.conf) file as you will need to change a few IP addresses there. For dev purpose you can use included self-signed SSL certificate, however you can also generate self-signed certificates if you prefer to.

If you are a Linux user, install [OpenSSL](https://www.openssl.org/) with apt-get. If you prefer Windows, get [Win32 OpenSSL v1.0.2e](http://slproweb.com/download/Win32OpenSSL-1_0_2e.exe). 

Generate Root CA certificate. Use common name: Bushido SSL
```
openssl genrsa -out ca.key 4096
req -new -x509 -days 1826 -key ca.key -out ca.crt
```
Generate Bushido wildcard unsigned certificate. Use common name: *.bushidowallet.com
```
genrsa -out star_bushidowallet_com.key 4096
req -new -key star_bushidowallet_com.key -out bushido.csr
```
Sign wildcard certificate with Root CA's key
```
x509 -req -days 730 -in bushido.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out bushido.crt
```
Copy bushido.crt content to bundle.crt and save it.

Build the image and run the container:
```
docker build -t bushido-nginx .
docker run --name bushido-nginx -v //c/Users/JohnDoe/Documents/bushido/docker-nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v //c/Users/JohnDoe/Documents/bushido/docker-nginx/conf/bushido.conf:/etc/nginx/conf.d/bushido.conf:ro -v //c/Users/JohnDoe/Documents/bushido/bushido-web-app/dist:/usr/share/nginx/html:ro -v //c/Users/JohnDoe/Documents/bushido/docker-nginx/conf/cert/bundle.crt:/etc/nginx/bundle.crt:ro -v //c/Users/JohnDoe/Documents/bushido/docker-nginx/conf/cert/star_bushidowallet_com.key:/etc/nginx/star_bushidowallet_com.key:ro -d -p 80:80 -p 443:443 bushido-nginx
```
Configure your hosts file to contain the following entries
```
192.168.99.100 app.bushidowallet.com
192.168.99.100 websockets.bushidowallet.com
192.168.99.100 api.bushidowallet.com
```
IP address is your Docker Machine's IP address.
