Based on your description—using the latest Spring Boot version and having your configuration in place—it sounds like the SSL bundle bean isn’t being created because Spring Boot’s auto‐configuration isn’t picking up any SSL bundle settings.

Here are a few things to check and try:
	1.	Ensure the SSL Bundle is Defined in Your Configuration
The auto‑configuration for SSL bundles will only create an SslBundles bean if you’ve defined at least one SSL bundle property. For example, in your application.yml, you should have something like:

spring:
  ssl:
    bundle:
      pem:
        mybundle:
          truststore:
            certificate: "classpath:server-cert.pem"

Make sure that:
	•	The bundle name (here, “mybundle”) exactly matches what you reference in your code (remember that names are case‑sensitive).
	•	The certificate file (e.g. server‑cert.pem) is actually located in src/main/resources or otherwise on the classpath.

	2.	Verify Dependencies and Spring Boot Version
Double‑check that you’re using Spring Boot 3.1 or later (which introduced SSL bundles) and that you have the proper dependencies (for example, if you’re using WebClient, ensure you have spring-boot-starter-webflux on the classpath).
	3.	Auto‑Configuration Trigger
The SslBundles bean is auto-configured only when it detects SSL bundle properties. If those properties are missing or misnamed, the bean won’t be created. You might try enabling debug logging (e.g. using --debug as a command-line argument) to see which auto‑configurations are being applied and whether the SSL bundle configuration is picked up.
	4.	Alternative Approaches
If you continue to face issues with the SSL bundle auto‑configuration, you might consider creating your own SslBundle manually using the underlying SSLContext APIs or, as an alternative, configuring your WebClient’s HttpClient (from reactor-netty) with a custom trust manager. This approach is more involved but can serve as a workaround until the SSL bundle auto‑configuration works as expected.

For example, a custom trust manager setup with reactor‑netty might look like this:

import io.netty.handler.ssl.SslContext;
import io.netty.handler.ssl.SslContextBuilder;
import io.netty.handler.ssl.util.InsecureTrustManagerFactory; // or use a proper TrustManagerFactory for your PEM
import reactor.netty.http.client.HttpClient;
import org.springframework.http.client.reactive.ReactorClientHttpConnector;
import org.springframework.web.reactive.function.client.WebClient;

SslContext sslContext = SslContextBuilder.forClient()
        // For production, create a TrustManagerFactory from your PEM certificate instead of insecure!
        .trustManager(InsecureTrustManagerFactory.INSTANCE)
        .build();

HttpClient httpClient = HttpClient.create().secure(sslSpec -> sslSpec.sslContext(sslContext));

WebClient webClient = WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();

(Note: The above uses an insecure trust manager just for illustration—you’d need to load your PEM certificate into a proper TrustManagerFactory.)

In summary, the most common cause for your issue is that the SSL bundle isn’t being auto-configured because its configuration isn’t detected. Double‑check your property names and file locations. If everything is correct and you’re on Spring Boot 3.1+, then enabling debug logging to review auto‑configuration can provide more insight.

Let me know if this helps or if you need further assistance!"