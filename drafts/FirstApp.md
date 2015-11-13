# The Java Application API
The Java Application API allows you to write programs for Streaming data exclusively in Java—no SPL! You can run the programs as Java programs, you can run them as standalone Streams applications, or you can run them as distributed Streams applications. If you need help getting your environment set up, visit please visit [the Java Application API set-up guide](Environment_Setup).

The primary goals of the Java Application API are to enable the developer to:
* Define the structure of a streaming application using only Java
* Pass java objects as tuples on a streaming
* Define how data is processed in a modular, scalable, and stateful manner.

Each of these points is covered in further detail the tutorials on this site, and below where we create a sample application that processes sample temperature readings from a device. If you're viewing this page, it's likely that you haven't worked with the Java Application API before, or possibly streaming applications in general. In this document, we cover a sample application and discuss at a high level the general principles behind streaming application development. 

Lastly, the Java Application API is fully compatible with IBM Streams version 4.0.0 and greater. The API is open source, and is available for download from our [streamsx.topology project on Github](https://github.com/IBMStreams/streamsx.topology)

# Data Streaming Principles & The JAA
Streaming applications are usually solutions that meet realtime data processing needs. Whereas frameworks like hadoop deals with batch jobs that eventually terminate after being submitted, datastreaming application are designed to run forever. For example, consider a company whose product scans temperature sensors across the world to determine weather patterns and trends. Since there is *always* a temperature, there is a perpetual need to process data. 

The application must be allowed run for an intederminate amount of time, and it also must be allowed to scale flexibly. Say, for example, that the number of temperature sensers doubles, correspondingly doubling the speed at which the application must process data. The Java Application API supports the ability to easily parallelize your data pipeline, and gives the developer the ability to specify which pieces of the application run on which resources across a certain number of specified processes.

The latter features will be covered in a [subsequent tuturial](UDP_Windowing). For now, let's take the simple example of reading data from a temperature sensor, and printing the output to the screen. 

## First Application: Topology Object
When writing an application with the Java Application API, the very first thing to do is to create the Topology object:
``` Java
Topology topology = new Topology("temperatureSensor");
```
The topology object contains information about the structure of your graph (i.e., your application), including how the data is generated and is processed. The Topology object also provides utility methods that allow us to define our data sources, in this case the temperature sensor. By invoking topology.endlessSource(), we can pass a Java Function that returns the next data item each time it is called.

## First Application: Defining a Data Source

For simplicity, we will simulate a temperature sensor by reading from a Java Random object as follows.  
``` Java
Random random = new Random();
        
TStream<Double> readings = topology.endlessSource(new Supplier<Double>(){
    @Override
    public Double get() {
        return random.nextGaussian();
    }
});
```
The *endlessSource* method will repeatedly call the Function's overridden get() method, and return a new random temperature reading each time. Although in this case we are obtaining our data by calling ```random.nextGaussian()```, in principle this could be substituted for any live data source such as reading from a Kafka cluster, or MQTT server. This will be shown in subsequent tutorials.

As a side note, the API is compatible with Java 8 so this data source could be written more concisely using a lambda expression:
``` Java
TStream<Double> readings = topology.endlessSource(() -> random.nextGaussian());
```

## First Application: The TStream

The *endlessSource* method produces a TStream, which is arguably the most important Java class in the Java Application API. 

**A TStream represents a potentially infinite flow of tuples in your application**. Since an application may run forever, there is no upper limit to the number of tuples that may flow over a TStream. Tuples flow one at a time over a TStream, and are processed by subsequent data **operations**. While we do not define any data operations in this example (such as filtering or transforming the data on a TStream), data operations are covered in the [common streams operations](CommonStreamOperations) tutorial.

One of the strengths of the Java Application API is that a tuple can be any Java Object, so long as it is serializable. As such, a TStream is parameterized to a Java type as was seen in the preceeding line:
```
TStream<Double> readings = ...;
```
In this case, it is parameterized to a Double.

## First Application: Printing to Output

Now, in true "Hello World" fashion, after obtaining the data we simply print it to standard output. 
``` Java
readings.print();
```
Since each tuple is a Java object, invoking TStream's print() method calls the toString() method on each tuple, and prints the results to output using System.out.println(). Although TStream's print() method is useful in its convenience, there are likely cases where the application will need to output to a file or Kafka. These cases are covered in the [Kafka](Kafka) and [common streams operations](CommonStreamOperations) tutorials.

## First Application: Submitting and Running The Application
After the application has been defined, it can be run by acquiring a *context* from the StreamsContextFactory, and invoking its submit() method while passing the topology object as a parameter:
``` Java 
StreamsContextFactory.getEmbedded().submit(topology);
```
The getEmbedded() method returns an EMBEDDED submission context, which runs the application in a single JVM on a single host. There are three primary submission contexts, each of which runs the application in a different manner.

* EMBEDDED - Runs the application in embedded mode. This is a simulated Java environment for running your Streams application.
* DISTRIBUTED - Runs the application in distributed mode. When an application is submitted with this context, a Streams Application Bundle (.sab file) is produced. The .sab file contains all of the application's logic and third-party dependencies, and is submitted to an IBM Streams instance automatically.
* STANDALONE - Runs the application in standalone mode. When running in this mode, the application also produces a Streams Applicaition Bundle (.sab file), but rather than submitting it to a cluster, the bundle is instead executable. The bundle will run within a single process, and may be terminated with ctrl-C interrupts.

# Entire Application
The application, in its entirety, is as follows:
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
