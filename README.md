# Custom Security Context Library
This library is designed
to target specific patterns for implementing Spring Security configurations related to access levels in your project.
It helps developers streamline the security context setup
and minimize the boilerplate code required for configuring security,
particularly when working with JWT.

> These are the requirements that must be met to be able to use this library effectively

- There should be a `Class` to manage `Users`.
- There should be a `Class` to manage `AccessLevels`.
- There should be a `Class` to manage `Permissions`.


## Setup Guidelines
### Adding the dependency locally

- Add the dependency
```xml
<dependency>
  <groupId>io.github.manu-tech-code</groupId>
  <artifactId>sso-security-context</artifactId>
  <version>1.1</version>
</dependency>
```

### Bean Configuration

- Create a `@Bean` of type `ContextConfig`

- Return a new instance `ContextConfig` with these required args in `camelCase`:
  - `repositoryName`: The name of the repository to fetch the user record.
  
  - `methodName`: The method to call on the `repository`.
    > The method should take the `user_id` of type `long`.
  - `prefix`: e.g. `ROLE_`, `SCOPE_` if not necessary an empty string can be used
  
  - `accessLevelFieldName`:  The field name of the access level defined in the parent class.

  - `permissionsFieldName`: The field name of the permissions defined in the AccessLevel Class.

> Example of `@Bean` Declaration in project main class

```java
package org.somepackage;

import com.manuel.sso_security_context.context.ContextConfig;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    ContextConfig contextConfig() {
        return new ContextConfig(
                "employeeRepository",
                "findByUserId",
                "ROLE_",
                "accessLevel",
                "permissions"
        );
    }
}
```

> **NB:** You can create a configuration class to initialize the `ContextConfig` bean

### OAuth2 Configuration

This library uses Springs OAuth2 Resource Server. The public key location should be stated in the `application.yaml` or `application.properties` file

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          public-key-location: classpath:<your-public-key-file>
```
```
spring.security.oauth2.resourceserver.jwt.public-key-location=classpath:<your-public-key-file>
```

- Your `SecurityFilterChain` config should include the following
```
.oauth2ResourceServer(oauth -> oauth
      .jwt(Customizer.withDefaults())
)
```

#### Example SecurityFilterChain Configuration
```java
package org.somepackage;

import org.springframework.context.annotation.Bean;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.stereotype.Component;

@Component
@EnableMethodSecurity(securedEnabled = true)
public class TestSecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity, TestAuthenticationEntryPoint entryPoint) throws Exception {
        httpSecurity
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
                .oauth2ResourceServer(oauth -> oauth
                        .authenticationEntryPoint(entryPoint)
                        .jwt(Customizer.withDefaults())
                );

        return httpSecurity.build();
    }
}
```

- That's it, you are good to go.

- To confirm if the authorities have been set successfully in the `SecurityContext`, you can log the `Authentication` object `SecurityContextHolder.getContext().getAuthentication()`.
