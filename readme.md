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

* running a kafka streams app can be done direclty from intelliJ
* its a good way for iterative dev 
* we will create topics run a console consumer and producer and see out streamer in action
* we create the input topic `kafka-topics.sh --create -bootstrap-server localhost:9092 --replication-factor 1 --partitions 2 --topic word-count-input`
* we create the output topic `kafka-topics.sh --create -bootstrap-server localhost:9092 --replication-factor 1 --partitions 2 --topic word-count-output`
* we fire up a consumer to listen to output `kafka-console-consumer.sh --bootstrap-server localhost:9092     --topic word-count-output     --from-beginning     --formatter kafka.tools.DefaultMessageFormatter     --property print.key=true     --property print.value=true     --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer     --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer`
* start a console producer `kafka-console-producer.sh --broker-list localhost:9092 --topic word-count-input`
* run the application and IT WORKS but its not instant
* we see that streams use offsets to resume. these are consumer offsets as streams implement the consumer api
* also app has app
* app rebalnces also all by itself
* streams app runs 4 tasks as we see
* its not instant as streams does internal batching say for 30sec

### Lecture 17. Debugging Application from IntelliJ IDEA

* we use intelliJ in debug mode
* ok we now how to debug with IDE
* we can execute commands after hitting breakpoint

### Lecture 18. Internal Topics for our Kafka Streams Application

 * running a Kafka Streams application may eventually create internal intermediary topics
 * Two types
    * repartition topics: in case we start transforming the key of our stream, a repartitioning will happen at some processor
    * changelog topics: in case we perform aggregations, kafka Streams will save compacted data in these topics
* internal topics:
    * are managed by kafka streams
    * are used by kafka streams to save/restore state and re-partition data
    * are prefixed by application.id parameter
    * shoul NEVER be deleted, altered or published to.
* for our app they are
```
word-counts-app-Counts-changelog
word-counts-app-Counts-repartition
```

* we can dump them with consumer to view but there is no reason

### Lecture 19. Packaging the application as Fat Jar & Running the Fat Jar

* in order to deploy the application to other machines, we often need to compile it as a .jar (Java ARchive)
* Default compilation in java only includes the code we write in the .jar file without the dependencies (libs)
* maven ahs a plugin to allow us to package all our cde + the dependencies into one .jar simply called a "fat jar"
* Demo:
    * package application as a fat jar
    * run application from the fat jar
