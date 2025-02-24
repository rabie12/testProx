Pour configurer votre application Spring Boot afin d’utiliser un certificat au format PEM sans clé privée, vous pouvez suivre les étapes suivantes :

1. Importer le certificat dans le Truststore Java

Si votre objectif est de faire confiance à un serveur LDAP sécurisé, vous devez ajouter le certificat public de ce serveur dans le Truststore de Java (cacerts). Cela permettra à votre application de reconnaître et de faire confiance au certificat du serveur lors de l’établissement de connexions sécurisées.

Étapes :
	•	Obtenir le certificat du serveur LDAP : Assurez-vous d’avoir le fichier PEM du certificat public du serveur LDAP (par exemple, ldap_cert.pem).
	•	Importer le certificat dans le Truststore Java :

keytool -import -alias ldapCert -keystore "$JAVA_HOME/lib/security/cacerts" -file /chemin/vers/ldap_cert.pem

Le mot de passe par défaut du Truststore est changeit.

2. Configurer Spring Boot pour utiliser le Truststore

Après avoir importé le certificat, configurez votre application Spring Boot pour utiliser le Truststore lors des connexions SSL/TLS.

Ajouter les propriétés suivantes dans application.properties ou application.yml :

# application.properties
spring.ldap.urls=ldaps://ldap.example.com:636
spring.ldap.base=dc=example,dc=com
spring.ldap.username=cn=admin,dc=example,dc=com
spring.ldap.password=yourpassword

# application.yml
spring:
  ldap:
    urls: ldaps://ldap.example.com:636
    base: dc=example,dc=com
    username: cn=admin,dc=example,dc=com
    password: yourpassword

Assurez-vous que l’URL utilise ldaps:// pour indiquer une connexion sécurisée.

3. Configurer le LdapContextSource pour utiliser SSL

Dans votre configuration Spring, assurez-vous que le LdapContextSource est correctement configuré pour utiliser SSL :

@Configuration
public class LdapConfig {

    @Value("${spring.ldap.urls}")
    private String ldapUrl;

    @Value("${spring.ldap.base}")
    private String ldapBase;

    @Value("${spring.ldap.username}")
    private String ldapUser;

    @Value("${spring.ldap.password}")
    private String ldapPassword;

    @Bean
    public LdapContextSource contextSource() {
        LdapContextSource contextSource = new LdapContextSource();
        contextSource.setUrl(ldapUrl); // Assurez-vous que l'URL commence par ldaps://
        contextSource.setBase(ldapBase);
        contextSource.setUserDn(ldapUser);
        contextSource.setPassword(ldapPassword);
        return contextSource;
    }

    @Bean
    public LdapTemplate ldapTemplate() {
        return new LdapTemplate(contextSource());
    }
}

Remarques importantes :
	•	Pas de clé privée nécessaire : Si vous ne faites que faire confiance à un serveur LDAP (c’est-à-dire que votre application agit en tant que client), vous n’avez pas besoin d’une clé privée. Seul le certificat public du serveur est requis.
	•	Authentification mutuelle (mTLS) : Si le serveur LDAP exige une authentification mutuelle, votre application devra fournir son propre certificat et sa clé privée. Dans ce cas, vous devrez configurer un keystore contenant votre certificat et votre clé privée.

En suivant ces étapes, votre application Spring Boot sera configurée pour établir des connexions sécurisées avec un serveur LDAP en utilisant un certificat PEM sans clé privée.