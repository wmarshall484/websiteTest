# Integrating SPL operators with the Java Application API

Since the applications written with the Java Application API are capable of running on the IBM Streams platform, it's natural that the API would integrate with SPL primitive operators and toolkits. IBM Streams comes with a number of toolkits that provide functionality such as text analysis, HDFS integration, and GeoSpatial processing. Furthermore, if you're currently working with IBM Streams, it's possible that you've implemented your own toolkits that you'd like to utilize. 

The purpose of this guide is to demonstrate the basics of interfacing the Java Application API with such toolkits. This is important not only for backward compatibility, but also because it shows that that API can interact with C++ operators in addition to Java ones. Although it isn't assumed that the reader has an understanding of SPL and the structure of toolkits, consulting [the IBM Knowledge Center](http://www-01.ibm.com/support/knowledgecenter/SSCRJU_4.0.0/com.ibm.streams.dev.doc/doc/creating_toolkits.html?lang=en) may prove informative.

# A small bit of background on toolkits and SPL primitive operators
Before the Java Application API was released, developing IBM Streams was a two-step process that involved defining C++ or Java *primitive operators* (which manipulated data), and then subsequently connecting them together into an application by using a language called SPL. Since, as a java developer, you may not want to learn an entirely new programming language to build your application, the Java Application API presents a useful alternative.

Yet the primitive operators are still very useful. For one thing, many primitive operators are organized into *toolkits* which come packaged with any IBM Streams release to provide tools for machine learning, statistical analysis, and pattern recognition. In addition, since primitive operators can be written in C++, a developer using Java Application API can have certain portions of the application written in C++ if so desired.