* to do so we need to add one more plugin in pom.xml file in <plugins> 
```
<!--package as one fat jar-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.1.0</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>com.github.achliopa.kafka.streams.WordCountsApp</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>assemble-all</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

* to do so we need to pass the path to our mainclass in the manifest as seen above
* one way to package is to click on Maven => word-counts=>lifecycle=>clean to clean the target folder
* then click on 'package'
* in the log we see the path where our fat jar was created
* we can use command line in our project root folder running `mvn clean package`
* the `word-counts-1.0-SNAPSHOT-jar-with-dependencies.jar` is the name of the file 
* we can run our app from command line in the project root dir as `java -jar target/word-counts-1.0-SNAPSHOT-jar-with-dependencies.jar`
* it runs!!!!!!
* ctrl+C to stop

### Lecture 20. Scaling our Application

* our input topic has 2 partitions, therefore we can launch up to 2 instances of our application in parallel without any changes in the code!!
* This is because a Kafka Streams application relies on KafkaConsumer and we saw in the Kafka basics course that we can add conumers to a consumer group by just running the same code
* we dont need a cluster we ust need threads
* we ll run 2 instances of the app and publish. we should see performance increase
* it works!!!

## Section 4: KStreams and KTables Simple Operations (Stateless)

### Lecture 22. Section Objectives

* Goals:
* Learn about theroretical aspects
    * KStream,KTable,Duality
    * Refresher on log compaction
* Learn about simple transformations and operations
    * Map/MapValues
    * Filter/FilterNot
    * FlatMap/FlatMapValues
    * Branch
    * toStream()
    * Reading from & Writing to /through kafka
* Documentation can be found [here](https://docs.confluent.io/current/streams/developer-guide/index.html#transform-a-stream)

### Lecture 23. KStream & KTables

* KStreams
    * is all Inserts
    * like a log
    * its infinite
    * unbounded data streams
* like a buffer as topic gets key,value pairs they are appended to the KStream obj
* KTables
    * all Upserts of non-null values
    * its all Deletes on null values
    * Similar to a table in a DB
    * Parallel with log compacted topics
* its like the log compaction on topics. if an updated value comes for the smae key it updated it does not simply append to the list
* if a null value comes for a key the entry is deleted
* KStream 
    * is for reading from a topic that is not compacted
    * if new data is partial information / transactional
* KTable 
    * reading from a topic that is log compacted (aggregations)
    * more if we want a structure that is like a "database table" where every update is self sufficient (e.g total bank balance)

### Lecture 24. Stateless vs Stateful Operation

* Stateless means that the result of a transformation only depends on the data-point we process. e.g "multiply by 2" is stateless because it does not need memory of the past for the calculation
* Stateful means that the result of a transformation also depends on external info (the state). A count operation is stateful because the app needs to know what happened since the start

### Lecture 25. MapValues / Map

* MapValues
    * takes one record and produces one record
    * is only affecting values
    * == does not change keys
    * == does not trigger repartition
    * used in KStreams and KTables
* Map
    * affects both keys and values
    * triggers a re-partitions
    * for Kstreams only
```
KStream<byte[],String> uppercased = stream.mapValues(value -> value.toUpperCase()); # (alice,cow) => (alice,COW)
```

### Lecture 26. Filter / FilterNot

* takes one record and produces zero or one record
* Filter
    * does not change keys/values
    * == does not trigger a repartition
    * for KStreams and KTables
* FilterNot
    * inverse of Filter
```
KStream<String,Long> onlyPositives = strea,filter((key,value) -> value > 0); # it deletes filtered records
```

### Lecture 27. FlatMapValues / FlatMap

* takes one record and produces zero,one or more records
* FlatMapValues
    * does not change keys
    * == does not trigger a repartition
    * for KStrems only
* FlatMap
    * change keys
    * == triggers a repartition
    * For KStreams only
```
words = sentences.flatMapValues(value -> Arrays.asList(value.split("\\s+")));
```

* example (alice,"alice is nice") =>[(alice, alice),(alice,is),(alice,nice)]

### Lecture 28. Branch

* Branch (split) a KSTream based on one or more predicates
* Predicates are evaluated in order, if no matches, records are dropped
* You get multiple KStreams as a result
```
KStream<String,Long>[] branches = stream.branch(
    (key,value) -> value > 100, /* first predicate */
    (key,value) -> value > 10,  /* second predicate */
    (key,value) -> value > 0,   /* third predicate */
);
```

### Lecture 29. SelectKey

* assigns a new key to the record (from old key and value)
* == marks the data for repartitioning
* best practice to isolate that transformation to know exactly where the partitioning happens
```
// use the first leteer of the key as the new key
rekeyed = stream.selectKey((key,value) -> key.substring(0,1)) # (alice,paris) => (a,paris)
```

### Lecture 30. Reading from Kafka

* We can read a topic as a KStream, KTable or a GlobalKTable
* read as KStream (serdes types are needed if we dont use defaults)
```
KStream<String,Long> wordCounts = builder.stream(
    Serdes.String(), /* key serde */
    Serdes.Long(),   /* value serde */
    "word-counts-input-topic" /* input topic */);
```

* read as KTable 
```
KTable<String,Long> wordCounts = builder.table(
    Serdes.String(), /* key serde */
    Serdes.Long(),   /* value serde */
    "word-counts-input-logic" /* input topic */);
```

* read as GlobalKTable 
```
GlobalKTable<String,Long> wordCounts = builder.globalTable(
    Serdes.String(), /* key serde */
    Serdes.Long(),   /* value serde */
    "word-counts-input-logic" /* input topic */);
