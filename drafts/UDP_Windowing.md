# API Features

The Java Application API comes with a number of features that are of great use to a streams developer. Chief among these are User-Defined Parallelism (UDP), and windowing. Both features are implementations of standard patterns that are commonly seen in many streaming application frameworks.

## User-Defined Parallelism:
If a particular portion of your graph is bottelnecking, and there needs to be additional throughput, making it a parallel region allows multiple threads and processes to handle the various transformations and filterings of the data in parallel. Take the temperature reading example from the intro guide. Let's suppose that temperature reading are being taken so rapidly, that one thread is insufficient to convert it to Celcius to Kelvin quickly enough. In this case, parallel is a great tool:

~~~~~~
    public static void main(String args[]){    
        Topology topology = new Topology("temperatureSensor");
        Random random = new Random();
        
        @SuppressWarnings("unchecked")
        TStream<Double> readings = topology.endlessSource(new Supplier<Double>(){
            @Override
            public Double get() {
                // Temperature in Farenheit
                return random.nextGaussian();
            }
            
        });
        
        TStream<Double> parallelReadings = readings.parallel(5);
    
        TStream<Double> kelvin = parallelReadings.transform(new Function<Double, Double>(){
            @Override
            public Double apply(Double temp) {
                return convertToKelvin(temp);
            }
        });
        
        kelvin.endParallel().print();
        StreamsContextFactory.getEmbedded().submit(topology);
    }
~~~~~~

In the above example, we created a topology and defined a pseudo temperature source, just like in the introductory example. Now, instead of just printing it to output, we also want to convert all of the source tuples from Farenheit to Kelvin. To do this, we've applied the transformation function `convertToKelvin()`

~~~~~~
...
        public Double apply(Double temp) {
            return convertToKelvin(temp);
        }
...
~~~~~~

Granted, converting from Farenheit to Celcious is not a particularly expensive task, however in principle it could be any expensive, inefficient, bottlenecking operation. Parallelizing an operation is simple, simply invoke `.parallel()` on a TStream whose data you wish to process in parallel:

~~~~~~
TStream<Double> parallelReadings = readings.parallel(5);
~~~~~~

then any operations performed on the returned TStream will occur in parallel to the degree specified (in this case '5', meaning five thread or processes will be used to execute the parallel portion of the graph).

To end parallel processing, invoke `endParallel()` on one of the returned TStreams. This will ensure that subsequent operations on the TStream returned by `unparallel()` will **not** be in parallel. In the above code, we call unparallel on the returned `kelvin` stream before calling print:

~~~~~~
kelvin.endParallel().print();
~~~~~~

Had we not called unparallel at all, the `print()` statement would have also been performed in parallel.

The general workflow for parallelizing portions of you graph should look like:

~~~~~~
stream_of_data -> invoke parallel -> perform a number of parallel operations -> unparallel -> perform non-parallel operations
~~~~~~

When submitting the application, parallelized TStreams will run in **n** separate threads, if running in a STANDALONE context, or in **n** separate processes if running distributed. Unfortunately, running parallel() in EMBEDDED mode is not fully supported, and currently will simply run everything serially with a single thread.

Lastly, nested parallelism is not supported, so invoking `parallel()` on a TStream that's already been parallelized will throw an exception.

## Windowing
For operations where it's necessary to keep track of the last *n* tuples on the stream, windows provide convenient functionality. Let's suppose that we want to record the maximum temperature over the past 10 seconds from the following source `readings`:
~~~~~~
        Topology topology = new Topology("temperatureSensor");
        Random random = new Random();
        
        @SuppressWarnings("unchecked")
        TStream<Double> readings = topology.endlessSource(new Supplier<Double>(){
            @Override
            public Double get() {
                return random.nextGaussian();
            }
            
        });
~~~~~~

To do this, we can invoke the TStreams's `last()` method, which creates a Window of either the last **n** tuples, or the last **n** seconds. In this case, we want a window of the last ten seconds:
~~~~~~
        TWindow<Double, ?> lastTenSeconds = readings.last(10, TimeUnit.SECONDS);
~~~~~~

The Window is templated to two parameters: the type of the tuple in the window ('Double', in this case), and the type of the window's partition Key. For now, we won't go into keyable windows, so it's sufficient to simply provide '?' as an argument for the second template parameter. When a Window is defined in this manner, it can be thought of as a list of tuples upon which an operation can be performed. This operation, such as finding the maximum, can be specified using the `aggregate()` method on Window:

~~~~~~
        TStream<Double> maxTemp = lastTenSeconds.aggregate(new Function<List<Double>, Double>(){
            @Override
            public Double apply(List<Double> temps) {
                Double max = temps.get(0);
                for(Double temp : temps){
                    if(temp > max)
                        max = temp;
                }
                return max;
            }
            
        });
        maxTemp.print();
        StreamsContextFactory.getEmbedded().submit(topology);
~~~~~~

We can see that the aggregate method loops over the list of tuples, which are java Doubles, it finds the largest value and returns it from the method. The returned values from the `aggregate()` method become tuples on the returned TStream, which in this case is called `maxTemp`. An important note is that the Function supplied to `aggregate()` is invoked every time a tuple is sent onto the TStream feeding the window. In the above example, the supplied Function would be called whenever a new tuple went over the `lastTenSeconds` TStream.

Windows can be used inside of parallel regions. Lastly, Windows can be used inside of parallel regions by invoking `.parallel(n).last(m)` on a TStream.