In this tutorial, we will not cover [the development of C++ or Java primitive operators](http://www-01.ibm.com/support/knowledgecenter/SSCRJU_4.0.0/com.ibm.streams.dev.doc/doc/developing_primitive_operators.html?lang=en), however the process for utilizing an already existing toolkit is outlined below.

# A sample toolkit and operator
To begin, suppose that we have a 'myTk' toolkit in the home directory. In the 'myTk' toolkit, there is one package named 'myPackageName', and one operator named 'myOperatorName':
``` bash
$ cd ~/
$ tree
|--  ./myTk
|   |--  ./myTk/appendPackage
|   |   |-- ./myTk/appendPackage/appendOperator
|   |   |   |-- <implementation of appendOperator>
|   |--  ./myTk/toolkit.xml
```

 As such, when the toolkit is included into the application, the full path of the operator will be **appendPackage::appendOperator**. The 'appendOperator' operator itself is very simple, it takes an SPL rstring, appends it with the string " appended!", and submits the resulting rstring as an output tuple. For example, if the input to the 'appendOperator' operator are the following rstring tuples (one per line):
```
 Rhinoceros
 Modest Mouse
 The cake is a lie
```

The output output tuples will be:
```
 Rhinoceros appended!
 Modest Mouse appended!
 The cake is a lie appended!
```

It's important to note that every primitive operator is strongly typed with respect to the tuples that are sent to and emitted from the operator. As such, each primitive operator is associated with a *stream schema*, which simply contains the types of its input and output tuples. For example, the stream schema for both the input and output tuples of 'appendOperator' would be:
```
tuple<rstring attribute_name>
```
Which makes sense, since 'appendOperator' both takes and produces an rstring. A hypothetical operator that takes an rstring and an integer would look like:
```
tuple<rstring first_attribute_name, uint32 second_attribute_name>
```

You'll notice that each attribute of a tuple requires a corresponding name. This is because certain primitive operators require that an attribute have a particular name to operate correctly; however, the vast majority of operators shipped with IBM Streams are flexible and allow the usage of any name.

Defining stream schemas is not necessary when just developing exclusively withing the Java Application API -- it only becomes necessary with interfacing with primitive operators. We see why this is true in the next section.

# Using the toolkit within the Java Application API
In the introductory tutorials to the Java Application API, we created TStream objects which represented the flows of data in our application. To utilize a primitive operator, we must first convert our TStream to a special kind of stream called an SPLStream. SPLStreams are exactly like TStreams, except instead of being templated to a Java type, as in:
```
TStream<Double> myStream = ...;
```

Their types are instead defined by the *stream schemas* that were mentioned in the previous section, e.g.:
```
tuple<rstring attribute_name>
```

To create an SPLStream it's also necessary to provide a transformation Function to convert from the types of the TStream's Java objects to the SPLStream's SPL types. To show this in action, let's suppose that we have a TStream of Java Strings that we want to run through the 'appendOperator' that is found in the 'myTk' toolkit:
``` Java
Topology topology = new Topology("primitiveOperatorTest");
TStream<String> strings = topology.strings("Rhinoceros", "Modest Mouse", "The cake is a lie");
```

The first thing we do is import the toolkit by using the *addToolkit* utility method found in com.ibm.streamsx.topology.spl.SPL:
``` Java
SPL.addToolkit(topology, new File("/home/streamsadmin/myTk"));
```

Then, we convert the TStream of Strings to an SPLStream of SPL tuples -- each tuple corresponding to the stream schema of the operator:
``` Java
StreamSchema rstringSchema = Type.Factory.getStreamSchema("tuple<rstring rstring_attr_name>");
SPLStream splInputStream = SPLStreams.convertStream(strings, new BiFunction<String, OutputTuple, OutputTuple>(){
			@Override
			public OutputTuple apply(String input_string, OutputTuple output_rstring) {
				output_rstring.setString("rstring_attr_name", input_string);
				return output_rstring;
			}	
		}, rstringSchema);
```
In the above lines of code, the convertStream method takes three parameters
* **strings** - The TStream to convert to an SPL Stream
* **A BiFunction** - The BiFunction may appear complicated, but its functionality is easy to understand. It takes two arguments
  * It's first argument is a Java String. This is the current Java tuple on the TStream (e.g., "Rhinoceros") to be transformed to an SPL tuple. 
  * Its second argument is an OutputTuple. An OutputTuple can be thought of as a wrapper for an SPL tuple. It contains methods such as *setString* or *setDouble* which take Java types (Strings, Doubles, etc.) and converts them to SPL types according to the provided schema. In the above code, we can see that the "rstring_attr_name" attribute is set by invoking ```output_rstring.setString("rstring_attr_name", input_string);```. After being modified, the OutputTuple is returned by the BiFunction.
* **rstringSchema** - This is the stream schema for the input to the 'appendOperator' primitive operator. This determines the name used when calling *OutputTuple.setString*.

Voila! We've created an SPLStream of SPL types. To use this stream and invoke the 'appendOperator' on its tuples, it's only one line of code: 
``` Java
SPLStream splOutputStream = SPL.invokeOperator("appendPackage::appendOperator", splInputStream, rstringSchema, new HashMap());
```
Similar to the way that transforming a TStream produces another TStream, invoking a primitive operator on an SPLStream produces another SPLStream; in this case *splOutputStream*. You may have noticed that *invokeOperator* takes a Map as an argument, this is in case the primitive operator requires any parameters which, in this case, it doesn't.

Now that we have the output we desired, we'd like to print it to output. Unfortunately, since the tuples on the stream are SPL tuples and not Java tuples, we can't simply invoke ```splOutputstream.print()```. First, we need to convert the SPL tuples back the Java strings in a manner similar to the earlier conversion:
``` Java
TStream<String> javaStrings = splOutputStream.convert(new Function<Tuple, String>(){
			@Override
			public String apply(Tuple inputTuple) {
				return inputTuple.getString("rstring_attr_name");
			}			
		});
```
This time, we take a *Tuple* object, and retrieve the value of the "rstring_attr_name" parameter as a Java String by invoking ```inputTuple.getString("rstring_attr_name");```, resulting in a TStream of Strings.

Now we can simply call print():
``` Java
javaStrings.print();
```
and submit:
``` Java
StreamsContextFactory.getStreamsContext("STANDALONE").submit(topology).get();
```

When the application is run, it correctly produces the following output:
```
 Rhinoceros appended!
 Modest Mouse appended!
 The cake is a lie appended!
 ```
 
 # To reiterate
 When using a primitive operator, the general structure of your application be the following:
 1) Convert form a TStream to an SPLStream
 2) Pass the SPLStream as input when invoking the primitive operator
 3) Convert the output SPLStream back to a TStream
 
 The application, in its entirety, is as follows:
 ``` Java
 import java.io.File;
import java.util.HashMap;

import com.ibm.streams.operator.OutputTuple;
import com.ibm.streams.operator.StreamSchema;
import com.ibm.streams.operator.Tuple;
import com.ibm.streams.operator.Type;
import com.ibm.streamsx.topology.TStream;
import com.ibm.streamsx.topology.Topology;
import com.ibm.streamsx.topology.context.StreamsContextFactory;
import com.ibm.streamsx.topology.function.BiFunction;
import com.ibm.streamsx.topology.function.Function;
import com.ibm.streamsx.topology.spl.SPLStream;
import com.ibm.streamsx.topology.spl.SPL;
import com.ibm.streamsx.topology.spl.SPLStreams;


public class SPLTest {
	public static void main(String[] args) throws Exception{
		Topology topology = new Topology("SPLTest");
		SPL.addToolkit(topology, new File("/home/streamsadmin/scratch/myTk"));
		
		TStream<String> strings = topology.strings("Rhinoceros", "Modest Mouse", "The cake is a lie");
		
		StreamSchema rstringSchema = Type.Factory.getStreamSchema("tuple<rstring rstring_attr_name>");
		SPLStream splInputStream = SPLStreams.convertStream(strings, new BiFunction<String, OutputTuple, OutputTuple>(){
			@Override
			public OutputTuple apply(String input_string, OutputTuple output_rstring) {
				output_rstring.setString("rstring_attr_name", input_string);
				return output_rstring;
			}	
		}, rstringSchema);
		
		SPLStream splOutputStream = SPL.invokeOperator("appendPackage::appendOperator", splInputStream, rstringSchema, new HashMap());
		TStream<String> javaStrings = splOutputStream.convert(new Function<Tuple, String>(){
			@Override
			public String apply(Tuple inputTuple) {
				return inputTuple.getString("rstring_attr_name");
			}			
		});
		
		javaStrings.print();
		StreamsContextFactory.getStreamsContext("STANDALONE").submit(topology).get();				
	}
}
```