```

### Lecture 31. Writing to Kafka

* We can write any KStream or KTable back to kafka
* If we write a KTable back to Kafka, think about creating a log compacted topic
* To: terminal operation - write the records to a topic
```
stream.to("my-output-topic");
table.to("my-output-topic");
```
* through: write to a topic and get back a stream/table from the topic
```
KStream<String,Long> newStream = stream.through("user-clicks-topic");
KTable<String,Long> newTable = table.through("my-table-output-topic");
```

### Lecture 32. Streams Marked for Re-Partition

* as soon as an operation can possible change the key, the stream will be marked for repartition
    * Map,FlatMap,SelectKey
* So only use these APIs if we need to change the key, otherwise use their counterparts
    * MapValues
    * FlatMapValues
* Repartitioning is done seamlessly behind the scenes but will incur a performance cose (read and write to Kafka)

### Lecture 33. Refresher on Log Compaction

* Lecture is same as Kafka Basics Course (Lecture 108-109)

### Lecture 34. KStream and KTables Duality

* Stream as a Table: A stream can be considered a changelog of a table, where each data record in the stream captures a state change of the table
* Table as a Stream: A table can be considered a snapshot, at a point in time, of the latest value for each key in a stream (s streams data records are key value pairs)
* Table => Stream : Stream captures only updates in  Table records
* Stream => Table: table records are streams records (uprdates) + previous records

### Lecture 35. Transforming a KTable to a KStream

* It is sometimes helpful to transform a KTbagle to a KStream in order to keep a changelog of all the changes to the Ktable (see last lecture on KStream/Ktable duality)
* This can be easily achieved in one line of code
```
KTable<byte[], String> table = ...;
// also a varian of .toStream() exists to select a new key for the resulting stream
KStream<byte[], String> stream = table.toStream();
```

### Lecture 36. Transforming a KStream to a KTable

* Two ways:
    * Chain a groupByKey() and an aggregation step (count,aggregate,reduce)
```
KTable<String,Long> table = usersAndColors.groupByKey().count();
```

    * Write back to kafka and read as KTable
```
// write to Kafka
stream.to("intermediary-topic");
// read from kafka as a table
KTable<String,String> table = builder.table("intermediary-topic");
```

## Section 5: Practice Exercise - FavouriteColour

### Lecture 38. FavouriteColour - Practice Exercise Description & Guidance

* we will take a comma delimited topic of userid, colour
    * filter out bad data
    * keep only color of "green","red","blue"
* get the running count of the favorite colours overall and output this to a topic
* A user's favorite color can change
* Steps
    * write topology
    * start finding the right transformations to apply (see previous section)
    * creating input and output topics (and intermediary topics if we think of any)
    * feed the sample data as a producer

### Lecture 39. Stuck? Here are some Hints!

* The input data does not have keys but represents updates
    * we should read it as a KStream and extract the key
    * we should write the result to Kafka (as a log compacted topic)
 * The results can now be read as a KTbale so that updates are correctly applied
 * We can now perform an affregation on the KTable (groupBY then count)
 * And write the results back to Kafka
 * Topology
    * Read one topic from Kafka (KStream)
    * Filter bad values
    * SelectKey that will be the userId
    * MapValues to extract the colour (as lowercase)
    * Filter to remove the colours
    * Write to Kafka as Intermediary topic
    * Read from kafka as KTable
    * GroupNy colors
    * Count to count colors occurencies (KTable)
    * Write to Kafka as final topic

### Lecture 40. Java Solution

* pom.xml is same as property file
* input topic is 'favourite-colour-input'
* intermediate topic is 'user-keys-and-colours'
* output topic is 'favourite-colour-output'

### Lecture 41. Running the application

* in favorite-colour.sh we find the commands to run our demo
```
kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic favourite-colour-input
kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic user-keys-and-colours --config cleanup.policy=compact
kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic favourite-colour-output --config cleanup.policy=compact
```

* we create the consumer
```
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic favourite-colour-output \
    --from-beginning \
    --formatter kafka.tools.DefaultMessageFormatter \
    --property print.key=true \
    --property print.value=true \
    --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
