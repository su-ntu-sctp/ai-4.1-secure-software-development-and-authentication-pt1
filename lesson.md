# Activity Solutions — Lesson 4.1: Spring Security

## Task 1: Updated SecurityConfig.java

The only change from the demo is adding the two new `requestMatchers` rules inside the `authorizeHttpRequests` block. The rest of the class stays exactly the same.

```java
package com.ntu.sg.simple_crm.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http
            .csrf(csrf -> csrf.disable())

            .authorizeHttpRequests(auth -> auth
                // Task 1, Rule 1: Only ADMIN can delete customers
                .requestMatchers(HttpMethod.DELETE, "/customers/**").hasRole("ADMIN")

                // From the demo: Only ADMIN can create customers
                .requestMatchers(HttpMethod.POST, "/customers/**").hasRole("ADMIN")

                // Task 1, Rule 2: Both USER and ADMIN can view customers
                .requestMatchers(HttpMethod.GET, "/customers/**").hasAnyRole("USER", "ADMIN")

                // All other requests require authentication
                .anyRequest().authenticated()
            )

            .httpBasic(Customizer.withDefaults());

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        UserDetails sales = User.builder()
                .username("sales_rep")
                .password(passwordEncoder.encode("sales123"))
                .roles("USER")
                .build();

        UserDetails manager = User.builder()
                .username("manager")
                .password(passwordEncoder.encode("admin123"))
                .roles("ADMIN")
                .build();

        return new InMemoryUserDetailsManager(sales, manager);
    }
}
```

---

## Task 2: Expected Postman Results

| Action | User | Expected Response |
|---|---|---|
| `DELETE /customers/1` | `sales_rep` | `403 Forbidden` |
| `GET /customers` | `sales_rep` | `200 OK` |
| `DELETE /customers/1` | `manager` | `200 OK` |
| `POST /customers` | `sales_rep` | `403 Forbidden` |
| `POST /customers` | `manager` | `201 Created` |

---

## Key Teaching Point: Rule Order Matters

Spring Security evaluates `authorizeHttpRequests` rules **top to bottom** and stops at the first match. Always place more specific rules (specific HTTP method + path) **above** broader rules like `.anyRequest().authenticated()`.

If `.anyRequest()` appeared first, it would match everything and your specific rules below it would never be reached.