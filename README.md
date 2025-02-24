I understand that you’re configuring a WebClient in your Spring Boot application to use a PEM-encoded truststore for SSL/TLS communication, and you don’t have a specific base URL to set during its initialization. Here’s how you can achieve this:

1. Ensure You’re Using Spring Boot 3.1 or Later

The SSL bundle feature, which simplifies SSL/TLS configurations using PEM files, was introduced in Spring Boot 3.1. Verify your project’s build.gradle or pom.xml to ensure you’re using the correct version.

2. Place Your PEM Certificate in the Classpath

Save your PEM-encoded certificate (e.g., server-cert.pem) in the src/main/resources directory of your project. This ensures it’s accessible on the classpath.

3. Configure the SSL Bundle in application.yml or application.properties

Define an SSL bundle that references your PEM certificate. In application.yml:

spring:
  ssl:
    bundle:
      pem:
        mybundle:
          truststore:
            certificate: "classpath:server-cert.pem"

This configuration creates an SSL bundle named mybundle that uses your PEM certificate as the truststore.

4. Configure the WebClient Without a Base URL

If you don’t have a specific base URL, you can configure the WebClient without setting one. Here’s how you can set up a WebClient bean that utilizes the defined SSL bundle:

import org.springframework.boot.ssl.SslBundle;
import org.springframework.boot.ssl.SslBundles;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class WebClientConfig {

    @Bean
    public WebClient webClient(WebClient.Builder webClientBuilder, SslBundles sslBundles) {
        SslBundle sslBundle = sslBundles.getBundle("mybundle");
        return webClientBuilder
                .apply(sslBundle.sslContext())
                .build();
    }
}

In this setup:
	•	sslBundles.getBundle("mybundle"): Retrieves the SSL bundle configured earlier.
	•	apply(sslBundle.sslContext()): Applies the SSL context from the bundle to the WebClient.

By not calling .baseUrl(), the WebClient is created without a predefined base URL, allowing you to specify the full URL for each request dynamically.

5. Using the Configured WebClient

With the WebClient bean configured, you can inject it into your services and use it to make requests:

import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Mono;

@Service
public class ApiService {

    private final WebClient webClient;

    public ApiService(WebClient webClient) {
        this.webClient = webClient;
    }

    public Mono<String> fetchData(String url) {
        return webClient.get()
                .uri(url)
                .retrieve()
                .bodyToMono(String.class);
    }
}

In this example, the fetchData method accepts a url parameter, allowing you to specify the target URL for each request.

6. Additional Considerations
	•	Multiple Certificates: If you need to trust multiple certificates, you can concatenate them into a single PEM file or specify multiple certificates in your configuration.
	•	Reloading Certificates: If your certificates change and you need the application to recognize updates without restarting, consider enabling the reload-on-update property.

By following these steps, you can configure your WebClient to use a PEM-encoded truststore without specifying a base URL, allowing for dynamic URL handling in your application.

References:
	•	Securing Spring Boot Applications With SSL
	•	Spring Boot SSL Documentation