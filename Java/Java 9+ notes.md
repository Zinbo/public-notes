# What are JEPs and JSRs?

## JEP
Java Enhancement Proposal.
A JEP is a document that is prosing an enhancement to Java core technology. These are proposals are typically for enhancements that are not ready to be specified yet. As the JEP-0 document states, JEPs may call for exploration of novel ideas. Generally speaking, prototyping will be required to separate the viable and non-viable ideas and clarify them to the point where a specification can be produced.

## JSR
Java Specification Request.
A JSR is a document created as part fo the Java Community Process (JCP) that is setting the scope for a team of people to develop a new specification. These specifications are (AFAIK) always Java related, but they frequently address things that are not going to be core to Java SE or Java EE technologiy. A typical JSR's subject material is a relatively nature technology; i.e. on that is in a state that ca be specified. 

## The relationship between them
1. JEPs propose and develop experimental ideas to the point where they could be specified. Not all JEPs come to fruition.
2. JSRs take mature ideas (e.g. resulting from a JEP), and produce a new specification, or modifiecations to an existing specification. Not all JSRs come to fruition.

# Changes to release model
Java has now moved t oa 5 months cadence.

Only every 3 years will a version have LTS:
- java 11, Java 17, etc.

We're now on Java 13, released 17 September, so the next LTS Java will be Java 17, which should be released around Sepetember 2021.

The reasons for this are many, but maiunly because Java was falling behind as a language. Java 7 was released July 2011 and Java 8 wasn't released until March 2014! These were big bang changes, and followed an almost waterfall pattern. If a feature didn't make it into the release you could be waiting quite a few years before you'd see it. Other languages were releasing at a much more frequent rate and gaining more popularity.
Another benefit is that the developers (supposedly) feel more relaxed now if a feature doesn't quite make it into the next release, as it'll get into the release after thyat 6 months later.

# Changes
## Java 9

### No more Java EE
JavaEE is no longer part of the core JDK. It has been handed over to Eclipse, which has called it Jakarta, as Oracle don't want to be associated with them. This includes XML bindings, JPA, DI, etc.

Currently Jakarta is at version 8. Their first goal was to completely replicate JavaEE 8 so that they had a solid foundation to work on. They're now working on adding more features. The scope of this could fill an entire learning series, so I won't go into more detail here.

### Modular Java
What does it mean to be modular?
Simply put, modularity is a design principle that helps us to achieve:
- loose coupling between components
- clear contracts and dependnecies between components
- hidden implementation using strong encapsulation

Aren't Jars modular though? yes however:
- There are explicit contracts and dependencies between jars
- there is weak encapsulation of elelemtns within the jars

There was another problem with JARs - the JAR hell. Multiple versions fo the JARs lying on the classpath resulted in the ClassLoader loading the first found class from the JAR, with very unexpected results.

The other issue with the JVM using classpath was that the compilation of the application would be successful but the application would fail at runtim with ClassNotFoundException, due to the missing JARs on the classpath at runtime.

One of Oracle's goals was to modularise the JDK. This would mean that they would be able to create smaller runtim libraries with a subset of modules from the JDK. Another goal was to encapsulation the internal APIs under the sun.* package, as they were never meant to be used by the public.

Each module requires a module-info.java file. This contains:
- its name
- the package it makes available publicly
- the modules it depends on
- any services it consumes
- any implmenration for the service it provides.

The last two items i nthe above list are not commoinly used. They are used only when services are being provided and consumed via the java.util.ServiceLoaded interface.

You place these module-info files in packages to modularise a package.

There is now a concept of link time, which happens between compile time and run time, in which a set of modules acan be assembled and optimised int oa custom run-time image.

### A better HTTP client
Not many people use the default HttpURLCOnnection API in Java, most people will use Apache HttpClient, Jetty, or RestTemplate. This is because HttpUrlConnection isn't very user friendly.

The new API has 3 core classes:
- HttpRequest
- HttpClient
- HttpResponse

The new HTTP client supports:
- bot hsynchronous and asynchronous requests
- HTTP/2 and HTTP/1.1 compatibility

HTTP/2 is a big step up from HTTP/1.1, which was last relased in 1997. HTTP/3 is getting more support now as awell, so hopefully we should see support for this in Java in the future too.

#### Finally a REPL
REPL - Read Eval Print Loop
Other languages have this, for example python. You can type `python` into your cmd to start the language shell and execute each line you write.

You'd mostly use it for seeing the output for small amounts of code, or learning a new feature.

