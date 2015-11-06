# Java Application API: First Application

The Java Application API enables developers to write IBM Streams Applications entirely in Java, requiring no knowledge of SPL.
	* Java code can be used to modify and generate data
	* Java objects can be passed as tuples of a strem

Open Source (link to project)[http://ibmstreams.github.io/streamsx.topology/]

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

Sources, transformations and sinks all use the Java Functional interface to allow developers to supply their own code. Developers may benefit from using 	
Java 8 lambdas, which are a more concise way of creating Java Functions.

Using Java 8 lambdas, defining a filter is only one line of code:
	* stream.filter( temperature -> temperature > -3.0 && temperature < 3.0)

Next, creating a histogram of the filtered temperatures demonstrates how we can make a stateful operator. In this case, the operation is termed 'stateful'
because the histogram of temperature readings persists in between tuples being sent on the stream. Here's how to do it:

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
Finally, we can print the histogram to output as follows:
	* tempReadings.print()
	
For now, submit your application as a standalone application:
	* StreamsContextFactory.getStreamsContext("EMBEDDED").submit(topology).get();

Embedded runs the entire app in a single JVM. For a further explanation of the various submission contexts, refer to [link](google.com)
When we run this as a Java Application, we get:
[0, 1, 4, 2, 7, 10, 12, 10, 4, 2, 1, 0]
[0, 1, 4, 2, 7, 10, 12, 10, 5, 2, 1, 0]
[0, 1, 4, 2, 7, 11, 12, 10, 5, 2, 1, 0]
[0, 1, 4, 2, 8, 11, 12, 10, 5, 2, 1, 0]

Note how the histogram is updated each time. For a more in-depth example of how to parallelize your datastream to remove bottlenecks, refer to the tutorial at [link](google.com).
Data sources can be read from more IoT friendly sources! For an example of reading from Kafka, using the build-in kafka support, refer to [kafka sample](google.com).
