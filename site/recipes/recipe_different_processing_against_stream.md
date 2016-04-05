---
title: Recipe 4. Applying Different Processing Against a Single Stream
---

In the previous [recipe](recipe_value_out_of_range), we learned how to filter a stream to obtain the interesting sensor readings and ignore the mundane data. Typically, a user scenario is more involved, where data is processed using different stream operations. Consider the following scenario, for example.

Suppose a package delivery company would like to monitor the gas mileage of their delivery trucks using embedded sensors. They would like to apply different analytics to the sensor data that can be used to make more informed business decisions. For instance, if a truck is reporting consistently poor mileage readings, the company might want to consider replacing that truck to save on gas costs. Perhaps the company also wants to convert the sensor readings to JSON format in order to easily display the data on a web page. It may also be interested in determining the expected gallons of gas used based on the current mileage.

In this instance, we can take the stream of mileage sensor readings and apply multiple types of processing against it so that we end up with streams that serve different purposes.

## Setting up the application

We assume that the environment has been set up following the steps outlined in the [Getting Started Guide](../docs/quarks-getting-started). Let's begin by creating a `DirectProvider` and `Topology`. We choose a `DevelopmentProvider` so that we can view the topology graph using the console URL (refer to the [Application Console](../docs/console) page for a more detailed explanation of this provider). The initial mileage value and the number of miles in a typical delivery route have also been defined.

```java
    import java.text.DecimalFormat;
    import java.util.Random;
    import java.util.concurrent.TimeUnit;

    import com.google.gson.JsonObject;

    import quarks.console.server.HttpServer;
    import quarks.providers.development.DevelopmentProvider;
    import quarks.providers.direct.DirectProvider;
    import quarks.topology.TStream;
    import quarks.topology.Topology;

    public class ApplyDifferentProcessingAgainstStream {
        /**
         * Hypothetical values for the initial gas mileage and the
         * number of miles in a typical delivery route
         */
        static double currentMileage = 10.5;
        static double ROUTE_MILES = 80;

        public static void main(String[] args) throws Exception {

            DirectProvider dp = new DevelopmentProvider();

            System.out.println(dp.getServices().getService(HttpServer.class).getConsoleUrl());

            Topology top = dp.newTopology("TruckSensor");

            // The rest of the code pieces belong here
        }
    }
```

## Generating mileage sensor readings

The next step is to simulate a stream of gas mileage readings. In our `main()`, we use the `poll()` method to generate a flow of tuples (readings), where each tuple arrives every second. We ensure that the generated reading is between 7.0 mpg and 14.0 mpg.

```java
    // Generate a stream of mileage sensor readings
    DecimalFormat df = new DecimalFormat("#.#");
    Random r = new Random();
    TStream<Double> mileage = top.poll(() -> {
        // Change current mileage by some random amount between -0.4 and 0.4
        while (true) {
            double newMileage = -0.4 + (0.4 + 0.4) * r.nextDouble() + currentMileage;
            // Ensure that new temperature is within [7.0, 14.0]
            if (newMileage >= 7.0 && newMileage <= 14.0) {
                currentMileage = Double.valueOf(df.format(newMileage));
                break;
            } else {
                continue;
            }
        }
        return currentMileage;
    }, 1, TimeUnit.SECONDS);
```

## Applying different processing to the stream

The company can now perform analytics on the `mileage` stream and feed it to different functions. 

First, we can filter out mileage values that are considered poor and tag the resulting stream for easier viewing in the console.

```java
    // Filter out the poor gas mileage readings
    TStream<Double> poorMileage = mileage
            .filter(mpg -> mpg <= 9).tag("filtered");
```

If the company also wants the readings to be in JSON, we can easily create a new stream and convert from type `Double` to `JsonObject`.

```java
    // Map Double to JsonObject
    TStream<JsonObject> json = mileage
            .map(mpg -> {
                JsonObject jObj = new JsonObject();
                jObj.addProperty("mileage", mpg);
                return jObj;
            }).tag("mapped");
```

In addition, we can calculate the estimated gallons of gas used based on the current mileage using `modify`.

```java
    // Modify mileage stream to obtain a stream containing the estimated gallons of gas used
    TStream<Double> gallonsUsed = mileage
            .modify(mpg -> Double.valueOf(df.format(ROUTE_MILES / mpg))).tag("modified");
```

