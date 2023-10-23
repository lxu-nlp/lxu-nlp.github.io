---
layout: post
title: "Dropwizard Example with HK2 and Swagger"
date: 2018-02-13
categories: [Programming]
tags: [java, dropwizard, legacy]
---

# What is Dropwizard?

“Dropwizard is a Java framework for developing ops-friendly, high-performance, RESTful web services.” It is pretty much like Spring Boot, with a lot of production-ready features.

Official Website: <https://www.dropwizard.io/>

# What is HK2?

Just like Spring and Guice, HK2 is a Dependency Injection (DI) framework that complies with JSR-330, and is integrated with Jersey 2.0. While HK2 is not as popular as other DI frameworks, it is by default available out-of-the-box in Dropwizard.

Introduction: <https://javaee.github.io/hk2/>

### How to use HK2 in Dropwizard?

We need to define an AbstractBinder, within which defines how HK2 should inject classes, and register it to the Jersey application.

### HK2 Dropwizard Integration Bundle

Sometimes working with HK2 is a little bit painful in certain use cases, and I will give a following example in this article. To ease the pain, we can add a HK2-Dropwizard Integration bundle.

Bundle GitHub: <https://github.com/baharclerode/dropwizard-hk2>

# What is Swagger?

Swagger is a powerful tool for API specifications. A popular use case is to auto-generate a web-client by annotating resources classes and methods.

Official Website: <https://swagger.io/>

### Swagger Dropwizard Integration Bundle

More than one open-source bundles available. A popular one:

<https://github.com/smoketurner/dropwizard-swagger>

# Sample Dropwizard Project with HK2 and Swagger

Initial project is planted from Dropwizard Maven Archetype. Full project available on my GitHub: <https://github.com/lxucs/Tryout-Swagger>

Add Swagger bundle.

```java
@Override
    public void initialize(final Bootstrap<TryoutSwaggerConfiguration> bootstrap) {
 
        // Add Swagger Bundle
        bootstrap.addBundle(new SwaggerBundle<TryoutSwaggerConfiguration>() {
            @Override
            protected SwaggerBundleConfiguration getSwaggerBundleConfiguration(TryoutSwaggerConfiguration config) {
                return config.getSwaggerBundleConfiguration();
            }
        });
 
        // Customize global JSON serialization settings
        bootstrap.getObjectMapper().disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    }
```

Define API and Service implementation.

```java
public class TryoutSwaggerApiImpl implements TryoutSwaggerApi {

    private final AtomicInteger counter;

    public TryoutSwaggerApiImpl() {
        counter = new AtomicInteger(0);
    }

    @Override
    public User getUserByName(@NotNull String name) {
        counter.incrementAndGet();

        User user = User.builder()
                .uuid(UUID.randomUUID())
                .name(name)
                .age(ThreadLocalRandom.current().nextInt(100))
                .lastUpdate(Instant.now())
                .build();
        return user;
    }

    @Override
    public int getHitCount() {
        return counter.get();
    }

}
```

To use HK2, define AbstractBinder and register to Jersey. Inside AbstractBinder, we bind the service with its interface, so that HK2 will know to inject the service class.

```java
@Override
    public void run(final TryoutSwaggerConfiguration configuration,
                    final Environment env) {

        JerseyEnvironment jersey = env.jersey();

        jersey.register(new AbstractBinder() {
            @Override
            protected void configure() {
                bind(TryoutSwaggerApiImpl.class).to(TryoutSwaggerApi.class).in(Singleton.class);
            }
        });

        jersey.register(TryoutSwaggerResource.class);
    }
```

Define Resource class. Use JSR-330 @Inject annotation to enable HK2 injection, and Swagger annotation to enable Swagger generation.

