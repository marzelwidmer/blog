---
title: Spring Boot Kubernetes Discovery
subTitle: Discovery Spring Boot in Kubernetes
date: "2019-09-18"
draft: false
tags: [k8s, Spring Boot]
categories: [k8s]
ShowToc: true
TocOpen: true
---
 
[Spring Cloud Kubernetes Ribbon](https://cloud.spring.io/spring-cloud-static/spring-cloud-kubernetes/1.1.0.M2/reference/html/#_ribbon_discovery_in_kubernetes) 
provide a mechanism to perform a client side load-balancing who is needed in a microservice architecture 
to allocate a list of all pods where our service is running (replicated)

This mechanism can automatically discover and reach all the endpoints of a specific service, and subsequently, 
it populates a Ribbon ServerList with information about the endpoints.

Let's start by adding the `spring-cloud-starter-kubernetes-ribbon` dependency to our `pom.xml` file:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-kubernetes-ribbon</artifactId>
</dependency>
```

When the list of the endpoints is populated, the K8s client will search the registered endpoints 
living in the current `namespace`

We also need to enable the ribbon client in the `application.yaml`:

```yaml
ribbon.http.client.enabled=true
```

Now add the `@EnableDiscoveryClient` annotation in the Spring Boot application.

```kotlin
@EnableDiscoveryClient
@SpringBootApplication
class CatalogServiceClient
```

As next we also add the `@LoadBalanced` annotation. 
```kotlin
class OrderServiceConfiguration {
 
     @Bean
     @LoadBalanced
     fun webClientBuilder() = WebClient.builder()
}
```


Now we can use now the service name `http://catalog-service` to call other services.

```kotlin
@Service
class CatalogServiceClient(private val webClientBuilder: WebClient.Builder) {

    companion object {
        val CATALOG_SERVICE_URL = "http://catalog-service/api/v1/animals/random"
    }

    fun getRandomAnimalNames(): Flux<String> {
        return this.webClientBuilder
                .baseUrl(CATALOG_SERVICE_URL).build()
                .get()
                .accept(MediaType.APPLICATION_JSON)
                .retrieve().bodyToFlux(String::class.java)
                .log()
    }
}

```
