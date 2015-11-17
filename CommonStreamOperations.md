# Common stream operations

After creating a TStream, the three primary operations that will be performed are *filter*, *transform*, and *sink*. Each operation accepts a user-supplied function to determine how to process one of the TStream's tuples. A few best practice use cases are outlined here.

## Filter

Invoking *filter* on a TStream allows the user to selectively allow and reject tuples from being passed along to another stream based on a provided predicate. For example, suppose that we have a TStream of String, where each String is a word out of the English dictionary.
``` Java
TStream<String> words = topology.source(/*code for reading from dictionary*/);
```
Furthermore, suppose that we want only a TStream of dictionary words that do not contain the letter "a". To create such a TStream, we simply invoke `.filter()`on the TStream of words:

``` Java
TStream<String> words = topology.source(/*code for reading from dictionary*/);
TStream<String> wordsWithoutA = words.filter(new Predicate<String>(){
    	@Override
			public boolean test(String word) {
				if(tuple.contains("a")){
					return false;
				}
				return true;
			}
		});
```

Or, more concisely by using Java 8 lambda expressions:

``` Java
TStream<String> words = topology.source(/*code for reading from dictionary*/);
TStream<String> wordsWithoutA = words.filter(word -> word.contains("a") ? false : true);
```
		
The returned TStream, *wordsWithoutA*, now contains all words without a lowercase "a". Whereas before, the output might have been:
```
...
qualify
quell
quixotic
quizzically
...
```
The output now is:
```
...
quell
quixotic
...
```

You'll notice that we provide a predicate function, and need to override only its `test()`method to return true or false for respectively permitting and rejecting the tuple. 

**Important:** While the code has access to the tuples of the TStream, and thus can modify their contents, a filter is intended to be an *immutable* operation. As such, tuples should not be altered. For details about transforming the contents or type of a tuple, refer to the transform operation in the next section.

## Transform

*Transform* is the workhorse of the Java Application API. It will likely be the most frequently used method in your application. Primarily, the `.transform()`method is responsible for taking the tuples of a TStream and doing any one of the following:
* Modifying the contents of the tuple
* Changing the type of the tuple
* Keeping track of state across tuples

For each of these, we'll walk through an example.
### Transform: Modifying tuple contents
Let's take the previous example of reading words from a dictionary. It's not necessarily the case that we want *exactly* the tuples of a stream; we might need to modify them before we use them. Instead of having a TStream of dictionary words, what if we wanted a TStream of only the first four letters of each word? The transform operation is best suited to this task because it permits data-modifying operations on the tuple:
``` Java
TStream<String> words = topology.source(/*code for reading from dictionary*/);
TStream<String> firstFourLetters = words.transform(new Function<String, String>(){
            @Override
            public String apply(String word) {
                return v.substring(0,4);
            }
            
        });
```
Or, more concisely by using Java 8 lambda expressions:

``` Java
TStream<String> words = topology.source(/*code for reading from dictionary*/);
TStream<String> firstFourLetters = words.transform(word -> word.substring(0,4));
```
Now, instead of the entire word, printing the *firstFourLetters* TStream produces this output:
```
...
qual
quel
quix
quiz
...
```
As you can see, invoking `.transform()` allows for the modification of tuples. Yet what if your application seeks to change the type? For the previous example, both the inputs and outputs of the transform function were strings. In the next section, we will demonstrate how `transform()`can change tuple types entirely.

### Transform: Changing the tuple type
The `transform()`method does not require that the input tuple type is the same as the output tuple type. In fact, one of the strengths of the Java Application API is that the tuples on a TStream can be used as parameters when creating tuples to pass on the output TStream. As an example, let's suppose that we have a TStream of Java Strings, each corresponding to an Integer:
``` Java
// Creates a TStream with four Java Strings as tuples -- "1", "2", "3", and "4"
TStream<String> stringIntegers = topology.strings("1","2","3","4");
```
If we want to perform an operation that treats the tuples as Java Integers (and not Java Strings), we need to transform the tuples to a new type using `.transform()`. This could be done as follows:

``` Java
// Creates a TStream with four Java Strings as tuples -- "1", "2", "3", and "4"
TStream<String> stringTuples = topology.strings("1","2","3","4");
TStream<Integer> integerTuples = stringTuples.transform(new Function<String, Integer>(){
            @Override
            public Integer apply(String stringInt) {
                return Integer.parseInt(stringInt);
            }    
        });
```

Or, as a Java 8 lambda expression:

