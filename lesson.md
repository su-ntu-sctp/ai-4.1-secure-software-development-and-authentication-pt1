# Lesson 4.1: Spring Security Fundamentals & Securing Simple-CRM

## Lesson Overview
This lesson introduces **Spring Security**, the industry-standard framework for protecting Spring-based applications. Up until now, our `simple-crm` project has been "open," meaning any user with the URL could create, read, update, or delete customer data. Today, we transition to a professional, secured architecture. You will learn how Spring Security intercepts requests using a **Filter Chain**, how to manage user identities via properties and beans, and how to implement **Role-Based Access Control (RBAC)**. Finally, we will implement **Password Encoding** using the BCrypt algorithm to ensure user credentials are stored safely.

## Lesson Objectives
By the end of this lesson, learners will be able to:
* **Illustrate** the flow of an HTTP request through the Spring Security Filter Chain to explain how requests are intercepted before reaching a Controller
* **Configure** custom user identities and Role-Based Access Control (RBAC) using a `SecurityFilterChain` bean to protect specific CRM endpoints
* **Implement** a `BCryptPasswordEncoder` to secure user credentials, moving from plain-text storage to industry-standard hashing

---

## Part 1: Security Theory & Architecture

### 1.1 Authentication vs. Authorization
Security in enterprise applications is built on two distinct pillars. It is helpful to think of them in terms of a physical office building with restricted access:

* **Authentication (AuthN)**: This is the front desk where you show your badge or ID card. The primary goal is to answer the question: **"Who are you?"**. The system verifies your credentials — typically a username and a password. If the credentials match the records, Spring Security creates a **Principal** object, which represents the currently logged-in user in the system's memory.
* **Authorization (AuthZ)**: This is your keycard permissions. Just because you are allowed inside the building doesn't mean you have the authority to enter the executive suite or the server room. The goal is to answer the question: **"What are you allowed to do?"**. In our CRM application, we might allow a `SALES_REP` to view customer lists, but we must ensure that only a `MANAGER` has the authorization to perform administrative tasks like deleting records.

### 1.2 The Security Filter Chain
A common misconception is that security logic lives inside your `@RestController`. In reality, Spring Security uses a **Servlet Filter** architecture. Imagine a series of "checkpoints" or "toll booths" that an incoming HTTP request must pass through before it is ever allowed to reach your `CustomerController` methods.

1. **Request Initiation**: A user sends a request, for example: `GET /customers`.
2. **The Filter Chain**: Spring Security intercepts this request before it hits your controller code.
    * **Credential Check**: The first filters check if the request contains any login info (like a Basic Auth header).
    * **Permission Check**: Later filters check if the authenticated user has the required role (e.g., `ROLE_USER` or `ROLE_ADMIN`) for the specific URL path they are trying to access.
3. **The Result**: If the request passes every single checkpoint in the chain, it finally reaches your Controller. If any filter fails (e.g., wrong password or insufficient role), the request is rejected with a `401 Unauthorized` or `403 Forbidden` response.

---

## Part 2: Default Security & Properties Configuration

### 2.1 Step 1: Adding the Security Starter
We begin by adding the Spring Security dependency to the `pom.xml` of your existing `simple-crm` project.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 2.2 Step 2: Observing Default Behavior
By default, Spring Security follows a **"Secure by Default"** philosophy.

1. **Run the App**: Start your CRM application using `mvn spring-boot:run`.
2. **Access Data**: Open your browser and go to `http://localhost:8080/customers`.
3. **The Lockdown**: You will be redirected to a generic login page.
4. **Credentials**: The default username is `user`. The password is a **unique UUID string** printed in your IDE console logs during startup.

### 2.3 Step 3: Configuring Security in application.properties
Using a random password from the console is inconvenient for development. We can set a static username and password in `application.properties` to make testing more predictable.

```properties
# Customizing the default security user
spring.security.user.name=admin
spring.security.user.password=password123
```

**Note**: Once you add these lines and restart the app, the console will no longer print a random password. You can now log in using `admin` and `password123`. This is a quick way to handle **Basic Authentication** during early development.

---

## Part 3: Custom Configuration & Password Encoding

### 3.1 The Importance of Password Encoding
In a real application, you should **never** store passwords in plain text. If a database is compromised, plain text passwords allow hackers to access every account immediately. Instead, we use a **Hashing Algorithm**.

