[Samantha]
In general, the goal of the document is to produce a getting started guide that helps the user underestand the ins and outs of the Java Application API.  After they finished reading the document, they should underestand it enough to write an useful Streams application using the API.  The document needs to be a bit deeper than just writing the first application.  We need to use the application to illustrate the various concepts and explains some of the details.  These details may be included once you start writing the document, but I want to make sure that it is the goal.  
[Samantha]

# Java Application API: First Application

The Java Application API enables developers to write IBM Streams Applications entirely in Java, requiring no knowledge of SPL.
* Java code can be used to modify and generate data
* Java objects can be passed as tuples of a stream
* 
[Samantha]
I think the Java code can be used in the following:
* to define the Streams application topology and how data is transformed
* to generate data for test
* to ingest data from external source
* to send data out to external data sinks
[Samantha]

Open Source [link to project](http://ibmstreams.github.io/streamsx.topology/)

The typical pattern for a JAA Streaming Application is source -> transform -> sink. Our application will
* Generate a dataset modeling readings from a temperature sensor.
* Filter it such that it's between a min and a max
* Create a historgram of the sensor's values
* Print the histogram to standard output.
	
## Building the Application

Since the JAA is pure Java, use the benefits of the Java ecosystem (Eclipse, code completion, etc...).

Defining a source is easy!
* Create a topology object, which keeps track of the information about the state of your application.
* Invoke the topology.endlessSource() method to repeatedly grab a random number from a gaussian distribution (the temperature).

[Samantha]
We shoud take this opportunity to introduce some basic concepts.
    *  What is a topology?
    *  What is a source?
    *  What are the different kinds of sources and how are they different from each other?  When to use what?
[Samantha]

Sources, transformations and sinks all use the Java Functional interface to allow developers to supply their own code. Developers may benefit from using 	
Java 8 lambdas, which are a more concise way of creating Java Functions.

Using Java 8 lambdas, defining a filter is only one line of code:
* stream.filter( temperature -> temperature > -3.0 && temperature < 3.0)
* 
[Samantha]
   * Other than filter, what can we do from a Stream?  How does the user discover / learn what they can do with a Stream?
[Samantha]

Next, creating a histogram of the filtered temperatures demonstrates how we can make a stateful operator. In this case, the operation is termed 'stateful'
because the histogram of temperature readings persists in between tuples being sent on the stream. Here's how to do it

* Invoke filtered_temps.transform()
* Supply a Java Function of the following form:
	
``` 
		new Function(){
			List<Integer> histogram = new List<>(10);
			List<String> apply(String temp){
				Integer index = (temp+(range/2))/(1/density);
				histogram.get(index) += 1;
				return histogram;
			}
```

[Samantha]
   * What does "Transform" mean?  Why is it that at one place, we do a filter, and then at another place we do a transform?
   * The main question is... when do people use transform?
[Samantha]

Finally, we can print the histogram to output as follows:
* tempReadings.print()
	
For now, submit your application as a standalone application:
* StreamsContextFactory.getStreamsContext("EMBEDDED").submit(topology).get();
* 
[Samantha]
   * What are the different submission modes?  How are they different from each other and when to use what?
   * What is really happening under the cover when you run and submit an application?
[Samantha]

Embedded runs the entire app in a single JVM. For a further explanation of the various submission contexts, refer to [link](google.com)
When we run this as a Java Application, we get
```
[0, 1, 4, 2, 7, 10, 12, 10, 4, 2, 1, 0]
[0, 1, 4, 2, 7, 10, 12, 10, 5, 2, 1, 0]
[0, 1, 4, 2, 7, 11, 12, 10, 5, 2, 1, 0]
[0, 1, 4, 2, 8, 11, 12, 10, 5, 2, 1, 0]
```
Note how the histogram is updated each time. For a more in-depth example of how to parallelize your datastream to remove bottlenecks, refer to the tutorial at [link](google.com).
Data sources can be read from more IoT friendly sources! For an example of reading from Kafka, using the build-in kafka support, refer to [kafka sample](google.com).

# Whole Application
(contents of entire application here)

[Samantha]
After this, I would like to see more of the following:

* troubleshooting - what happens if the applciation does not run?  how should they go about debugging it?
* how do people replace the source to ingest from Kafka?
* How do people replace the print and send data to some external source?  maybe HDFS?  HBase?
* how do people ingest from an external source that we have not added support for?
* import / export 
* how do they integrate with existing spl applications?

[Samantha]
