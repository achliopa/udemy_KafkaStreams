# Udemy Course: Apache Kafka Series - Kafka Streams for Data Processing

* [Course Link](https://www.udemy.com/course/kafka-streams/)
* [Course Repo]()
* [Lecturers Website](https://courses.datacumulus.com/)

## Section 1: Kafka Streams - First Look

### Lecture 1. What is Kafka Streams?

* Easy data processing and transformation library within Kafka
* used for:
    * data transformations
    * data enrichment
    * fraud detection
    * monitoring and alerting
* what they are:
    * standard Java Apps
    * no need for separate cluster
    * highly scalable, elastic and fault tolerant
    * offer exactly once capabilities (no loss of data)
    * one record at a timeprocessing (no batching)

### Lecture 2. Course Objective / Prerequisites / Target Students

* Write and package Kafka Streams Apps
    * Word Count: get familiar with concepts
    * Favorite Colour: practice more Kafka Streams transsformations API (+Skala Version)
    * BankBalance App to use aggregations and the power of Exactly Once semantics
    * StreamEnrich to join and enrich a KStream using a GlobalKTable

### Lecture 4. Running your first Kafka Streams Application: WordCount

* Steps
    * Download Kafka Binaries
    * Start Zookeeper and Kafka
    * Create input and output topics using 'kafka-topics'
    * Publish data to the input topic
    * Run the WordCount example
    * Stream the output topic using 'kafka-console-consumer'
* we have it on our machine. 
* we start zookeeper `zookeeper-server-start.sh config/zookeeper.properties`
* we start kafka broker `kafka-server-start.sh config/server.properties`
* we create an input topic `kafka-topics.sh --create -bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic streams-plaintext-input`
* we create an output topic `kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic streams-wordcount-output`
* we list topics `kafka-topics.sh --bootstrap-server localhost:9092 --list`
* we publish data to the input topic `kafka-console-producer.sh --broker-list localhost:9092 --topic streams-plaintext-input`
* we enter some data
```
kafka streams udemy
kafka data processing
kafka streams course
```

* we start a consumer to verify topic  functionality `kafka-console-consumer.sh --bootsp-server localhost:9092 --topic streams-plaintext-input --from-beginning`

* we start a consumer that uses a formatter
    * 
```
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic streams-wordcount-output \
    --from-beginning \
    --formatter kafka.tools.DefaultMessageFormatter \
    --property print.key=true \
    --property print.value=true \
    --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```

* we start the demo stream application `kafka-run-class.sh org.apache.kafka.streams.examples.wordcount.WordCountDemo`

* output appears in the terminal with coutns for each word in the topic
* we stop kafka streams app and consumer
* in topics list we see that additional topics are added by kafka

### Lecture 5. Kafka Streams vs other stream processing libraries (Spark Streaming, NiFI, Flink

* Spark Streaming does microbatching (near RT)
* Kafka Streaming is pure data streaming (RT)
* Spark et all need a cluster
* Kafka scales easily . just add java processes
* Exactly once semantics
* All code based

## Section 3: End to End Kafka Streams Application - Word Count

### Lecture 7. Section Objective

* understand the fundamentals of Kafka Streams (Topology theory)
* IDE Setup (Dev Env)
* Understand basic Kafka Streams configs
* Understand Java 8 Lambdas
* First application write-up
* Demo to Run in DevEnv
* Lear about Kafka Streams App internals
* How to package code in a jar to run elsewhere
* how to scale the kafka streams apps

### Lecture 8. Kafka Streams Core Concepts

* a Stream is a sequence of immutable data records, that fully ordered can be replayed and is fault tolerant (think of a Kafka Topic as a parallel). in a processor topology it is the edge
* a stream processor is a node in the procesor topology (graph). it transforms incoming streams record by record and may create a new stream out of it
* a topology is a graph of processors chained together by streams
* A source processor is a special processor that takes ints data directly from a Kafka Topic. it has no predecessors in a topology and does not transform the data
* A sink processor is a processor that does not have children, its sends the data directly to a Kafka topic
* In this course we will leverage the High Level DSL
    * it is the simplest
    * has all the operations we need to perform most transformation tasks
    * it is very descriptive
    * it is extensive
* There is also a Low Level Processor API
    * imperative API
    * used for very complex logic, rarely needed

### Lecture 9. Environment and IDE Setup: Java 8, Maven, IntelliJ IDEA

* We need
    * java 8 JDK (have it) check with `java -version`
    * Maven 3 (???)
    * Jetbrains IntelliJ IDEA CE (Have it)
* choose oracle jdk openjdk should work also
* maven: go to [downloads](https://maven.apache.org/download.cgi#) get the bin extract it add it to PATH (we have it)
* same way for IntelliJ (we have it)

### Lecture 10. Starter Project Setup

* We will create a starter project with the required maven depndencies
    * kafka Streams client
    * Logging Libs (log4j)
    * maven plugins
* run intellij => file => new => project => maven (java 1.8) =>
    * groupid: con.github.achliopa.kafka.streams
    * artifactid: word-counts
    * projectname: word-counts
* enable auto import
* in pom.xml (maven file) we will add the <dependencies>
* we cp latest [kafka-streams maven dep](https://mvnrepository.com/artifact/org.apache.kafka/kafka-streams) in pom.xml
* we cp latest stable [slf4j log maven dep](https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12) in pom.xml
* we do same for [slf4j api](https://mvnrepository.com/artifact/org.slf4j/slf4j-api)
* in /resources we create a new file 'log4j.properties
* we cp content into from another ready project
```
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%p %m (%c:%L) %n
```
* its log config for stdout
* we will also add plugins to our project. in pom.xml we add a <build><plugins> tag after <dependencies>
* in there we cp the maven compiler plugin from a ready project in courseRepo
```
            <!--force java 8-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
```

* this forces java 8 to be used for compile

### Lecture 11. Kafka Streams Application Properties

* a stream application, when communicating to kafka is leveraging the Consumer and Poducer API
* Therefore all the configurations we learned in the basic course are still applicable
* `bootstrap.servers` need to connect to kafka (usually at port 9092)
* `auto.offset.reset.config` set to earliest to consume the topic from start
* `application.id` specific to Streams application, will be used for
    * consumer `group.id` = application.id (most important one to remember)
    * default `client.id` prefix
    * prefix to internal changelog topics
* `default.[key|value].serde` (for serialization and deserialization of data)
* src=>main=>java New=>java class 'com.github.achliopa.kafka.streams.WordCountsApp'
* add a main() with a println and run to test all is ok
* we follow producer / consumer api std setting the propertis using Properties object and use StreamsConfig
```
Properties config = new Properties();
        config.put(StreamsConfig.APPLICATION_ID_CONFIG, "word-counts-app");
        config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        config.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        config.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG,Serdes.String().getClass());
```

### Lecture 12. Java 8 Lambda Functions - quick overview

* Assune we have a stream `KStream<String,Long> stream=...;`
* In Java & what we used to write when passing a function
```
stream.filter(new Predicate<String,Long>() {
    @Override
    public boolean test(String key,Long value) {
        return value > 0;
    }
});
```
* to pass a callback called in every element of a stream to do filtering we instantiate an object and override its test() method
* in Java 8 we can now use an anonymous lambda function (python with JS arrow function style ... sweet!!)
```
stream.filter((key,value) -> value > 0);
```

* types are inferred at compile time

### Lecture 13. Word Count Application Topology

* We ll write the app topology using the High Level DSL for our app
* Remember data in Kafka streams is <Key,Value>
* we use ta `StreamsBuilder` for the app `StreamsBuilder builder = new StreamsBuilder();`
1. `Stream` from Kafka `<null, "Kafka Kafka Streams">`
2. `MapValues` lowercase `<null, "kafka kafka streams">`
3. `FlatMapValues` split by space `<null,"kafka">,<null,"kafka">,<null,"streams">`
4. `SelectKey` to apply a key `<"kafka","kafka">,<"kafka","kafka">,<"kafka","streams">`
5. `GroupByKey` before aggregation `(<"kafka","kafka">,<"kafka","kafka">),(<"kafka","streams">)`
6. `Count` occurences in each group `<"kafka",2>,<"streams",1>`
7. `To` in order to write the results back to Kafka `data point is written to Kafka`
```
KStream<String,String>  textLines = builder.stream("word-count-input");
KTable<String, Long> wordCounts = textLines
.mapValues(textLine -> textLine.toLowerCase())
.flatMapValues(lowercasedTextLine -> Arrays.asList(lowercasedTextLine.split(" ")))
.selectKey((ignoredKey,word) -> word)
.groupByKey()
.count(Materialized.as("Counts"));
wordCounts.toStream().to("word-count-output", Produced.with(Serdes.String(), Serdes.Long()));
```

* note that we now changes value to Long as it is reflected in KTable and serdes
* we then create the streams app and start it
```
KafkaStreams streams = new KafkaStreams(builder.build(),config);
        streams.start();
```

### Lecture 14. Printing the Kafka Streams Topology

* printing the topology at the start of the application is helpful while developing (and even in production) as it helps understand the application flow directly from the first lines of the logs
* the topology represents all the streams and processors of our streams application
```
KafkaStreams streams = new KafkaStreams(builder.build(),config);
streams.start();
System.out.println(streams.toString());
```

### Lecture 15. Kafka Streams Graceful Shutdown

* close gracefully and avoid coruptions
* adding a shutdown hook is key to allow for a graceful shutdown of the Kafka Streams application, which will help the speed of restart
* this should be in every Kafka Streams application we create
```
// Add shutdown hook to stop the kafka Streams threads.
// We can optionally provide a timeout to 'close'
Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
```

### Lecture 16. Running Application from IntelliJ IDEA