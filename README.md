Pour configurer votre application Spring Boot afin d’utiliser un truststore au format PEM, vous pouvez exploiter la fonctionnalité des SSL Bundles introduite dans Spring Boot 3.1. Cette approche simplifie la gestion des certificats et des clés en permettant de les regrouper sous forme de bundles réutilisables.

Étapes de Configuration
	1.	Préparer le certificat PEM
Assurez-vous de disposer du certificat PEM du serveur auquel votre application doit faire confiance, par exemple server-cert.pem.
	2.	Configurer le SSL Bundle dans application.yml ou application.properties
Dans votre fichier de configuration Spring Boot, définissez un bundle SSL en spécifiant le certificat PEM :
Exemple avec application.yml :

spring:
  ssl:
    bundle:
      pem:
        truststore-bundle:
          truststore:
            certificate: "classpath:server-cert.pem"

Exemple avec application.properties :

spring.ssl.bundle.pem.truststore-bundle.truststore.certificate=classpath:server-cert.pem

Dans ces exemples, truststore-bundle est le nom que vous attribuez au bundle SSL. Le chemin classpath:server-cert.pem indique que le certificat est situé dans le classpath de votre application.

	3.	Appliquer le SSL Bundle à votre application
Pour utiliser ce bundle SSL dans votre application, vous pouvez le référencer dans les configurations appropriées. Par exemple, pour sécuriser un client REST :

@Service
public class MyService {

    private final RestTemplate restTemplate;

    import org.springframework.boot.ssl.SslBundle;
import org.springframework.boot.ssl.SslBundles;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class WebClientConfig {

    @@Service
public class MyService {

    private final RestTemplate restTemplate;

    public MyService(RestTemplateBuilder restTemplateBuilder, SslBundles sslBundles) {
        this.restTemplate = restTemplateBuilder
            .setSslBundle(sslBundles.getBundle("truststore-bundle"))
            .build();
    }

    // Méthodes de service utilisant restTemplate
}
}
    // Méthodes de service utilisant restTemplate
}

Dans cet exemple, le RestTemplate est configuré pour utiliser le bundle SSL nommé truststore-bundle, assurant ainsi que les communications HTTP utilisent le certificat spécifié.

Remarques Importantes
	•	Version de Spring Boot : Assurez-vous d’utiliser Spring Boot 3.1 ou une version ultérieure pour bénéficier de la fonctionnalité des SSL Bundles.
	•	Chemin du certificat : Le préfixe classpath: indique que le fichier server-cert.pem doit être placé dans le classpath de votre application, généralement dans le répertoire src/main/resources.
	•	Authentification Mutuelle (mTLS) : Si votre application nécessite une authentification mutuelle, vous devrez également configurer un keystore contenant votre certificat client et sa clé privée. Cela peut être fait en ajoutant une section keystore dans la configuration du bundle SSL.

En suivant ces étapes, vous pourrez configurer votre application Spring Boot pour utiliser un truststore au format PEM, facilitant ainsi la gestion des certificats et améliorant la sécurité de vos communications.