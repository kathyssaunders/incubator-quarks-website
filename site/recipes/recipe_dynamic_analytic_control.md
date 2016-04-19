---
title: Dynamically Enabling Analytic Flows
---

This recipe addresses the question: How can I dynamically enable or disable entire portions of my application's analytics?

Imagine a topology that has a variety of analytics that it can perform.  Each analytic flow comes with certain costs in terms of demands on the CPU or memory and implications for power consumption.  Hence an application may wish to dynamically control whether or not an analytic flow is currently enabled.

## Valve

A ``quarks.topology.plumbing.Valve`` is a simple construct that can be inserted in stream flows to dynamically enable or disable downstream processing.  A Valve is used as a Predicate to ``TStream.filter()``.  The valve is either OPEN, tuples are accepted, or CLOSED, tuples are rejected, essentially enabling or disabling downstream processing respectively.

For example, consider a a topology consisting of 3 analytic processing flows that want to be dynamically enabled or disabled:

```java
    Valve<Readings> flow1Valve = new Valve<>();  // default is OPEN
    Valve<Readings> flow2Valve = new Valve<>(Valve.State.CLOSED);
    Valve<Readings> flow3Valve = new Valve<>(Valve.State.CLOSED);

    TStream<Readings> readings = topology.poll(mySensor, 1, TimeUnit.SECONDS);
    addAnalyticFlow1(readings.filter(flow1Valve));
    addAnalyticFlow2(readings.filter(flow2Valve));
    addAnalyticFlow3(readings.filter(flow3Valve));
```

Elsewhere in the application, perhaps as a result of processing some device command from an external service such as when using an IotProvider or IotDevice, valves may be opened and closed dynamically to achieve the desired effects.  For example:

```java
   TStream<JsonObject> cmds = simulatedValveCommands(topology);
   cmds.sink(json -> {
       String valveId = json.getPrimitive("valve").getAsString();
       String stateName = json.getPrimitive("state").getAsString();
       Valve.State state = Valve.State.valueOf(stateName);
       switch(valveId) {
         case "flow1": flow1Valve.setState(state); break;
         case "flow2": flow2Valve.setState(state); break;
         case "flow3": flow3Valve.setState(state); break;
       }
   });
```

## Loosely Coupled Quarks Applications

Another approach for achieving dynamic control over what analytics flows are running is to utilize loosely coupled applications.  The following is a brief introduction to the topic.

In this approach, the overall application is partitioned into multiple applications (topologies). In the above example there could be four applications: one that publishes the sensor Readings stream, and one for each of the analytic flows.

The separate applications can connect to each other using the ``quarks.connectors.pubsub.PublishSubscribe`` connector.

Rather than having all of the analytic topologies running all of the time, applications can be registered with an ApplicationService.  Registered applications can then be started dynamically and a started application, a "job", can be dynamically cancelled.

The Quarks IotProvider packages up all of these services and more in support of this style of use.