---
title: Hello Quarks!
---

Quarks' pure Java implementation is a powerful feature which allows it to be run on the majority of JVM-compatible systems. It also has the added benefit of enabling the developer to develop applications entirely within the Eclipse and Intellij ecosystems. For the purposes of this recipe, it will be assumed that the developer is using Eclipse. To begin the Hello World recipe, create a new project and import the necessary libraries as outlined in the [Getting Started Guide](../docs/quarks-getting-started). Next, write the following template application:

``` java
    public static void main(String[] args) {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
    }
```

The *DirectProvider* is an object which allows the user to submit and run the final application. It also creates the *Topology* object, which gives the developer the ability to define a stream of strings.

## Using Topology.strings
The primary abstraction in Quarks is the `TStream`. A *TStream* represents the flow of data in a Quarks application; for example, the periodic floating point readings from a temperature sensor. The data items which are sent through a `TStream` are Java objects -- in the "Hello Quarks!" example, we are sending a single java.lang.String. There are a number of ways to create a `TStream`, and `Topology.strings` is the simplest. The user specifies a number of strings which will be used as the stream's data items.


``` java
    public static void main(String[] args) {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        TStream<String> helloStream = top.strings("Hello Quarks!");
    }
```

The `helloStream` stream is created, and the "Hello Quarks!" string will be sent as its single data item.
## Printing to Output
`TStream.print` can be used to print the data items of a stream to standard output by invoking the `toString` method of each data item. In this case the data items are already strings, but in principle `TStream.print` can be called on any stream, regardless of the datatype carried by the stream.

``` java
    public static void main(String[] args) {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        TStream<String> helloStream = top.strings("Hello Quarks!");
		helloStream.print();
    }
```

## Submitting the Application
The only remaining step is to submit the application, which is performed by the `DirectProvider`. Submitting a Quarks application initializes the threads which execute the `Topology`, and begins processing its data sources.

``` java
    public static void main(String[] args) {
        DirectProvider dp = new DirectProvider();
        Topology top = dp.newTopology();
        TStream<String> helloStream = top.strings("Hello Quarks!");
        helloStream.print();
        dp.submit(top);
    }
```

After running the application, the output is "Hello Quarks!":

```
Hello Quarks!
```