The three examples demonstrated here are a small subset of the many other possibilities of stream processing.

With each of these resulting streams, the company can perform further analytics, but at this point, we terminate the streams by printing out the tuples on each stream.

```java
    // Terminate the streams
    poorMileage.sink(mpg -> System.out.println("Poor mileage! " + mpg + " mpg"));
    json.sink(mpg -> System.out.println("JSON: " + mpg));
    gallonsUsed.sink(gas -> System.out.println("Gallons of gas: " + gas + "\n"));
```

We end our application by submitting the `Topology`.

## Observing the output

When the final application is run, the output looks something like the following:

```
    JSON: {"mileage":9.5}
    Gallons of gas: 8.4

    JSON: {"mileage":9.2}
    Gallons of gas: 8.7

    Poor mileage! 9.0 mpg
    JSON: {"mileage":9.0}
    Gallons of gas: 8.9

    Poor mileage! 8.8 mpg
    JSON: {"mileage":8.8}
    Gallons of gas: 9.1
```

## A look at the topology graph

Let's see what the topology graph looks like. We can view it using the console URL that was printed to standard output at the start of the application. We see that original stream is fanned out to three separate streams, and the `filter`, `map`, and `modify` operations are applied.

<img src="images/different_processing_against_stream_topology_graph.jpg">

## The final application

```java
    import java.text.DecimalFormat;
    import java.util.Random;
    import java.util.concurrent.TimeUnit;

    import com.google.gson.JsonObject;

    import quarks.console.server.HttpServer;
    import quarks.providers.development.DevelopmentProvider;
    import quarks.providers.direct.DirectProvider;
    import quarks.topology.TStream;
    import quarks.topology.Topology;

    /**
     * Fan out stream and perform different analytics on the resulting streams.
     */
    public class ApplyDifferentProcessingAgainstStream {
        /**
         * Hypothetical values for the initial gas mileage and the
         * number of miles in a typical delivery route
         */
        static double currentMileage = 10.5;
        static double ROUTE_MILES = 80;

        /**
         * Polls a simulated delivery truck sensor to periodically obtain
         * mileage readings (in miles/gallon). Feed the stream of sensor
         * readings to different functions (filter, map, and modify).
         */
        public static void main(String[] args) throws Exception {

            DirectProvider dp = new DevelopmentProvider();

            System.out.println(dp.getServices().getService(HttpServer.class).getConsoleUrl());

            Topology top = dp.newTopology("TruckSensor");

            // Generate a stream of mileage sensor readings
            DecimalFormat df = new DecimalFormat("#.#");
            Random r = new Random();
            TStream<Double> mileage = top.poll(() -> {
                // Change current mileage by some random amount between -0.4 and 0.4
                while (true) {
                    double newMileage = -0.4 + (0.4 + 0.4) * r.nextDouble() + currentMileage;
                    // Ensure that new temperature is within [7.0, 14.0]
                    if (newMileage >= 7.0 && newMileage <= 14.0) {
                        currentMileage = Double.valueOf(df.format(newMileage));
                        break;
                    } else {
                        continue;
                    }
                }
                return currentMileage;
            }, 1, TimeUnit.SECONDS);

            // Filter out the poor gas mileage readings
            TStream<Double> poorMileage = mileage
                    .filter(mpg -> mpg <= 9).tag("filtered");

            // Map Double to JsonObject
            TStream<JsonObject> json = mileage
                    .map(mpg -> {
                        JsonObject jObj = new JsonObject();
                        jObj.addProperty("mileage", mpg);
                        return jObj;
                    }).tag("mapped");

            // Modify mileage stream to obtain a stream containing the estimated gallons of gas used
            TStream<Double> gallonsUsed = mileage
                    .modify(mpg -> Double.valueOf(df.format(ROUTE_MILES / mpg))).tag("modified");

            // Terminate the streams
            poorMileage.sink(mpg -> System.out.println("Poor mileage! " + mpg + " mpg"));
            json.sink(mpg -> System.out.println("JSON: " + mpg));
            gallonsUsed.sink(gas -> System.out.println("Gallons of gas: " + gas + "\n"));

            dp.submit(top);
        }
    }
```