jShell doesn't require you to write complete java code. You don't need to declare classes if you don't want to, you don't need to include sem-colons.  
You can define variables, methods, classes, imports, expressions. You can also completely overwrite any of these.

### Emojis4J
Unicode 7 is here in Java, which means you can finally use emojis!
We have to be careful here though. Lets look at a an exmaple:
```java
public class Emojis4J {
    public static void main(String[] args) {
        String bear = "🐻";
        char bearChar = bear.charAt(0);

        String aString = "a";
        char aChar = stringWithSingpleChar.charAt(0);

        System.out.println("The emoji from String is: " + bear)
        System.out.println("a char is " + aChar);
        System.out.println("Bear char is: " + bearChar);
    }
}
```

This prints out:
```
The emoji from string is: 🐻
a char is: a
Bear char is: ?
```
The bear char didn't work. Why not?
This is because the emojis are 5 digit hex numbers. Each hex number needs 4 bits, therefore a 5 digit hex number needs 5x4=20 bits. The bear emoji is represented by \u1f43b. A java character is 2 bytes=2x8 bits = 16 bits. Not big enough to hold an emoji!
To get around this you can represent emojis with two 16 bit hexadecimal values. The bear is represented with the string \ud83d\udc3b.  
These are called surrogate pairs, and are used to solve the problem of chars in languages nnot big enough to hold some unicode characters.  
We can conver the bear to 2 chars properly like so:
```java
public class Emojis4J {
    public static void main(String[] args) {
        String bear = "BEARCHANGEME";
        int bearCodepoint = bear.codePointAt(bear.offsetCodePoints(0, 0));
        char[] bearSurrogates = {Character.highSurrogate(bearCodepoint), Character.lowSurrogate(bearCodepoint)};
        System.out.println("Bear in surrogate and code point for:" + String.valueOf(bearSurrogates));
    }
}
```

### Keystores
Transition the default keystore type from JKS to PKCS12.

### Java on a raspberry pi?
Port JDK 9 to Linux/AArch64

### Smarter Strings
Strings are stored in an array of chars, with each char taking 2 bytes (for unicode-16).  
However analysis has shown that most characters are from Latin-1, which only requires 1 byte. An ecoding flag has been added to String which, based on the content of the string, will uyse either two bytes per character or one byte.  

### Factory methods for collections
Finally, java offers a standard way to instantiate collections.  
There is now immutable collections which instantiate the COllection interface but don't implement any mutable methods.
```java
public class JavaCollections {
    public static void main(String[] args) {
        //Immutable Collections:
        // Empty List
        List.of();

        // Easy initialisation of list, array-based List implementation (if more than 2 elements)
        final List<Stirng> immutableList = List.of("String1", "String2");
        // This will throw the exception: Exception in thread "main" java.lang.UnsupportedOperationException
        immutableList.add("String3");

        // Empty map
        Map.of();

        // Easy initialisation of map, array-based Map implementation
        Map.of("My key1", 1, "My Key2", 2);

        // Mutable Collections
        // Not great, this way might end up creating up to 3 arrays
        List<String> mutableList = new ArrayList<>(List.of("String1"));

        // This will work
        mutableList.add("String4");
    }
}
```

### Applets finally die
The Applet API is deprecated. An Applet is a java program that can be embedded into a web page, as a way to make a web page more dynamic. Youmight have seen them in tyhe past, but I doubt you see them anymore!

### Goodbye Concurrent Mark Sweep (CMS)
The Concurrent Mark Sweep Garbage Collection was deprecated.  

The Concurrent Mark Sweep (CMS) collector is designed for applications that prefer shorter garbage collection pauses and that can afford to share processor resources with the garbage collector while the application is running. Typically applications that have a relatively large set of long-lived data (a large tenured generation) and run on machines with two or more processors tend to benefit from the use of this collector. However, this collector should be considered for any application with a low pause time requirement.

The decision was made to reduce the maintenance burden of GC cdoe base and accelerate new development.

### Hello (to a subset of) ECMAScript 6
ECMAScript 6 features are added to Nashorn.  
Well, some of them.  
THe rest will be delivered in future major JDK releases.   

...Or will they?

### Exprimenting with Ahead-of-Time compilation
The way that most JVMs (roughly) work is like so:
- byte code is executed by the JVM
- The JVM will either:
    - interpret the byte code
    - compile it to machine code using what is known as the Just-In-Time (JIT) compiler
        - with HotSpot JVMs they'll only use JIT if some piece of code is often used

