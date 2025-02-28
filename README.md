L’erreur indique que Spring ne parvient pas à trouver un bean de type IbanSearchHistoryMapper lors de l’injection dans BankInfoValidationService.

Solutions possibles :

1. Vérifier que IbanSearchHistoryMapper est bien annoté avec @Mapper

Si tu utilises MapStruct, assure-toi que l’interface IbanSearchHistoryMapper a bien l’annotation :

@Mapper(componentModel = "spring")
public interface IbanSearchHistoryMapper {
    // Mapping methods here
}

✅ L’option componentModel = "spring" permet à MapStruct de générer un bean Spring.

2. Vérifier que MapStruct est bien configuré dans pom.xml

Si tu utilises Maven, ajoute :

<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>

<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.5.5.Final</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok-mapstruct-binding</artifactId>
    <version>0.2.0</version>
    <scope>provided</scope>
</dependency>

✅ Assure-toi aussi que Lombok est bien activé si tu utilises @Data ou @AllArgsConstructor.

3. Vérifier que le package du mapper est bien scanné par Spring

Si ton IbanSearchHistoryMapper est dans un package différent de ton application principale, ajoute un @ComponentScan dans BankInfoApplication :

@SpringBootApplication
@ComponentScan(basePackages = "eu.olky.bankInfo")
public class BankInfoApplication {
    public static void main(String[] args) {
        SpringApplication.run(BankInfoApplication.class, args);
    }
}

✅ Remplace "eu.olky.bankInfo" par le package parent de ton projet.

4. Forcer la génération du code MapStruct

Si le problème persiste, force la génération des classes MapStruct :
	•	Mets à jour Maven : mvn clean install
	•	Vérifie dans target/generated-sources/annotations si IbanSearchHistoryMapperImpl a bien été généré.

Conclusion

✔️ Ajoute @Mapper(componentModel = "spring")
✔️ Vérifie les dépendances MapStruct
✔️ Ajoute @ComponentScan si besoin
✔️ Regénère le code avec Maven

Essaie ça et dis-moi si l’erreur persiste !