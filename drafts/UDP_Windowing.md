# API Features

The Java Application API comes with a number of features that are of great use to a streams developer

## User-Defined Parallelism:
If a particular portion of your graph is bottelnecking, and there needs to be additional throughput, consider making it a parallel region. Take the temperature reading example from the intro guide. Let's suppose that temperature reading are being taken so rapidly, that one thread is insufficient to convert it to Celcius to Kelvin quickly enough. In this case, parallel is a great tool
``` Java
        
        Topology topology = new Topology("temperatureSensor");
        Random random = new Random();
        
        @SuppressWarnings("unchecked")
        TStream<Double> readings = topology.endlessSource(new Supplier<Double>(){
            @Override
            public Double get() {
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
```
Key points:
* it's necessary to call unparallel each time parallel is invoked.
* Nested parallelism is not supported. E.g. parallel().parallel().
* The inside of a parallel will run both in a separate thread, if running locally in STANDALONE, or in *n* separate processes if running distributed.
* When running in an EMBEDDED context, .parallel() does nothing. 

## Windowing
For operations where it's necessary to keep track of the last *n* tuples on the stream, windows provide convenient functionality. Let's suppose that we want to record the maximum temperature over the past 10 seconds:
``` Java        
        Topology topology = new Topology("temperatureSensor");
        Random random = new Random();
        
        @SuppressWarnings("unchecked")
        TStream<Double> readings = topology.endlessSource(new Supplier<Double>(){
            @Override
            public Double get() {
                return random.nextGaussian();
            }
            
        });
        
        TWindow<Double, ?> lastTenSeconds = readings.last(10, TimeUnit.SECONDS);
        
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
```
Key points:
* The aggregate function is called every time a tuple is sent on the stream.
* Here, we took the last ten *seconds* of tuples on the stream, but we could also take, say, the last 100 tuples on the stream.
* Windows can be used inside of parallel regions.