This is great, but it can mean that large programs the JIT can take a long time to "warm up", and code may never be compiled at all (so it will execute slower).

Ahead-of-Time compilation allows us to compile java classes to native code prior to launching the JCM. This would mean that our program should run a lot faster, as there is no interpretation or compilation happening whilst the program is running.

## Java 10

### Finally, the var keyword
Type inference is now extended to declaration of local variables with initialisers.

Remember that some type inference was already in Java
```java
int maxWeight = blocks.steam()
    .filter(b -> b.getColor() == BLUE)
    .mapToInt(Block::getWeight)
    .max();
```

It's important to remember that objects still have a type - it's just that the type is inferred. It's not like JavaScript, you can't just switch a variables type.

### Improving G1GC
G1 GC was made the default GC in JDK 9. G1 GC was designed to avoid full collections, but when the concurrent collections can't reclaim memory fast enough, a fall back GC will occur.

This feature will improve G1 worst-case latencies by making the full GC parallel.

The old default was the parallel collection, which had a parallel full GC.

### Dogfooding their own language
There is a new JIT compiler, which is written in Java. Previously these were written in C++ (a specific dialect of C++ apparently, which made it hard for new developers to add new features.)

This is still in the early experimental stages, but wouldn't it be great to have a JIT compiler that developers can improve?

So we now have:
- A new experimental JIT compiler
- An expertimental AOT compiler
- Focusing on other languages apart from Java (i.e. ECMAScript).

## Java 11 - The first new LTS version!

### HTTP2 is officially out of the incubator
It's official

### Var for lambdas
We can now use `var` in lambdas now

```java
(String s1, String s2) -> s1 + s2
```

is the same as:
```java
(s1, s2) -> s1 + s2
```

Is now the same as:
```java
(var s1, var  s2) -> s1 + s2
```

Why bother withy var? We can now do things like this, which we couldn't before (without explicitly declaring the types):
```java
(@Nonnull var s1, @Nullable var s2) -> s1 + s2
```

### Emojis4J2
There are 56 more emojis!
You can now display the bitcoin symbol.

### Launching Single-File programs
You can now run single-file java programs without compiling them. Any arguments placed after the file name are treated as arguments for the class, e.g.
```java
java Factorial.java 3 4 5
```

### Epsilon, The Garbage Collector for AWS Lambda and Google Cloud Functions?
A new GC has been added! ... Which doesn't collect any garbage.

It handles memory allocation but does not implement any actual memory reclamation mechanism. Once the heap is exhausted, the JVM will shut down.

The main usages of this will be for testing and (extremly) short-lived jobs.


### Official goodbye to JavaEE and CORBA
CORBA = Common Object Rerquest Broker Architecture. It enables communiocation between software written in diffgernet langauge sand running on different computers.

No one uses it any more.

Goobye, Java EE.

### A non-blocking HTTP Client
There is now an option to send and receive information over HTTP in a non-blocking manner.

#### A new experimental Garbage Collector - ZGC
ZGC is marketed as "a scalable low-latency garbage collector".
The aim is to drastically reduce the length of GC pauses, and to more effectively use larger amounts of memory in an efficient manner.

### JavaScript even Oracle can't keep Up.
Well, Oracle, you tried.

They officially give up on Nashorn and trying to keep up with the "rapid pace at which ECMAScript languages consutrcts, along with APIs are adapted and modified".

## Java 12

### ANOTHER new garbage collector; Shenadoah
Reduces GC pause times by doing evacuation work concurrently with the running Java threads. Pause times with Shanandoah are idependent of heap size, meaning you will have the same consistent pause times whether your heap is 200MB or 200GB.

Shenandoak is appropriate for applications which value responsiveness and predictable short pauses over througput or memory footprint.

### Switch expressions, not just statements (A.K.A Kotlins when expressions)
Tjere are a few things which are annoying about switch statements:
- If you don't include a `break` after every case, you'll fall throuhg into the next case
- the scope of variables is odd - if you declare a variable in one case, it will be available in the other cases
- They're quite verbose and span many lines - not particularly easy on the yees.
- Often in switch statements you'll want to assighn a value to a variable. YOu can do this, but it's not great to look at.
Lets look at a switch statement which is used to assign a variable to the number of letters in an enum value:
```java
int numLetters;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        numLetters = 6;
        break;
    case TUESDAY:
        numLetters = 7;
        break;
    case THURSDAY:
    case SATURDAY:
        numLetters = 8;
        break;
    case WEDNESDAY:
        numLetters = 9;
        break;
    default:
        throw IllegalStateException("Wat: " + day);
}
```
What this JEP is proposing is to be able to write that logic like this:
```java
int numLetters = switch(day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```

