# Sprint Webflux Demo Project for learning

## Keycloak Server

### Run Server

Run docker command, start keycloak admin server
```shell
docker run -p KEYCLOAK_SERVER_PORT:8080 --name keycloak -d -e KEYCLOAK_ADMIN=YOUR_KEYCLOAK_ADMIN_ID -e KEYCLOAK_ADMIN_PASSWORD=YOUR_KEYCLOAK_ADMIN_PASSWORD quay.io/keycloak/keycloak:19.0.3 start-dev
```
edit command `KEYCLOAK_SERVER_PORT` for your external keycloak server port,

`YOUR_KEYCLOAK_ADMIN_ID` for your keycloak admin ID

`YOUR_KEYCLOAK_ADMIN_PASSWORD` for your keycloak password

then Create Your Own _Realm_ and _Client_

## Keycloak Config for Spring Boot

```yaml
Gateway - application.yml

keycloak:
  base-url: YOUR_KEYCLOAK_SERVER_URL
  realm: YOUR_REALM_NAME
  realm-url: ${keycloak.base-url}/realms/${keycloak.realm}
  clientId: YOUR_CLIENT
  clientSecret: YOUR_CLIENT_SECRET

spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          jwk-set-uri: ${keycloak.base-url}/realms/${keycloak.realm}/protocol/openid-connect/certs
          issuer-uri: ${keycloak.base-url}/realms/${keycloak.realm}
      client:
        registration:
          keycloak:
            provider: keycloak
            client-id: ${keycloak.clientId}
            client-secret: ${keycloak.clientSecret}
            authorization-grant-type: authorization_code
        provider:
          keycloak:
            authorization-uri: ${keycloak.realm-url}/protocol/openid-connect/auth
            jwk-set-uri: ${keycloak.realm-url}/protocol/openid-connect/certs
            token-uri: ${keycloak.realm-url}/protocol/openid-connect/token
            user-info-uri: ${keycloak.realm-url}/protocol/openid-connect/userinfo
            user-name-attribute: preferred_username
            issuer-uri: ${keycloak.base-url}/realms/${keycloak.realm}
```
edit the value which has prefix `YOUR_` such as `YOUR_KEYCLOAK_SERVER_URL`..
- `spring.security.oauth2.client` option is for OAuth2 Login in your Application.
- `spring.security.oauth2.resourceserver` is for Authorization by JWT(or other token..)

You can use one or both of them for your usage.

## Security Config for OAuth2
```java
// gateway.config.SecurityConfig

@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    SecurityWebFilterChain configure(ServerHttpSecurity http) {
        return http
                .authorizeExchange(exchanges ->
                        exchanges
                                .pathMatchers("/login", "/login/**").permitAll()
                                .anyExchange().authenticated()
                )
                .csrf().disable()
                .cors().disable()
                .oauth2Login(Customizer.withDefaults())
                .oauth2ResourceServer(ServerHttpSecurity.OAuth2ResourceServerSpec::jwt)
                .build();
    }
}
```

Configure Webflux Security Options by define ServerHttpSecurity.

Use both `Oauth2 Login` for keycloak login authorization
and `Oauth2 Resource Server` for keycloak Jwt authorization.