```

* fire up producer `kafka-console-producer.sh --broker-list localhost:9092 --topic favourite-colour-input`

* in project `mvn clean package`

* run stream app `java -jar target/favorite-color-java-1.0-SNAPSHOT-jar-with-dependencies.jar`

### Lecture 42. Scala Solution

* Scala is a prog language that also runs on the JVM
* It is the basic lang in Spark
* just a walkthrouh in scala project
* 'build.sbt' is the pom.xml equivalnt with dependencies and version
* kafka jhas not scala libs 
* porperties file is same
* code is very similar
* create the topics,consumer,producer
* to launch hust run in in IDE

## Section 6: KStreams and KTables Advanced Operations (Stateful)

### Lecture 43. Section Objective

* KTable Stateful Opers
    * groupBy
* KGroupedStream / KGroupedTable Opers
    * Count
    * Reduce
    * Aggregate
* KStreams adv ops
    * Peek
    * Transform / TransformValues
* [Documentation](https://docs.confluent.io/current/streams/developer-guide/dsl-api.html#stateful-transformations)

### Lecture 44. KTable groupBy

* GroupBy allows us to perform more aggregations within a KTable (add,decrease)
* we have used it in the previous section during our favorite colour example
* it trigers a repartition because the key changes 
```
// group the table by a new key and key type
KGroupTable<String,Integer> groupedTable = table.groupBy(
    (key,value) -> KeyValue.pair(value, value.length()),
    Serdes.String(), /* key (note: type was modified) */
    Serdes.Integer() /* value (note: type was modified));
```

### Lecture 45. KGroupedStream / KGroupedTable Count

* As a reminder, KGroupedStream are obtained after a `groupBy/groupByKey()` call on a Kstream
* `Count` counts the number of record by grouped key
* If used on KGroupedStream:
    * Null keys or values are ignored
* If used on KGroupedTable:
    * Null keys are ignored
    * Null values are treated as deletes

### Lecture 46. KGroupedStream / KGroupedTable Aggregate

* KGroupedStream Aggregate
* You need an initializer (of any type) an adder  a Serde and a State Store name (name of the aggregation)
* Example: Count total string length by key
```
// aggregating a KGroupedStream (note how the value type changes from String to Long)
KTable<byte[], Long> aggregtedStream = groupedStream.aggregate(
    () -> 0L, /* initializer */
    (aggKey, newValue, aggValue) -> aggValue + newValue.length(), /* adder */
    Serdes.Long(), /* serdes  for aggregated value */
    "aggregated-stream-store" /* state store name */
);
```

* KGroupedTable Aggregate
* you need an initializer (of any type), an adder, a subtractor, a Serde and a State Store name (name of the aggregation)
* Example: Count total string length by key
```
// aggregating a KGroupedStream (note how the value type changes from String to Long)
KTable<byte[], Long> aggregtedSTable = groupedTable.aggregate(
    () -> 0L, /* initializer */
    (aggKey, newValue, aggValue) -> aggValue + newValue.length(), /* adder */
    (aggKey, oldValue, aggValue) -> aggValue - oldValue.length() /* subtractor */
    Serdes.Long(), /* serdes  for aggregated value */
    "aggregated-stream-store" /* state store name */
);
```

### Lecture 47. KGroupedStream / KGroupedTable Reduce

* Similar to Aggregate but the result type has to be the same as an input 
* (int,int) => (int) (e.g a*b)
* (String,String) => (String) (e.g concat(a,b))
```
// Reducing a KGroupStreams
KTable<String,Long> aggregatedStream = groupedStream.reduce(
    (aggValue, newValue) -> aggValue + newValue, /* adder */
    "reduced-stream-store" /* state store name */);
