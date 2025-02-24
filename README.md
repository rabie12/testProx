
package eu.olky.wallet.trading.config.security;


import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.ldap.core.support.LdapContextSource;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.ldap.authentication.LdapAuthenticationProvider;
import org.springframework.security.ldap.authentication.PasswordComparisonAuthenticator;
import org.springframework.security.ldap.userdetails.DefaultLdapAuthoritiesPopulator;
import org.springframework.security.ldap.userdetails.LdapAuthoritiesPopulator;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.web.SecurityFilterChain;


@Configuration
public class LdapSecurityConfig {

    private static final String[] AUTH_WHITELIST = {
            // -- Swagger UI v2
            "/v2/api-docs",
            "/swagger-resources",
            "/swagger-resources/**",
            "/configuration/ui",
            "/configuration/security",
            "/swagger-ui.html",
            "/webjars/**",
            // -- Swagger UI v3 (OpenAPI)
            "/v3/api-docs/**",
            "/swagger-ui/**"
    };

    @Value("${spring.security.ldap.urls}")
    private String ldapUrl;

    @Value("${spring.security.ldap.base}")
    private String ldapBase;

    @Value("${spring.security.ldap.userDnPatterns}")
    private String userDnPattern;

    @Value("${spring.security.ldap.group-search-base}")
    private String groupSearchBase;

    @Value("${spring.security.ldap.user}")
    private String adminUser;

    @Value("${spring.security.ldap.secret}")
    private String adminSecret;


    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers(AUTH_WHITELIST).permitAll()
                                .anyRequest().permitAll()
//                      .requestMatchers("/auth/**").permitAll()
//                      .requestMatchers("/admin/**").hasRole("ADMIN")
//                        .anyRequest().authenticated()
                );
 /*               .oauth2ResourceServer((oauth2) -> oauth2
                        .jwt(Customizer.withDefaults())
                )
                .sessionManagement(sess -> sess.sessionCreationPolicy(SessionCreationPolicy.STATELESS));*/
        return http.build();
    }
    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withJwkSetUri("http://localhost:8080/oauth2/jwks").build();
    }


    @Bean
    public LdapAuthoritiesPopulator ldapAuthoritiesPopulator() {
        DefaultLdapAuthoritiesPopulator populator = new DefaultLdapAuthoritiesPopulator(ldapContextSource(), "ou=groups");
        populator.setGroupSearchFilter("(uniqueMember={0})");
        populator.setRolePrefix("ROLE_");
        return populator;
    }
    @Bean
    public LdapAuthenticationProvider ldapAuthenticationProvider() {
        PasswordComparisonAuthenticator authenticator =
                new PasswordComparisonAuthenticator(ldapContextSource());
        authenticator.setUserDnPatterns(new String[]{userDnPattern});
        authenticator.setPasswordAttributeName("password");

        LdapAuthoritiesPopulator authoritiesPopulator =
                new DefaultLdapAuthoritiesPopulator(ldapContextSource(), groupSearchBase);

        return new LdapAuthenticationProvider(authenticator, authoritiesPopulator);
    }

    @Bean
    public LdapContextSource ldapContextSource() {
        LdapContextSource contextSource = new LdapContextSource();
        contextSource.setUrl(ldapUrl);
        contextSource.setBase(ldapBase);
        contextSource.setUserDn(adminUser);
        contextSource.setPassword(adminSecret);
        contextSource.afterPropertiesSet();
        return contextSource;
    }
}




    public boolean isUserInGroup(String username, String groupName) {
        LdapQuery query = LdapQueryBuilder.query()
                .base(base)
                .where("cn").is("OlkyWallet_UAT_AppBO")
                .and("member").is("cn=" + username + ",OU=Groupe_Olky,DC=OLKYPAY,DC=LOCAL");

        List<String> result = ldapTemplate.search(query, new AttributesMapper<String>() {
            @Override
            public String mapFromAttributes(Attributes attributes) throws NamingException {
                return attributes.get("cn") != null ? attributes.get("cn").get().toString() : null;
            }
        });
