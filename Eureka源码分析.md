# Eureka源码分析

## Eureka Client

`@EnableDiscoveryClient`注解中有一个`autoRegister()`方法

```java
@Import({EnableDiscoveryClientImportSelector.class})
public @interface EnableDiscoveryClient {
    boolean autoRegister() default true;
}
```

我们可以看到`DiscoveryClient`实现了`EurekaClient`

```java
@Singleton
public class DiscoveryClient implements EurekaClient{
    //...
}
```

`@EnableDiscoveryClient`继承了`@EnableEurekaClient`

所以两者都可以实现服务注册

但是`DiscoveryClient`是真正实现服务注册的类

在`DiscoveryClient`中有一个过期了的方法`getServiceUrlsFromConfig`，它会调用`clientConfig.getAvailabilityZones`，获取配置文件中的`Eureka Server`的url

如果`eureka.client.service-url`的`zone`为空，会取默认的zone，也就是`eureka.client.service-url.defaultZone`

```java
//public static final String DEFAULT_URL = "http://localhost:8761" + DEFAULT_PREFIX + "/";
//public static final String DEFAULT_ZONE = "defaultZone";
public String[] getAvailabilityZones(String region) {
		String value = this.availabilityZones.get(region);
		if (value == null) {
			value = DEFAULT_ZONE;
		}
		return value.split(",");
	}
```

如果连`eureka.client.service-url`都没有配置

默认就是`http://localhost:8761/eureka/`

在`DiscoveryClient`构造方法被调用的时候

会调用一个`initScheduledTasks()`方法

该方法先判断是否需要去`eureka server`拉取注册信息

```java
if (this.clientConfig.shouldFetchRegistry()) {
//...
}
```

然后判断是否需要注册自身信息

```JAVA
if (this.clientConfig.shouldRegisterWithEureka()) {
//...
}
```

（因为有些服务可能只调用其他服务，或者只提供服务，所以需要这两个判断）

如果需要注册自身信息，会启动一个线程`InstanceInfoReplicator`

```JAVA
this.instanceInfoReplicator.start(this.clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
```

在这个线程的`run()`方法中会调用`this.discoveryClient.register()`

```java
/**
 * Register with the eureka service by making the appropriate REST call.
 */
boolean register() throws Throwable {
    logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
    EurekaHttpResponse<Void> httpResponse;
    try {
        httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
    } catch (Exception e) {
        logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
        throw e;
    }
    if (logger.isInfoEnabled()) {
        logger.info(PREFIX + "{} - registration status: {}", appPathIdentifier, httpResponse.getStatusCode());
    }
    return httpResponse.getStatusCode() == 204;
}
```

通过注释可以发现，注册到`Eureka Server`的方式是通过`RESTful`的`HTTP`请求