// reducing a KGroupedTable
KTable<String,Long> aggregatedTable = groupedTable.reduce(
    (aggValue, newValue) -> aggValue + newValue, /* adder */
    (aggValue, oldValue) -> aggValue - oldValue, /* subtr */
    "reduced-table-store" /* state store name */);
```

### Lecture 48. KStream peek

* Peek allows us to apply a side-effect operation to a KStream and get the same KStream as a result
* A sideeffect could be:
    * printing the stream to the console
    * statistics collection
* Warning: It could be executed multiple times as it is a side effect (in case of failure) at least once operation
```
KStream<byte[], String> stream = ...;

// Java 8+ example, using lambba expressions
KStream<byte[],String> unmodifiedStream = stream.peek(
    (key,value) -> System.out.println("key"+key+", value="+value));
```

### Lecture 49. KStream Transform / TransformValues

* Applies a Transformer to each record
* Transformer leverages the low level Processor API
* The processor API is really advanced. we can see it [here](https://kafka.apache.org/0110/documentation/streams/developer-guide#streams_processor) and [here](https://docs.confluent.io/current/streams/developer-guide/index.html#streams-developer-guide-processor-api)
* TransformValues do not trigger re-partition like Transform

### Lecture 50. What if I want to write to an external System?

* Although it is theoretically doable to do it using Kafka Streams lib, it is NOT recomended
* use a Connect API from the output topic

### Lecture 51. Summary Diagram

* all the opers in a concise diagram

![diaram](https://docs.confluent.io/current/_images/streams-stateful_operations.png)

## Section 7: Exactly Once Semantics - Theory

### Lecture 52. What's Exactly Once?

* Exactly Once is the Holy Grail of Streaming
* Check out [Article1](https://medium.com/@jaykreps/exactly-once-support-in-apache-kafka-55e1fdd0a35f) [Article2](https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/) [Article3](https://techcrunch.com/2017/06/30/confluent-achieves-holy-grail-of-exactly-once-delivery-on-kafka-messaging-service/?guccounter=1&guce_referrer=aHR0cHM6Ly93d3cudWRlbXkuY29tL2NvdXJzZS9rYWZrYS1zdHJlYW1zL2xlYXJuL2xlY3R1cmUvNzY1ODg1OA&guce_referrer_sig=AQAAALuPzVYiXXiIfSt8b1bAwinS_qw2ezC8GVu_adjWVV-WcX0MaXryHuo292mzZo6Jmm4zreCmQsjsFiirjxjJOxiQ4yisQLPHY-gGw3yV438S78EioS1o_aYRNNBpJJBnTRrMpDwhR_vvXN51ZC-yXqvL8f1gUJC_KR3pbvtlVSiM)
* What is Exactly Once??
    * its the ability to guarantee that data processing on each message will happen only once, and that pushing the message back to Kafka will also happen effectively only once (Kafka will de-dup)
    * it is guaranteed only when both input and output systems is Kafka 
    * only works if Streams and Brokers are at v >=0.11
* Kafka Streams as Kaka Consumer (at least once)
    * we receive twice the same message if the Kafka broker reboots or our kafka consumer restarts
    * thi is because offsets are committed once in a while but the data mayhave been processed already
* As a Kafka Producer
    * we send twoce the same message to kafka if we dont receive an ack back from kafka (because of the rerty logic)
    * not receiveing the ack does not mean kafka hasn't received our message. it means that thenetwork just failed instead of kafka

### Lecture 53. Exactly Once in Kafka 0.11

* without to much detail
* the producers a re now idempotent (if the same message is sent twoce or more due t oretries kafka will make sure to only keep one copy of it)
* we can write multiple messages to different Kafka topics as part of one transaction (either all are wirtten or none). This is a new advanced API
* To acieve this they had to change the logic and internal message format in kafka 0,11 therefore this is the minium version offering it
* we only jhave to use the libs offering it. like Kafka Streams
* [Talk](https://www.youtube.com/watch?v=Wo9jlaz8h0k&t=1s)

### Lecture 54. What's the problem with at least once anyway?

* Cases when its not acceptable to have at least once (or when we need exactly once)
    * getting the exact count by key for a stream
    * summing up bank transactions to compute a bank balance
    * any financial conputation
    * any operation that is non idempotent
* cases when its acceptable to have at least once semantics
    * operations on time windows
    * approximate operations (counting IP hits)
    * idempotent ops (min,max)

### Lecture 55. How to do exactly once in Kafka Streams

* one more line of code
```
Properties props = new Properties();
...
props.put(StreamsConfig.PROCESSING_GUARANTEE_CONFIG, StreamsConfig.EXACTLY_ONCE);
...
KafkaStreams stream = new KafkaStreams(builder.build(),props);
```

* Currently Kafka Streams is the only library that has implemented this fear. its possible that Spark, Flink will follow
* Whats the trade off?
    * results are published in transactions. (latency)
    * we can fine tune this with `commit.interval.ms`

### Lecture Section 8: Exactly Once Semantics - Practice Exercise - BankBalance

### Lecture 56. BankBalance - Exercise Overview

* we have to create a kafka producer that outputs 100  msg p. sec to a topic. Each message is random in money (a positive value) and outputs evenly transactions for 6 customers. the data shoul look like `{ "Name":"John", "amount": 123, "time": "2017-07-19T05:23:24"}`
* we have to create kafka streams application that takes these transactions and will compoute the total money in their balance (the balance starts with 0$) and the latest time an update was received. as we expect the total money is not idempotent (sum) but latest time is (max)
* Run the producer and streams app and see an exactly once pipeline

### Lecture 57. Kafka Producer Guidance

* writing a kafka producer we did in beginners course. we need a key for the custemer in our message
* we can use the lib 'JacksonDatabind' to generate json or interpolate strings

### Lecture 58. Kafka Producer Solution

* `BankTransactionsProducer`
* we cp the whole project and open it with IDEA
* in pom.xml we have also kafka-client dep
* properties file is added
* in PRoducer class
* we want full acknowledge
    * LINGER_MS_CONFIG is not good for production
* we NEED to set ENABLE_INDEMPOTTENCE_TRUE to make sure our producer is idempotent
* newRandomTransaction build the transaction
* input topic is "bank-transaction"
* output topic is "bank-balance-exactly-once"

### Lecture 59. Kafka Streams Guidance & Hints

* same as before, practice writing down the topology before implementing the program
* look at previous lectures to understand which additional config we need tand which advanced transformation is needed
* to write a JSON serde we can leverage the ones in the package org.apache.kafka.connect.json or use a String Serde and then parse to JSON
* Topology
    * Read one topic from Kafka (KStream)
    * GroupByKey, because our topic already has the right key no repartition happens
    * Aggregate, to compute the "bank balance"
    * To in order to write to Kafka

### Lecture 60. Kafka Streams Solution

* we use exactly once property also a JSON serdes
* JsonNode clas is used to hold JSON object

### Lecture 61. Running the BankBalance Application

* 'bank-balance.sh' holds the commands we need to run
* we create the 2 topics
```
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic bank-transactions
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic bank-balance-exactly-once --config cleanup.policy=compact
```
* we fire aup a consumer 
```
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic bank-balance-exactly-once \
    --from-beginning \
    --formatter kafka.tools.DefaultMessageFormatter \
    --property print.key=true \
    --property print.value=true \
    --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