```java
@Path("/user")
@Consumes(MediaType.APPLICATION_JSON)
@Produces(MediaType.APPLICATION_JSON)
@Api(description = "Tryout Swagger", protocols = "http")
public class TryoutSwaggerResource {

    private final TryoutSwaggerApi api;

    @Inject
    public TryoutSwaggerResource(@NotNull TryoutSwaggerApi api) {
        this.api = api;
    }

    @GET
    @Path("/alive")
    public String testAlive() {
        return "yes";
    }

    @GET
    @Path("/detail/{name}")
    @ApiOperation(value = "Returns the user detail by name.")
    public User getUserByName(@PathParam("name") String name) {
        return api.getUserByName(name);     //Dropwizard will handle response
    }

}
```

Add Swagger configuration in Yaml.

```text
swagger:
  resourcePackage: org.liyanxu.tryoutswagger.resources
```

Run the application with parameters “server config.yml“. The Swagger bundle will auto-generate a client based on the resourcePackage.

Go to <https://localhost:8080/swagger>

![Swagger-1](/assets/img/legacy/swagger-ui.png)

Click the first GET API and try to send requests.

![Swagger-2](/assets/img/legacy/swagger-tryout.png)

We can see the request and response on the UI page.

## Use bundle to Inject Managed etc. by HK2

In the above example, TryoutSwaggerApi is injected by HK2 in the resource class, without manually being created. This is nice. However, there are some common use cases that need already-created instances, such as Dropwizard Managed, or HealthCheck.

See this issue that represents the common use cases where HK2 is hard to work with: [Using jersey’s HK2 to inject healthchecks, lifecycle Manageds](https://github.com/dropwizard/dropwizard/issues/862)

We can also have an example here. Let’s define a lifecycle-managed class, which spits out API hit count when the application starts and stops. It takes the API as its constructor parameter.

```java
@Slf4j
public class ApiInfoManaged implements Managed {

    private final TryoutSwaggerApi api;

    @Inject
    public ApiInfoManaged(TryoutSwaggerApi api) {
        this.api = api;
    }

    @Override
    public void start() throws Exception {
        log.info("Start: api hit count is " + api.getHitCount());
    }

    @Override
    public void stop() throws Exception {
        log.info("Stop: api hit count is " + api.getHitCount());
    }
}
```

We want the Dropwizard managing this class by passing an instance to the Dropwizard environment lifecycle. However, since its constructor parameter, which is TryoutSwaggerApi, is injected by HK2, we don’t have a reference to the TryoutSwaggerApi instance. Hence, we cannot create an instance of ApiInfoManaged. There are also two points that need to be noticed:

* We cannot use the serviceLocator to create the instance, because the serviceLocator hasn’t been initialized in the run() method.
* We can, however, manipulate the serviceLocator by adding an applicationEventListener or passing our own serviceLocator instance. But that requires more work.
Luckily, an engineer created the convenient HK2-Dropwizard Integration bundle; by using the bundle, we can just register the ApiInfoManaged class to jersey, and it will be created by HK2 injection, and managed by Dropwizard lifecycle.

Add HK2 Integration bundle.

```java
@Override
    public void initialize(final Bootstrap<TryoutSwaggerConfiguration> bootstrap) {

        // This ensures there's only ever one HK2 bundle; don't use bootstrap.addBundle(new HK2Bundle<>());
        HK2Bundle.addTo(bootstrap);

        // Add Swagger Bundle
        bootstrap.addBundle(new SwaggerBundle<TryoutSwaggerConfiguration>() {
```

Change Managed interface from Dropwizard to InjectableManaged interface from the bundle .

```java
@Slf4j
public class ApiInfoManaged implements InjectableManaged {

    private final TryoutSwaggerApi api;
```

Register ApiInfoManaged to Jersey.

```java
jersey.register(ApiInfoManaged.class);  // Create Managed class by DI
/*
Instead of:
env.lifecycle().manage(new ApiInfoManaged(...));
*/
```

Run the application and hit the endpoint twice. Stop the application.

See the console output:

```text
...ApiInfoManaged: Start: api hit count is 0
......
...ApiInfoManaged: Stop: api hit count is 2
```

Great! The ApiInfoManaged is working!
