# Dubbo Spring Boot Project 

[![Build Status](https://travis-ci.org/apache/incubator-dubbo-spring-boot-project.svg?branch=master)](https://travis-ci.org/apache/incubator-dubbo-spring-boot-project) 
[![codecov](https://codecov.io/gh/apache/incubator-dubbo-spring-boot-project/branch/master/graph/badge.svg)](https://codecov.io/gh/apache/incubator-dubbo-spring-boot-project)
[![Gitter](https://badges.gitter.im/alibaba/dubbo.svg)](https://gitter.im/alibaba/dubbo?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)
![license](https://img.shields.io/github/license/apache/incubator-dubbo-spring-boot-project.svg)
![maven](https://img.shields.io/maven-central/v/com.alibaba.boot/dubbo-spring-boot-starter.svg)

[Apache Dubbo(incubating)](https://github.com/apache/incubator-dubbo) Spring Boot Project makes it easy to create [Spring Boot](https://github.com/spring-projects/spring-boot/) application using Dubbo as RPC Framework. What's more, it aslo provides 

* [auto-configure features](dubbo-spring-boot-autoconfigure) (e.g., annotation-driven, auto configuration, externalized configuration).
* [production-ready features](dubbo-spring-boot-actuator) (e.g., security, health checks, externalized configuration).

> Apache Dubbo(incubating) is a high-performance, java based [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call) framework open-sourced by Alibaba. As in many RPC systems, dubbo is based around the idea of defining a service, specifying the methods that can be called remotely with their parameters and return types. On the server side, the server implements this interface and runs a dubbo server to handle client calls. On the client side, the client has a stub that provides the same methods as the server.

## [中文说明](README_CN.md)


## Released version

You can introduce the latest `dubbo-spring-boot-starter` to your project by adding the following dependency to your pom.xml
```xml
<properties>
    <spring-boot.version>2.1.1.RELEASE</spring-boot.version>
    <dubbo.version>2.6.5</dubbo.version>
</properties>
    
<dependencyManagement>
    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    
        <!-- Dubbo dependencies -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo-dependencies-bom</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Dubbo Spring Boot Starter -->
    <dependency>
        <groupId>com.alibaba.boot</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>0.2.1.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>${dubbo.version}</version>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
    </dependency>
</dependencies>
```

If your project failed to resolve the dependency, try to add the following repository:
```xml
<repositories>
    <repository>
        <id>sonatype-nexus-snapshots</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```


## Developing Versions

For now, `dubbo-spring-boot-starter` will separate two versions for Spring Boot 2.x and 1.x once release : 

* [`0.2.x`](https://github.com/apache/incubator-dubbo-spring-boot-project) is a main stream release version for Spring Boot 2.x

* [`0.1.x`](https://github.com/apache/incubator-dubbo-spring-boot-project/tree/0.1.x) is a legacy version for maintaining Spring Boot 1.x


### Build from Source

If you'd like to attempt to experience latest features, you also can build from source as follow:

1. Maven install current project in your local repository.
> Maven install = `mvn install`


### Dependencies

| versions | Java  | Spring Boot | Dubbo      |
| -------- | ----- | ----------- | ---------- |
| `0.2.1`  | 1.8+ | `2.1.x` | `2.6.5` + |
| `0.1.1`  | 1.7+ | `1.5.x` | `2.6.5` + |



## Getting Started

If you don't know about Dubbo, please take a few minutes to learn http://dubbo.apache.org/. After that  you could dive deep into dubbo [user guide](http://dubbo.apache.org/en-us/docs/user/quick-start.html).

Usually, There are two usage scenarios for Dubbo applications, one is Dubbo service(s) provider, another is Dubbo service(s) consumer, thus let's get a quick start on them.

First of all, we suppose an interface as Dubbo RPC API that  a service provider exports and a service client consumes: 

```java
public interface DemoService {

    String sayHello(String name);

}
```



### Dubbo service(s) provider

1. Service Provider implements `DemoService`

    ```java
    @Service(version = "1.0.0")
    public class DefaultDemoService implements DemoService {

        /**
        * The default value of ${dubbo.application.name} is ${spring.application.name}
        */
        @Value("${dubbo.application.name}")
        private String serviceName;

        public String sayHello(String name) {
            return String.format("[%s] : Hello, %s", serviceName, name);
        }
    }
    ```

2. Provides a bootstrap class

    ```java
    @EnableAutoConfiguration
    public class DubboProviderDemo {

        public static void main(String[] args) {
            SpringApplication.run(DubboProviderDemo.class,args);
        }
    }
    ```

3. Configures the `application.properties`:

    ```properties
    # Spring boot application
    spring.application.name=dubbo-auto-configuration-provider-demo

    # Base packages to scan Dubbo Component: @com.alibaba.dubbo.config.annotation.Service
    dubbo.scan.base-packages=com.alibaba.boot.dubbo.demo.provider.service

    # Dubbo Application
    ## The default value of dubbo.application.name is ${spring.application.name}
    ## dubbo.application.name=${spring.application.name}

    # Dubbo Protocol
    dubbo.protocol.name=dubbo
    dubbo.protocol.port=12345

    ## Dubbo Registry
    dubbo.registry.address=N/A
    ```



### Dubbo service(s) consumer

1. Service consumer also provides a bootstrap class to reference `DemoService`

    ```java
    @EnableAutoConfiguration
    public class DubboConsumerBootstrap {

        private final Logger logger = LoggerFactory.getLogger(getClass());

        @Reference(version = "1.0.0", url = "dubbo://localhost:12345")
        private DemoService demoService;

        @Bean
        public ApplicationRunner runner() {
            return args -> {
                logger.info(demoService.sayHello("mercyblitz"));
            };
        }

        public static void main(String[] args) {
            SpringApplication.run(DubboConsumerBootstrap.class).close();
        }
    }
    ```

2. configures `application.properties`

    ```properties
    # Spring boot application
    spring.application.name = dubbo-consumer-demo
    server.port = 8080
    management.port = 8081


    # Dubbo Config properties
    ## ApplicationConfig Bean
    dubbo.application.id = dubbo-consumer-demo
    dubbo.application.name = dubbo-consumer-demo

    ## ProtocolConfig Bean
    dubbo.protocol.id = dubbo
    dubbo.protocol.name = dubbo
    dubbo.protocol.port = 12345
    ```

If `DubboProviderDemo` works well, please mark sure `DubboProviderDemo` is started.

More details, please refer to [Samples](dubbo-spring-boot-samples).



## Getting help

Having trouble with Dubbo Spring Boot? We’d like to help!

- If you are upgrading, read the [release notes](https://github.com/apache/incubator-dubbo-spring-boot-project/releases) for upgrade instructions and "new and noteworthy" features.
- Ask a question - You can subscribe [Dubbo User Mailling List](mailto:dev-subscribe@dubbo.apache.org).
- Report bugs at [issues](https://github.com/apache/incubator-dubbo-spring-boot-project/issues).




## Building from Source

If you want to try out thr latest features of Dubbo Spring Boot, it can be easily built with the [maven wrapper](https://github.com/takari/maven-wrapper). Your JDK is 1.8 or above.

```
$ ./mvnw clean install
```



## Modules

There are some modules in Dubbo Spring Boot Project, let's take a look at below overview:



### [dubbo-spring-boot-parent](dubbo-spring-boot-parent)

The main usage of `dubbo-spring-boot-parent` is providing dependencies management for other modules.



### [dubbo-spring-boot-autoconfigure](dubbo-spring-boot-autoconfigure)

`dubbo-spring-boot-autoconfigure` uses Spring Boot's `@EnableAutoConfiguration` which helps core Dubbo's components to be auto-configured by `DubboAutoConfiguration`. It reduces code, eliminates XML configuration. 



### [dubbo-spring-boot-actuator](dubbo-spring-boot-actuator)

`dubbo-spring-boot-actuator` provides production-ready features (e.g., [health checks](https://github.com/dubbo/dubbo-spring-boot-project/tree/master/dubbo-spring-boot-actuator#health-checks), [endpoints](https://github.com/dubbo/dubbo-spring-boot-project/tree/master/dubbo-spring-boot-actuator#endpoints), and [externalized configuration](https://github.com/dubbo/dubbo-spring-boot-project/tree/master/dubbo-spring-boot-actuator#externalized-configuration)).



### [dubbo-spring-boot-starter](dubbo-spring-boot-starter)

`dubbo-spring-boot-starter` is a standard Spring Boot Starter, which contains [dubbo-spring-boot-autoconfigure](dubbo-spring-boot-autoconfigure) and [dubbo-spring-boot-actuator](dubbo-spring-boot-actuator). It will be imported into your application directly.



### [dubbo-spring-boot-samples](dubbo-spring-boot-samples)

The samples project of Dubbo Spring Boot that includes:

- [Auto-Configuaration Samples](dubbo-spring-boot-samples/auto-configure-samples)
- [Externalized Configuration Samples](dubbo-spring-boot-samples/externalized-configuration-samples)
- [Registry Zookeeper Samples](dubbo-spring-boot-samples/dubbo-registry-zookeeper-samples)
- [Registry Nacos Samples](dubbo-spring-boot-samples/dubbo-registry-nacos-samples)
- [Sample API](dubbo-spring-boot-samples/sample-api)