Note that this is a preview, and is subject to change.

### More Optimisations to Memory Management
More improvements to:
- return unused memory faster from garbage
- genenrate a class data-sharing archive to improve out-of-the-box startup time.

## Java 13

### MORE Optimisations to Memory Management
- Enhance ZGC to return unused heap memory to the operating system
- More improvements to class data-sharing archive.

### Rebuilding the Socket API for the first time since... Java 1
The current implementation is written partly in Java and partly in C. As you can image this isn't fun to extend or debug. There are also a number of several concurrency issues which can't easily be solved with the current implementation.

This will also not work with fibers (discussed later).

### More improvements to switch expressions (but still just a preview)
Default was added to the case -> labels syntax:
```java
static void howMany(int k) {
    switch(k) {
        case 1 -> System.out.println("one");
        case 2 -> System.out.println("two");
        default -> System.out.println("many");
    }
}
```

This could also be written like this:
```java
static void howMany(int k) {
    System.out.println(
        switch (k) {
            case 1 -> "one"
            case 2 -> "two"
            default -> "many"
        }
    );
}
```

And of course, you can still use it to assign a variable. Like we showed in the previous example with numLetters.

Yield was also added to return a result when a full block is needed:
```java
int j = switch (day) {
    case MONDAY     -> 0;
    case TURSDAY    -> 1;
    default         -> {
        int k = day.toString().length();
        int result = f(k);
        yield result;
    }
};
```

You can also use yield to use switch expressions in a more switch statmenet-esque manner (but with no fall-through):
```java
int result = switch (s) {
    case "Foo":
        yield 1;
    case "Bar":
        yield 2;
    default:
        System.out.println("Neither Foo nor Bar, hmmm...");
        yield 0;
}
```

