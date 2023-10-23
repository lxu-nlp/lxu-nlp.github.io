---
layout: post
title: "Java Thread-Safe Problems: Safe Publication and Visibility"
date: 2018-05-19
categories: [Programming]
tags: [java, thread]
---

As Software Engineer, a lot of times we need to write classes that are multithread-safe. Even though thread-safe is not a hard requirement, it is often preferred to also write code in a thread-safe manner. Therefore, the abilities to follow thread-safe practices and detect potential problems should be well aware of by software engineers.

The initiative to write this article is that I recently submitted some code that could cause potential threading problems, and my teammate got me when doing code review. This is a basic summary of my one-day study towards Java threading problems. There are far more content to be learned about Java Memory Model and it is a complicated problem by itself. In this article, we are going to focus on two basic things: Object Publication and Variable Visibility, and their best practices. The root of both problems is multiple threads interacting with the same shared variable.

# Object Publication

### Problem

Let's look at the following code, and identify the problem.

First, we have an immutable class Sample that is thread-safe itself.

```java
@Data
class Sample {

    private final int a;
    private final int b;

    public Sample(int a, int b) {
        this.a = a;
        this.b = b;
    }
}
```

Then we have a class Publisher that can publish this Sample class.

```java
public class Publisher {

    private Sample sample = null;

    public void initSample(int a, int b) {
        sample = new Sample(a, b);
    }

    public Sample getSample() {
        return sample;
    }
}
```

What is the problem here?

When there are two threads T1 and T2, T1 is calling `initSample(1, 2)`, and T2 is calling `getSample()`, T2 can potentially get a partially initialized object! Such as, `sample.getA() == 1`, `sample.getB() == 0` (Default value). This is because compiler can rearrange the statement order that the write to the sample field can happen before the full execution of the sample's constructor.

Safe Object Publication is about avoiding publishing a partially initialized object.

### Solution 1: Use Synchronization

The simplest way is to use synchronization, which limits all interaction with this sample field to one thread at a time. In this case, if one thread is calling `initSample()`, no other threads can call either `initSample()` or `getSample()` unless the first thread finishes the sample initialization and releases the lock. Therefore, we won't get a partially initialized object.

However, the downside is that this may not be always desired, since locking can cause performance and deadlock issues if not used appropriately.

```java
public class Publisher {

    private Sample sample = null;
    private final Object lock = new Object();

    public void initSample(int a, int b) {
        synchronized (lock) {
            sample = new Sample(a, b);
        }
    }

    public Sample getSample() {
        synchronized (lock) {
            return sample;
        }
    }
}
```

### Solution 2: Move Initialization inside Constructor + Use Final Field

According to Java spec, "a thread that can only see a reference to an object after that object has been completely initialized is guaranteed to see the correctly initialized values for that object's `final` fields".

By making the sample field `final`, we can guarantee that after the construction of Publisher, any threads calling `getSample()` will get a fully initialized object.

However, the downsides are that:
* The sample field can only be initialized once.
* The Publisher itself should remain unpublished before the end of construction, otherwise the sample field can have uninitialized value, if a thread calls `getSample()` before the Sample constructor statement gets executed.

```java
public class Publisher {

    private final Sample sample;

    public Publisher() {
        sample = new Sample(1, 2);
    }

    public Sample getSample() {
        return sample;
    }
}
```

### Solution 3: Use Volatile Field

During multithreading, JVM may utilize the local cache of each cores to improve the performance. To keep the local cache consistent on certain fields, JVM guarantees that any local value change of `volatile` fields will be pushed to main memory and distributed to all other caches. Therefore, `volatile` guarantees the visibility across multiple threads.

### Solution 4: Use Factory Pattern without Shared Variable

It is thread-safe without the use of shared variable, because the object initialization is local to each thread. It is a common pattern and we should use it if it fits the requirement.

The downside is that it will create a new object each time, instead of just returning a reference of an existing object, which may not be always desired.

```java
public class Publisher {

    public Sample getSample(int a, int b) {
        return new Sample(a, b);
    }
}
```

### Solution 5: Use Singleton Pattern with Shared Static Final Variable

Singleton pattern is thread-safe because each time it only returns the same reference. It may or may not be desired depending on the requirements.

Here we use an inner static class to have the property of lazy initialization.

```java
public class Publisher {

    public static class singletonUtil {
        public static final Sample instance = new Sample(1, 2);
    }

    public Sample getSample() {
        return singletonUtil.instance;
    }
}
```
