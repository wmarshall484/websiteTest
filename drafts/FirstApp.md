# Introduction
The Java Application API allows you to write programs for Streaming data exclusively in Java—no SPL! You can run the programs as Java programs, you can run them as standalone Streams applications, or you can run them as distributed Streams applications. If you need help getting your environment set up, visit please visit [the Java Application API set-up guide](www.google.com).

The JAA enables the developer to:
* Define the structure of a streaming application using only Java
* Pass java objects as tuples on a streaming
* Define how data is processed in a modular, scalable manner.

If you're viewing this page, it's likely that you haven't worked with the Java Application API before. 
* Walk through an introductory example with temperature readings.
* Talk about the basic source -> transform -> sink processing paradigm.

# First Application
In its entirety, a simple Java Application API app looks like the following:
``` Java 
import java.util.Arrays;
import java.util.Random;

import com.ibm.streamsx.topology.TStream;
import com.ibm.streamsx.topology.Topology;
import com.ibm.streamsx.topology.context.StreamsContextFactory;
import com.ibm.streamsx.topology.function.Supplier;

public class TemperatureTest {
    public static void main(String[] args){
        
        Topology topology = new Topology("temperatureSensor");
        Random random = new Random();
        
        @SuppressWarnings("unchecked")
        TStream<Double> readings = topology.endlessSource(new Supplier<Double>(){
            @Override
            public Double get() {
                return random.nextGaussian();
            }
            
        });
    
        readings.print();
        StreamsContextFactory.getEmbedded().submit(topology);
    }
}
```

### Create the Topology object
The topology object contains all the information about the information flow and processing of your topology. For the purposes of this example, it enables developers to define their data sources.
Different kinds of data sources:
* User supplied (i.e, topology.strings("a", "b", "c");
* Kafka (link to kafka example)
* HDFS (link to hdfs example that Kris Hildrum created)

In our case, we simply invoke the topology.source() method which takes a Java Supplier object to return the correct data item upon each invocation. 

### Printing to output
Printing to standard output is as easy as invoking the TStream.sink() method which call .toString() on each of the Java Objects in the stream. 

### Submitting the application
* EMBEDDED - Run the application in embedded mode. This is a simulated Java environment for running your Streams application.
* DISTRIBUTED - Run the application in distributed mode. When an application is submitted with this context, a Streams Application Bundle (.sab file) is produced. The .sab file contains all of the application's logic and third-party dependencies, and is submitted to a Streams instance automatically.
* STANDALONE - Run the application in stand alone mode. When running in this mode, the application also produces a Streams Applicaition Bundle (.sab file), but rather than submitting it to a cluster, the bundle is instead executable. The bundle will run within its own process, and may be terminated with ctrl-C interrupts.
