## RestClient

If you are not using Spring WebFlux or Project Reactor in your application we recommend that you use [`RestClient`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.html) to call remote REST services.

The [`RestClient`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.html) interface provides a functional style blocking API.

Spring Boot creates and pre-configures a prototype [`RestClient.Builder`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.Builder.html) bean for you. It is strongly advised to inject it in your components and use it to create [`RestClient`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.html) instances. Spring Boot is configuring that builder with [`HttpMessageConverters`](https://docs.spring.io/spring-boot/3.4.6-SNAPSHOT/api/java/org/springframework/boot/autoconfigure/http/HttpMessageConverters.html) and an appropriate [`ClientHttpRequestFactory`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/http/client/ClientHttpRequestFactory.html).

The following code shows a typical example:

- Java
    
- Kotlin
    

```java
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class MyService {

	private final RestClient restClient;

	public MyService(RestClient.Builder restClientBuilder) {
		this.restClient = restClientBuilder.baseUrl("https://example.org").build();
	}

	public Details someRestCall(String name) {
		return this.restClient.get().uri("/{name}/details", name).retrieve().body(Details.class);
	}

}
```

### [](https://docs.spring.io/spring-boot/3.4-SNAPSHOT/reference/io/rest-client.html#io.rest-client.restclient.customization)RestClient Customization

There are three main approaches to [`RestClient`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.html) customization, depending on how broadly you want the customizations to apply.

To make the scope of any customizations as narrow as possible, inject the auto-configured [`RestClient.Builder`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.Builder.html) and then call its methods as required. [`RestClient.Builder`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.Builder.html) instances are stateful: Any change on the builder is reflected in all clients subsequently created with it. If you want to create several clients with the same builder, you can also consider cloning the builder with `RestClient.Builder other = builder.clone();`.

To make an application-wide, additive customization to all [`RestClient.Builder`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.Builder.html) instances, you can declare [`RestClientCustomizer`](https://docs.spring.io/spring-boot/3.4.6-SNAPSHOT/api/java/org/springframework/boot/web/client/RestClientCustomizer.html) beans and change the [`RestClient.Builder`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.Builder.html) locally at the point of injection.

Finally, you can fall back to the original API and use `RestClient.create()`. In that case, no auto-configuration or [`RestClientCustomizer`](https://docs.spring.io/spring-boot/3.4.6-SNAPSHOT/api/java/org/springframework/boot/web/client/RestClientCustomizer.html) is applied.

|   |   |
|---|---|
||You can also change the [global HTTP client configuration](https://docs.spring.io/spring-boot/3.4-SNAPSHOT/reference/io/rest-client.html#io.rest-client.clienthttprequestfactory.configuration).|

### [](https://docs.spring.io/spring-boot/3.4-SNAPSHOT/reference/io/rest-client.html#io.rest-client.restclient.ssl)RestClient SSL Support

If you need custom SSL configuration on the [`ClientHttpRequestFactory`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/http/client/ClientHttpRequestFactory.html) used by the [`RestClient`](https://docs.spring.io/spring-framework/docs/6.2.x/javadoc-api/org/springframework/web/client/RestClient.html), you can inject a [`RestClientSsl`](https://docs.spring.io/spring-boot/3.4.6-SNAPSHOT/api/java/org/springframework/boot/autoconfigure/web/client/RestClientSsl.html) instance that can be used with the builder’s `apply` method.

The [`RestClientSsl`](https://docs.spring.io/spring-boot/3.4.6-SNAPSHOT/api/java/org/springframework/boot/autoconfigure/web/client/RestClientSsl.html) interface provides access to any [SSL bundles](https://docs.spring.io/spring-boot/3.4-SNAPSHOT/reference/features/ssl.html#features.ssl.bundles) that you have defined in your `application.properties` or `application.yaml` file.

The following code shows a typical example:

- Java
    
- Kotlin
    

```java
import org.springframework.boot.autoconfigure.web.client.RestClientSsl;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class MyService {

	private final RestClient restClient;

	public MyService(RestClient.Builder restClientBuilder, RestClientSsl ssl) {
		this.restClient = restClientBuilder.baseUrl("https://example.org").apply(ssl.fromBundle("mybundle")).build();
	}

	public Details someRestCall(String name) {
		return this.restClient.get().uri("/{name}/details", name).retrieve().body(Details.class);
	}

}
```

If you need to apply other customization in addition to an SSL bundle, you can use the [`ClientHttpRequestFactorySettings`](https://docs.spring.io/spring-boot/3.4.6-SNAPSHOT/api/java/org/springframework/boot/http/client/ClientHttpRequestFactorySettings.html) class with [`ClientHttpRequestFactoryBuilder`](https://docs.spring.io/spring-boot/3.4.6-SNAPSHOT/api/java/org/springframework/boot/http/client/ClientHttpRequestFactoryBuilder.html):

- Java
    
- Kotlin
    

```java
import java.time.Duration;

import org.springframework.boot.http.client.ClientHttpRequestFactoryBuilder;
import org.springframework.boot.http.client.ClientHttpRequestFactorySettings;
import org.springframework.boot.ssl.SslBundles;
import org.springframework.http.client.ClientHttpRequestFactory;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestClient;

@Service
public class MyService {

	private final RestClient restClient;

	public MyService(RestClient.Builder restClientBuilder, SslBundles sslBundles) {
		ClientHttpRequestFactorySettings settings = ClientHttpRequestFactorySettings
			.ofSslBundle(sslBundles.getBundle("mybundle"))
			.withReadTimeout(Duration.ofMinutes(2));
		ClientHttpRequestFactory requestFactory = ClientHttpRequestFactoryBuilder.detect().build(settings);
		this.restClient = restClientBuilder.baseUrl("https://example.org").requestFactory(requestFactory).build();
	}

	public Details someRestCall(String name) {
		return this.restClient.get().uri("/{name}/details", name).retrieve().body(Details.class);
	}

}
```