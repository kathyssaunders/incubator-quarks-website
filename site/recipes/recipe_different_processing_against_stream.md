---
title: Recipe 4. Applying Different Processing Against a Single Stream
---

In the previous [example](recipe_value_out_of_range), we learned how to filter a stream to obtain the interesting sensor readings and ignore the mundane data. Typically, a user scenario is more involved, where data is processed by executing multiple stream operations sequentially. Consider the following scenario, for example.

Suppose a package delivery company would like to monitor the gas mileage of their delivery trucks using embedded sensors. They would like to apply different analytics to the sensor data that can be used to make more informed business decisions. For instance, if a truck is reporting consistently poor mileage readings, the company might want to consider replacing that truck to save on gas costs.

In this instance, we can apply different processing against the stream of mileage readings from the sensor and generate warnings when poor gas mileage is detected.

## Setting up the application

We assume that the environment has been set up following the steps outlined in the [Getting Started Guide](../docs/quarks-getting-started). Let's begin by creating a `DirectProvider` and `Topology`. We choose a `DevelopmentProvider` so that we can view the topology graph using the console URL (refer to the [Application Console](../docs/console) page for a more detailed explanation of this provider). The initial mileage value has also been defined.

```java
    public class TruckSensor {
        /**
        * Hypothetical value for initial gas mileage
        */
        static double currentMileage = 10.5;

        public static void main(String[] args) throws Exception {

            DirectProvider dp = new DevelopmentProvider();

            System.out.println(dp.getServices().getService(HttpServer.class).getConsoleUrl());

            Topology top = dp.newTopology("TruckSensor");

            // The rest of the code pieces belong here.
        }
    }
```

## Generating mileage sensor readings

The next step is to simulate a stream of gas mileage readings. In our `main()`, we use the `poll()` method to generate a flow of tuples (readings), where each tuple arrives every second.

```java
    // Generate a stream of mileage sensor readings.
    Random r = new Random();
    TStream<Double> mileage = top.poll(() -> {
        // Change current mileage by some random amount between -0.4 and 0.4.
        double newMileage = -0.4 + (0.4 + 0.4) * r.nextDouble() + currentMileage;
        currentMileage = newMileage;
        return currentMileage;
    }, 1, TimeUnit.SECONDS);
```

## Applying different processing to the stream

The company can now perform analytics on the `mileage` stream to generate poor gas mileage warnings by starting with the original stream and altering it. First, we filter out values that are out of range. Then, we tag the stream with the _mileage_ tag for easier viewing in the console (we add this tag after every step). Next, we truncate the reading to one decimal place. We then use `peek` to print out the current mileage so that we can get a sense of the truck's current state. After, we keep the mileage readings that are considered "poor." Finally, we terminate the stream by printing out warnings for poor gas mileage.

```java
    DecimalFormat df = new DecimalFormat("#.#");

    // Perform analytics on mileage readings to generate poor gas mileage warnings.
    mileage = mileage.filter(tuple -> tuple >= 7.0 && tuple <= 14.0);
    mileage = mileage.tag("mileage");
    mileage = mileage.modify(tuple -> Double.valueOf(df.format(tuple)));
    mileage = mileage.tag("mileage");
    mileage = mileage.peek(tuple -> System.out.println("Mileage: " + tuple + " mpg"));
    mileage = mileage.tag("mileage");
    mileage = mileage.filter(tuple -> tuple <= 9);
    mileage = mileage.tag("mileage");
    mileage.sink(tuple -> System.out.println("Poor gas mileage! Only getting " + tuple + " mpg!"));
```

We end our application by submitting the `Topology`.

## Observing the output

When the final application is run, the output looks something like the following:

```
    Mileage: 9.6 mpg
    Mileage: 9.3 mpg
    Mileage: 9.6 mpg
    Mileage: 9.3 mpg
    Mileage: 9.4 mpg
    Mileage: 9.4 mpg
    Mileage: 9.2 mpg
    Mileage: 9.0 mpg
    Poor gas mileage! Only getting 9.0 mpg!
```

## A look at the topology graph

Let's see what the topology graph looks like. We can view it using the console URL that was printed to standard output at the start of the application. Notice how the graph makes it easier to visualize the various stream operations.

<img src="images/different_processing_against_stream_topology_graph.jpg">

## The final application

```java
    import java.text.DecimalFormat;
    import java.util.Random;
    import java.util.concurrent.TimeUnit;

    import quarks.console.server.HttpServer;
    import quarks.providers.development.DevelopmentProvider;
    import quarks.providers.direct.DirectProvider;
    import quarks.topology.TStream;
    import quarks.topology.Topology;

    public class TruckSensor {
        /**
         * Hypothetical value for initial gas mileage
         */
        static double currentMileage = 10.5;

        /**
         * Polls a simulated delivery truck sensor to periodically obtain
         * mileage readings (in miles/gallon). Perform a series of data
         * analysis steps on the stream to generate warnings.
         */
        public static void main(String[] args) throws Exception {

            DirectProvider dp = new DevelopmentProvider();

            System.out.println(dp.getServices().getService(HttpServer.class).getConsoleUrl());

            Topology top = dp.newTopology("TruckSensor");

            // Generate a stream of mileage sensor readings.
            Random r = new Random();
            TStream<Double> mileage = top.poll(() -> {
                // Change current mileage by some random amount between -0.4 and 0.4.
                double newMileage = -0.4 + (0.4 + 0.4) * r.nextDouble() + currentMileage;
                currentMileage = newMileage;
                return currentMileage;
            }, 1, TimeUnit.SECONDS);

            DecimalFormat df = new DecimalFormat("#.#");

            // Perform analytics on mileage readings to generate poor gas mileage warnings.
            mileage = mileage.filter(tuple -> tuple >= 7.0 && tuple <= 14.0);
            mileage = mileage.tag("mileage");
            mileage = mileage.modify(tuple -> Double.valueOf(df.format(tuple)));
            mileage = mileage.tag("mileage");
            mileage = mileage.peek(tuple -> System.out.println("Mileage: " + tuple + " mpg"));
            mileage = mileage.tag("mileage");
            mileage = mileage.filter(tuple -> tuple <= 9);
            mileage = mileage.tag("mileage");
            mileage.sink(tuple -> System.out.println("Poor gas mileage! Only getting " + tuple + " mpg!"));

            dp.submit(top);
        }
    }
```
