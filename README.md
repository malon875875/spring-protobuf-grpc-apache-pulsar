# Sample Spring Boot app with GRPC Protobuf/ Apache Pulsar

It consists of two apps `main-app` and `greeting-service`. 
`main-app` takes person name via endpoint `/greet/{fName}/{lName}` which will be sent to the pulsar topic `person-message`. 
It will be received by another app `greeting-service` which will create a greeting messsage and puts to `greeting-message` topic which will be received by `main-app`.

`protobuf-model` is a common module that contains the Greeting.proto and Person.proto files and `protobuf-maven-plugin` to generate Java classes

Use the script on pulsar-docker.sh to start pulsar instance.

## Notes
* Explanation of this code -- [Protobuf Apache Pulsar with Spring Boot](http://blog.gtiwari333.com/2020/09/protobuf-apache-pulsar-spring-boot.html)
  * there are two applications, main-app and greeting-service
  * main-app
    * /greet/{fName}/{lName} produces message "Person" to the topic person-message
  * greeting-service
    * listens to topic person-message with a MessageListener
    * for each message "Person" received in topic person-message
      * converts message "Person" to message "Greeting"
      * produces message "Greeting" to topic greeting-message
* Original source code -- https://github.com/malon875875/spring-protobuf-grpc-apache-pulsar 
* [Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data](https://developers.google.com/protocol-buffers/)
* [What is @PostConstruct](https://docs.oracle.com/javaee/5/api/javax/annotation/PostConstruct.html)
  * _The PostConstruct annotation is used on a method that needs to be executed after dependency injection is done to perform any initialization._

* After the Injection is done, the Pulsar Consumer uses a "Message Listener" for listening to messages in the topic
  * [_Pulsar Listener Config: We can register the message listener using @PostConstruct_](http://blog.gtiwari333.com/2020/09/protobuf-apache-pulsar-spring-boot.html)
  * **it is using old apache pulsar 2.6.1, do I still need "Message Listener" in apache pulsar latest version or can I use receiveAsync ?**
    * https://stackoverflow.com/questions/59380557/why-use-messagelisteners-in-apache-pulsar-and-not-simply-consumer-receive
    * http://mail-archives.apache.org/mod_mbox/pulsar-users/201906.mbox/%3C5d04b627.1c69fb81.8f6a7.d717@mx.google.com%3E
    * **both .messageListener and .receiveAsync are used in latest apache pulsar**
    * Interface Consumer<T>.[receiveAsync](https://pulsar.incubator.apache.org/api/client/2.8.0-SNAPSHOT/index.html?org/apache/pulsar/client/api/Consumer.html) -- pull message from topic in non blocking way
    * Interface ConsumerBuilder<T>.[messageListener](https://pulsar.incubator.apache.org/api/client/2.8.0-SNAPSHOT/index.html?org/apache/pulsar/client/api/ConsumerBuilder.html) -- When a MessageListener is set, application will receive messages through it. Calls to Consumer.receive() will not be allowed. 
      * https://pulsar.incubator.apache.org/api/client/2.8.0-SNAPSHOT/index.html?org/apache/pulsar/client/api/MessageListener.html

## Run
* mvn clean compile
* run apache pulsar in docker pulsar-docker.sh
  * docker run -it   -p 6650:6650   -p 8080:8080   apachepulsar/pulsar:2.2.0   bin/pulsar standalone

## Notes
* Register the Message Listener ConsumerBuilder<T>.[messageListener](https://pulsar.incubator.apache.org/api/client/2.8.0-SNAPSHOT/index.html?org/apache/pulsar/client/api/ConsumerBuilder.html) using @PostConstruct  
* Make sure though to use in @PostConstruct
