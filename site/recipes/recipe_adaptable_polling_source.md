---
title: Adaptable Polling Source Frequency
---

The [Writing a Source Function](recipe_source_function.html) recipe introduced the basics of creating a source stream by polling a data source periodically.

Oftentimes, a user wants the polling frequency to be adaptable rather than static.  For example, an event such a sudden rise in a temperature sensor may motivate more frequent polling of the sensor and analysis of the data until the condition subsides.  A change in the polling frequency may be a result of locally performed analytics or via a command from an external source.

A Quarks IotProvider and IoTDevice with its command streams would be a natural way to control the application.  In this recipe we will just simulate a "set polling frequency" command stream.

The ``Topology.poll()`` documentation describes how the polling frequency may be changed at runtime.

The mechanism is based on a more general Quarks runtime ``quarks.execution.services.ControlService`` service.  The runtime registers "control beans" for entities that are controllable.  These controls can be retrieved at runtime via the service.

At runtime, ``Topology.poll()`` registers a ``quarks.execution.mbeans.PeriodicMXBean`` control. Retrieving the control at runtime requires setting an alias on the poll generated stream using ``TStream.alias()``.

## Create the polled stream and set its alias

```java
        Topology top = ...;
        SimulatedTemperatureSensor tempSensor = new SimulatedTemperatureSensor();
        TStream<Double> engineTemp = top.poll(tempSensor, 1, TimeUnit.SECONDS)
                                      .alias("engineTemp")
                                      .tag("engineTemp");
```

It's also a good practice to add tags to streams to improve the usability of the development mode Quarks console.

## Define a set polling frequency method

```java
    static <T> void setPollFrequency(TStream<T> pollStream, long period, TimeUnit unit) {
        // get the topology's runtime ControlService service
        ControlService cs = pollStream.topology().getRuntimeServiceSupplier()
                                    .get().getService(ControlService.class);

        // using the the stream's alias, get its PeriodicMXBean control
        PeriodicMXBean control = cs.getControl("periodic", pollStream.getAlias(), PeriodicMXBean.class);

        // change the polling frequency using the control
        System.out.println("Setting period="+period+" "+unit+" stream="+pollStream);
        control.setPeriod(period, unit);
    }
```

## Process the "set polling frequency" command stream

Our commands are on the "TStream&lt;JsonObject&gt; cmds" stream.  Each JsonObject tuple is a command with the properties "period" and "unit".

```java
        cmds.sink(json -> setPollFrequency(engineTemp,
            json.getAsJsonPrimitive("period").getAsLong(),
            TimeUnit.valueOf(json.getAsJsonPrimitive("unit").getAsString())));
```

## The final application

```java
import java.util.Date;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

import com.google.gson.JsonObject;

import quarks.execution.mbeans.PeriodicMXBean;
import quarks.execution.services.ControlService;
import quarks.providers.development.DevelopmentProvider;
import quarks.providers.direct.DirectProvider;
import quarks.samples.utils.sensor.SimulatedTemperatureSensor;
import quarks.topology.TStream;
import quarks.topology.Topology;

/**
 * A recipe for a polling source stream with an adaptable polling frequency.
 */
public class AdaptablePollingSource {

    /**
     * Poll a temperature sensor to periodically obtain temperature readings.
     * Respond to a simulated command stream to change the polling frequency.
     */
    public static void main(String[] args) throws Exception {

        DirectProvider dp = new DirectProvider();

        Topology top = dp.newTopology("TemperatureSensor");
        
        // Generate a polled temperature sensor stream and set its alias
        SimulatedTemperatureSensor tempSensor = new SimulatedTemperatureSensor();
        TStream<Double> engineTemp = top.poll(tempSensor, 1, TimeUnit.SECONDS)
                                      .alias("engineTemp")
                                      .tag("engineTemp");

        // Report the time each temperature reading arrives and the value
        engineTemp.peek(tuple -> System.out.println(new Date() + " temp=" + tuple));
        
        // Generate a simulated "set polling frequency" command stream
        TStream<JsonObject> cmds = simulatedSetPollingFrequencyCmds(top);
        
        // Process the commands to change the polling frequency
        cmds.sink(json -> setPollFrequency(engineTemp,
            json.getAsJsonPrimitive("period").getAsLong(),
            TimeUnit.valueOf(json.getAsJsonPrimitive("unit").getAsString())));

        dp.submit(top);
    }
    
    static <T> void setPollFrequency(TStream<T> pollStream, long period, TimeUnit unit) {
        // get the topology's runtime ControlService service
        ControlService cs = pollStream.topology().getRuntimeServiceSupplier()
                                    .get().getService(ControlService.class);

        // using the the stream's alias, get its PeriodicMXBean control
        PeriodicMXBean control = cs.getControl("periodic", pollStream.getAlias(), PeriodicMXBean.class);

        // change the polling frequency using the control
        System.out.println("Setting period="+period+" "+unit+" stream="+pollStream);
        control.setPeriod(period, unit);
    }
    
    static TStream<JsonObject> simulatedSetPollingFrequencyCmds(Topology top) {
        AtomicInteger lastPeriod = new AtomicInteger(1);
        TStream<JsonObject> cmds = top.poll(() -> {
                // toggle between 1 and 2 sec period
                int newPeriod = lastPeriod.get() == 1 ? 2 : 1;
                lastPeriod.set(newPeriod);
                JsonObject jo = new JsonObject();
                jo.addProperty("period", newPeriod);
                jo.addProperty("unit", TimeUnit.SECONDS.toString());
                return jo;
            }, 5, TimeUnit.SECONDS)
            .tag("cmds");
        return cmds;
    }

}

```