``` Java
// Creates a TStream with four Java Strings as tuples -- "1", "2", "3", and "4"
TStream<String> stringTuples = topology.strings("1","2","3","4");
TStream<Integer> integerTuples = stringTuples.transform(stringTuple -> Integer.parseInt(stringTuple));
```
Now that actual Java Integers are being passed as tuples on the Stream, operations such as addition, subtraction, and multiplication can directly be invoked:
``` Java
// Creates a TStream with four Java Strings as tuples -- "1", "2", "3", and "4"
TStream<String> stringTuples = topology.strings("1","2","3","4");
TStream<Integer> integerTuples = stringTuples.transform(stringTuple -> Integer.parseInt(stringTuple));
integerTuples.transform(integerTuple -> integerTuple * 2 + 1).print();
```

Yielding the following output:
```
3
5
7
9
```
Although this example only converts Strings to Integers, in principle it could work for any two arbitrary Java types, so long as they are both serializable. 

Another strength of the API is that users aren't restricted to only passing datatypes defined by the Java runtime (String, Integer, Double, and so on) -- users can define their own classes and datatype, and pass them as tuples on a TStream.

### Transform: Keeping track of state across tuples
The previous examples would be termed *stateless* operators. A stateless operator does not keep track of any information about tuples that have been seen in the past, for example, the number of tuples that have been passed on a TStream, or the sum of all Integers seen on a TStream. Yet keeping track of state is both an easy and powerful part of the Java Application API, enabling a much broader range of applications.

Although the following example pertains primarily to the `transform()` method, in principle *any* of the source, filter, sink, modify, or transform methods can keep track of state. 

An example of a stateful operator would be one which outputs the average of the last ten Doubles in a TStream. For this example, we first define a TStreams of random numbers:
``` Java
TStream<Double> doubles= topology.endlessSource(new Supplier<Double>(){
            Random random = new Random();
            @Override
            public Double get() {
                return random.nextGaussian();
            }  
      });
```

You'll note that in defining the *doubles* TStream, Supplier contains the **random** field, which is used to generate the random numbers.

Next, we want to define an operation which consumes the *doubles* stream and keeps track of its moving average across the last ten tuples. Similar to how **random** was defined as a part of the internal state of *doubles*, we can define a LinkedList to keep track of the tuples on the TStream:

``` Java
TStream<Double> avg = doubles.transform(new Function<Double, Double>(){
            LinkedList<Double> lastTen = new LinkedList<>();
            @Override
            public Double apply(Double d) {
                lastTen.addLast(d);
                if(lastTen.size() > 10)
                    lastTen.removeFirst();
                return calculateAverage(lastTen);
            }
            
        });
```
This is an important point: the state of the operator does *not* reset between tuples. If there are eight tuples in the *lastTen* LinkedList at the start of the invocation of `apply()`, then the next invocation of `apply()` immediately following will see nine tuples in the LinkedList.

Although in this case our state is a LinkedList with the ultimate goal of calculating a moving average, the following are some examples of state used in sources, sinks, and transformations.

| Operation        | State           | Goal  |
| ------------- |-------------| -----|
| Source	| File Handle	| Reading lines from a file to send as tuples	|
| Source	| Socket	| Listening on a TCP socket for data to send as tuples	|
| Transform | HashMap | A HashMap can be used as state to make a histogram of the most commonly seen tuples|
| Transform | Set | A set can be used as state to keep track of all unique tuples on a TStream |
| Sink | File Handle | Similar to source, except the file handle is used to write tuples to a file |
| Sink | Socket | Similar to source, except the socket is used to write tuples to a TCP stream |

## Sink
*Sink* takes tuples from a stream, but does not return a stream as output. It is invoked on a TStream when the tuples themselves are output to the system. The output can take the form of a file, a log, a TCP connection, HDFS, Kafka, or anything that you can define -- because operations such as sink can be stateful, you can provide any handle that is required.

TStream comes with the `.print()`method out of the box. The `.print()`method is a sink that simple invokes ``` System.out.println(tuple.toString()); ``` on every tuple that is sent on the stream.

The following is an example of a sink that writes the string representation of a tuple to standard error instead of standard output:
``` Java
TStream<String> strings = ...;
strings.sink(new Consumer<String>(){
            @Override
            public void accept(String string) {
                System.err.println(string);             
            }     
  });
```
Or, more concisely with Java 8 lambda expressions:
``` Java
TStream<String> strings = ...;
strings.sing(string -> System.err.println(string));
```
For a more in-depth tutorial on how to write a sink that writes to a file, visit the [file reading/writing](http://www.google.com) introduction. Or, for Kafka, the [digesting and ingesting with Kafka](www.google.com) tutorial should contain relevant information.