* we start producer and streams app

## Section 9: Joins - KStream to GlobalKTable example

### Lecture 63. What are joins in Kafka Streams?

* Joining means taking a KStream and/or KTable and creating a new KStream or KTabe from it
* There are 4 kind of Joins (SQLlike) and the most common one will be analyzed in a further  sction, including behaviour and usage
* Joins status in v0.11
    * KStream-to-KStream [Windowed] - InnerJoin: Supported, LeftJoin: Supported, OuterJoin: Supported
    * KTable-to-KTable [Non-Windowed] - InnerJoin: Supported, LeftJoin: Supported, OuterJoin: Supported
    * KStream-to-KTable [Non-Windowed] - InnerJoin: Supported, LeftJoin: Supported, OuterJoin: NotSupported
    * KStream-to-GlobalKTable [Not-Windowed] - InnerJoin: Supported, LeftJoin: Supported, OuterJoin: NotSupported

### Lecture 64. Join Constraints and GlobalKTables

* These 3 Joins:
    * KStream/KStream
    * KTable/KTable
    * KStream/KTable
* can only happen when the data is co-partitioned. otherwise the join wont be possible and Kafka Streams will fail with a Runtime Error
* This mens that the same number of partitions is there on the strea and/or the table
* To co-partition data, if the number of partitions is different, write back the topics through Kafka before the join (This has a network cose)
* If our KTable data is reasonably small, and can fit on each of our Kafka Streams application, we can read it is a GlobalKTable
* With GlobalKTables, we can join any stream to our table even if data doesnt have the same number of partition
* thats because the table data lives on every Streams application instance
* the downside is HDD space req, its ok for reasonably sized dataset but can grow a lot
* we will demo a join using the GlobalKTable technique
* [Joins Explained](https://www.confluent.io/blog/crossing-streams-joins-apache-kafka/)

### Lecture 65. The different types of joins: Inner Join, Left Join, Outer Join

* Inner Join
    * Join the data only if it has matches streams of data
    * example: show me the students with lockers
* Left Join 
    * join all the data from left whether or not it has a match on the right
    * example "show me the students with and without a locker"
* Outer Join
    * only available for KStream/Kstream joins
    * its a left join combined with a right join

### Lecture 66. Creating a join with UserEnrich Kafka Streams App

* Join User Purchases (KStream) to UserData (GlobalKTable)
* Write a producer to explain the different scenarios
* observe the output
* Topology
    * Read one topic from kafka (KStream)
    * Read the other topic from Kafka (GlobalKTable)
    * Inner Join
    * Write to Kafka the result of the inner join
    * Left Join
    * Write to Kafka the result of the the Left Join
* in the app code we read from one topic as a globalKtable
* the inner join is done with a lambda that takes the key,value pair of the stream an returns the key based on which the join jhappens
* mapper takes a value of the stream and and table and joins them
* we output to a topic
* then we do  left join. in that case user info (table) might be null. in mapper we need to handle it
* in producer we block the future with .get()

### Lecture 68. Running the Kafka Streams Join application

* in 'join-example.sh' we see how to run the example 
* we create input and output topics and 2 consumers for both join types
* we choose same num of parttions

## Section 10: Testing your Kafka Streams Application

### Lecture 70. Testing in Kafka Streams

* From kafka 1.1 there is now the possibility to test the Topology object of the Kafka Streams App
* This doe not require to run Kafka in our tests and make it simple and efficient to ensure our streams applications are working the way we wnat theem to
* this requires some efactoring
* When running a Kafka Streams Application:
    * KafkaStreamsApp consumes and produces records to/from topics
* When testing a Kafka Streams App
    * we dont have the kafka brokers and topics. we use 
    * a consumer record geenrator (Factory)
    * a produce record reader + Tests
* To do so:
    * we will refactor the word-count app to extract the topology builder in a separate method
    * we ll then add testing dependencies to our maven pom.xml and create and setup tests
* What we will test
    * Test 1: ensure that word count works while pushing multiple keywords. push "testing Kafka Streams" and ensure the counts are correct, push "testing Kafka again" and ensure the coutns are correct (kafka and testing now have a count of 2)
    * Test 2: ensure that word count does counts by using lowercase words

### Lecture 71. Setup your Kafka Streams project

* we need test utility in kafka and junit dep
```
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams-test-utils</artifactId>
            <version>2.0.1</version>
            <scope>test</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/junit/junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
```

### Lecture 72. Hands-On: Test your WordCount application

* extract builder from app class
* follow example and run tests
