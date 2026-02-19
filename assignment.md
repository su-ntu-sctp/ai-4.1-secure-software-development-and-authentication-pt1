# Assignment (Optional)

## Brief

Secure a Spring Boot application using Spring Security with Basic Authentication, Role-Based Access Control (RBAC), and BCrypt password encoding.

1. **Basic Spring Security Setup with Password Encoding**
   - Use your existing project (e.g., simple-crm, EmployeeManagementSystem, or any previous REST API project)
   - Add the Spring Security starter dependency to `pom.xml`
   - Create a `SecurityConfig` class in a `config` package with:
     - `@Configuration` and `@EnableWebSecurity` annotations
   - Create a `PasswordEncoder` bean that returns `new BCryptPasswordEncoder()`
   - Create a `UserDetailsService` bean with two in-memory users:
     - Username: "user", Password: "user123", Role: "USER"
     - Username: "admin", Password: "admin123", Role: "ADMIN"
   - Remember to encode passwords using `passwordEncoder.encode()`
   - Test the application:
     - Start the application
     - Try accessing any endpoint without credentials (should be blocked)
     - Use Postman with Basic Auth to login with both user accounts
     - Verify both users can successfully authenticate

2. **Implement Role-Based Access Control (RBAC)**
   - Create a `SecurityFilterChain` bean in your `SecurityConfig` class
   - Configure the filter chain to:
     - Disable CSRF: `.csrf(csrf -> csrf.disable())`
     - Enable Basic Authentication: `.httpBasic(Customizer.withDefaults())`
     - Define authorization rules using `.authorizeHttpRequests()`:
       - Allow public access to one endpoint (e.g., GET /public or GET /health) using `.permitAll()`
       - Restrict POST requests to your main resource (e.g., POST /customers, POST /employees) to only users with "ADMIN" role
       - Restrict DELETE requests to only users with "ADMIN" role
       - Allow GET requests to all authenticated users (both USER and ADMIN roles)
   - Test with Postman:
     - Login as "user" and verify:
       - Can access GET endpoints (200 OK)
       - Cannot access POST/DELETE endpoints (403 Forbidden)
     - Login as "admin" and verify:
       - Can access all endpoints including POST and DELETE (200/201 OK)
   - Document your test results with screenshots or descriptions

## Submission (Optional)

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

## References
- Java: https://docs.oracle.com/javase/
- Spring Boot: https://docs.spring.io/spring-boot/docs/current/reference/html/
- PostgreSQL: https://www.postgresql.org/docs/
- OWASP: https://cheatsheetseries.owasp.org/