They have now said that switch expressions must be exhaustive, meaning for all possible values there must ber a matching switch label (which doesn't have to be the case for switch statements).

### Finally, multiline strings (well, a preview of them)! But still no string interpolation :(
Text blocks, as they're called, is a multi-line string literal that avoid the need for most escape sequences, automatically formats string in a predictable way, and gives the developer control over format when desired.
```java
String html = "<html>\n" +
              "     <body>\n" +
              "         <p>Hello World</p>\n" +
              "     </body>\n" +
              "<html>\n";

String html = """
                <html>
                    <body>
                        <p>Hello, world!</p>
                    </body>
                </html>
            """;
```

## Java 14
**Note:** This is still in development so very likely to lose/gain features!

### Streaming Java Flight Recorder Data
Java Flight Recorder is a monitoring tool that collects information about the events in a JVM during the execution of an application.
Currently the way this works is that JFK saves data about the events in a single output file, flight.jfr. You can then use tools to analyse these files, such as Java Mission Control (which comes bundled with Java).

Now the problem with this is that to consume data a user must start recording, stop it, dump the contents to disk and then parse the recording file. This works well for application profiling, where typically at least a munute of data is being recorded a t time, but not for monitoring purpose.s

This JEPP aims to provide an API for the continuous consumption of JFK data on disk.

### A new Ground-breaking feature: Null pointer excpetions will now tell you which variable was null
"It is not a goal to track down the ultimate producer of a null reference, only the unlucky consumer."

Right now, if you try to do `a.i = 99` and `a` is null, you'll get an exception like this:
```
Exception in thread "main" java.lang.NullPointerException
    at Prog.main(Prog.java:5)
```
This isn't too bad, you can see that the problem happend on line 5, so you can deduce that `a` was null.  
However what if you were trying to do `a.b.c.i = 99;`? Which variable is null?  
Now you'll get a much more useful exception:
```
Exception in thread "main" java.lang.NullPointerException:
        Cannot read field "c" because "a.b." is null
    at Prog.main(Prog.java:5)
```

## Ongoing Projects
A project is a collaborative effort to prodice a specific artifact, which may be a body of code, or documentation, or some other material. A project must be sponsored by one or more groups. A project may have web contnet, one or more file repositories, and one or more mailing lists.

### Project Amber
The goal of Project Amber is to explore and incubate smaller, productivity-oriented Java language features that have been accepted as candidate JEPs.

#### Pattern Matching for instanceof (A useful feature that Kotlin already has)
Often you'll want to tell the type of an object, and then perform some kind of operation to that object from a specific API. For example, you might have an Animal Interface, and a Dog class. The dog class has a method, called `wagTail()` that animal does not have. If your object's pointer type is animal, you'll need to cast it Dog to be able to call `wagTail()`.
- I'm not saying this is a GOOD idea, just as an example

Right now to do this you'd have to do something like this:
```java
if (animal instanceof Dog) {
    Dog dog = (Dog) animal;
    dopg.wagTail();
}
```
(This is known as the instanceof-and-cast idiom).

What this JEP aims to do is provide some pattern matching so Java can implicitly cast the object for you:
```java
if(animal instanceof Dog dog ) {
    // Animal has been cast to dog as dog variable
    dog.wagtail();
} else {
    // animal won't be cast to dog
}
```

This will play a big part in switch expressions as well, which I'm assuming it will owrk simialr to Kotlin whne expression:
```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

Kotlin will implicitly cast x to Int, String, or IntArray. Kotlin calls them smart casts.

### Project Graal
Probably one of the most exciting things to come out of the JDK for a long time.
Graal has two parts:
- Graal, the compiler: This is the JIT compiler we talked about earlier
- GraalVM, the VM: The goal is to replace theHotSpot JVM entirely, using the JIT compiler, as well as be a new polyglot virtual machine that can support a large set of languages, including Java, JavaScript, Ruby, Python, C, etc.). Don't need nashorn  any more!

We cna also use the Graal compiler in the Ahead-of-Time mode, to compile our projects down to native code.

### Project Loom
The goal of this project is to explore fibers- lightweight user-mode threads. This is in competition with GoLang, which prides itself on having a much better concurrency model than Java with its coroutines.

Fibers are use-mode lightweight threads that allow synchronous (blocking) code to be efficiently scheduled, so that it performs as well as asnychronous code, but is simpler to read and write, debug, monitor, and profile. 

### Project Sumatra
Enable Java applications to take advatnage of heterogenous processing units (GPUs/APUs).

### Project Valhalla
This is an exciting project!
The goals of this project are to explore:
- inline types
- generics specialisation
- possibly reified generics

#### Inline Types
"code like a class, work like an int".  
This means that they should be a composite data type (cdoe like a class) but lack identity and not pay the object header cost if at all possible (work like an int).  
It would be similar to a C struct.

From the very first verisions of Java until the present day, Java has had only two types of vlaues: primitive types and object references. This model is extremely simple and easy for developers to understand, but can have performance trade-offs. For example, dealing with arrays of objects ivolves unavoidable indirections and this can result in processor cache misses.

The goal of inline classes is to improve the affinity of Java programs to modern hardwre. This is to be achieved by revisiting a very fundamental part of the Java platform - the model of Java's data values.

#### Generic Specialisation
When we want to generify over language primitives, we currently used boxed types, such as Integer for int or Float for float. This boxing creates an additional layer of indirection, thereby defeating the purpose of using primitives for performance enhancement in the first place.

Therefore, we see many dedicated specialisations for primitive types in existing frameworks and libraries, like IntStream<T> or ToIntFunction<T>. This is done to keep the performance improvement of using primitives.

So, specialisation generics is an effort to remove the needs for those "hacks". Instead, the Javalanguage strives to enable generic types for basically everything: object references, primitives, value types, and maybe even void.

#### Reified Generics
Java suffers from Type Erasure - when we use generics, java knows what the generic type is at compile time, but not at run time.

If you're unsure about type erasure, lets look at an example. say we have this code:
```java
public static <E> boolean containsElement(E[] elements, E element) {
    for (E e: elements) {
        if(e.equals(element)) {
            return return
        }
    }
    return false;
}
```
Once compiled, the unbound type E will get replaced with a type of `Object`
```java
public static boolean containsElement(Object[] elements, Object element) {
    for (Object e: elements) {
        if(e.equals(element)) {
            return return
        }
    }
    return false;
}
```

This can cause problems, elts look at an example of overloading a method:
```java
public class printer {
    public void print(List<String> words) {}
    public void print(List<Integer> numbers) {}
}
```
This won't work, you'll get the compilation error:
```
name clash: print(java.util.List<java.lang.Integer>) and print(java.util.List<java.lang.String>) have the same erasure
```
This is because they'll both compile down to:
```java
public void print(List<Object> words) {}
```
There are other problems due to type erasure.  
There are some ways to get around type erasure, but its annoying and messy.

Reified generics would mean that the compiler would preserve the type of a generic, and so we wouldn't have these problems!