Hashing is a one-way process that turns a password into a complex string of characters. **BCrypt** is one of the most trusted hashing algorithms because it is slow and computationally expensive, making "brute force" attacks very difficult. When a user logs in, Spring Security hashes the provided password and compares it to the hash stored in the system.

### 3.2 Creating the SecurityConfig (Instructor Demo)

In VS Code, right-click your main package folder → **New Folder** → name it `config`. Then create `SecurityConfig.java` inside it.

**Demo Goal**: Define a Password Encoder and secure the `POST /customers` endpoint so only an `ADMIN` can create customers.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    // Define the Password Encoder Bean.
    // Spring Security will use this to hash and match user passwords.
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // 1. Disable CSRF to allow Postman to send POST/PUT/DELETE requests.
            .csrf(csrf -> csrf.disable())

            // 2. Define Authorization rules
            .authorizeHttpRequests(auth -> auth
                // Allow anyone to access the public app info endpoint.
                .requestMatchers("/app-info").permitAll()

                // Only users with the ADMIN role can create customers.
                .requestMatchers(HttpMethod.POST, "/customers/**").hasRole("ADMIN")

                // Every other request requires the user to be logged in.
                .anyRequest().authenticated()
            )

            // 3. Enable Basic Authentication for Postman (standard for REST APIs).
            .httpBasic(Customizer.withDefaults());

        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        // Create a standard staff user.
        // We use passwordEncoder.encode() to store the password securely.
        UserDetails sales = User.builder()
            .username("sales_rep")
            .password(passwordEncoder.encode("sales123"))
            .roles("USER")
            .build();

        // Create an administrative user.
        UserDetails manager = User.builder()
            .username("manager")
            .password(passwordEncoder.encode("admin123"))
            .roles("ADMIN")
            .build();

        return new InMemoryUserDetailsManager(sales, manager);
    }
}
```

### 3.3 Understanding UserDetailsService

The `UserDetailsService` is one of the **core extension points** in Spring Security and plays a central role in the **authentication process**.

When a user attempts to log in, Spring Security does not directly know how or where your users are stored. Instead, it delegates this responsibility to the `UserDetailsService`. Spring Security calls the `loadUserByUsername()` method and passes the username entered by the user.

Your `UserDetailsService` implementation must then return a `UserDetails` object containing:
- the username
- the encoded password
- the roles or authorities assigned to the user

Spring Security then compares the encoded password returned by `UserDetailsService` with the password provided during login (after encoding it). If they match, authentication succeeds. If the user is not found or the passwords do not match, authentication fails immediately.

In real-world applications, `UserDetailsService` typically loads users from a database. In this lesson, we use an **in-memory implementation** so you can focus on understanding how Spring Security works without introducing database complexity.

If this concept feels difficult at first, that is completely normal. Many developers treat the `UserDetailsService` as standard configuration code and reuse common patterns. With practice, its role in the authentication flow will become much clearer.

---

## Part 4: Activity — Securing CRM Endpoints **(20 minutes)**

### Task 1: Rule Implementation
Update your `SecurityConfig.java` to enforce these organisational rules:

1. **Managerial Oversight**: Only users with the `ADMIN` role should be allowed to delete a customer record (`DELETE /customers/{id}`).
2. **Staff View**: Ensure both `USER` and `ADMIN` roles can view the customer list (`GET /customers`).

### Task 2: Testing with Postman
1. **Auth Setup**: In Postman, go to **Authorization**, select **Basic Auth**, and enter `sales_rep` credentials.
2. **Verify Restriction**: Try to `DELETE` a customer. You should receive a **403 Forbidden** status code.
3. **Verify Access**: Try to `GET` the list of customers. This should return **200 OK**.
4. **Admin Check**: Change your credentials to `manager` and verify you can successfully `DELETE` a customer record.

---

## Summary
* **Authentication vs. Authorization**: Authentication identifies who you are (ID check); Authorization determines your access level (keycard check).
* **Filter Chain**: Spring Security acts as a series of checkpoints (Filters) that validate requests before they reach your RestController.
* **Properties Config**: You can quickly set a single user and password in `application.properties` for simple development tasks.
* **BCrypt Password Encoder**: Passwords must never be stored in plain text. We use the BCrypt algorithm to hash passwords securely.
* **Role-Based Access Control (RBAC)**: By assigning Roles (`USER`, `ADMIN`) to users, we can restrict specific HTTP methods (like `DELETE` or `POST`) to authorised personnel only.

---

END