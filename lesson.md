# Module 4 – Lesson 4.1  
# Spring Security Fundamentals (Basic Authentication & Role-Based Authorization)

---

## Lesson Overview

In all the previous modules, our `simple-crm` application has been completely **open**. Any user who knows the API endpoints can create, update, or delete customer data. While this is acceptable for learning purposes, it is **not acceptable in real-world applications**.

In this lesson, we introduce **Spring Security**, the standard security framework used in Spring-based applications. Spring Security allows us to control **who** can access our application and **what** they are allowed to do.

This lesson focuses on **foundational security concepts** and applies them directly to the existing **Simple CRM project** built in the Spring Boot module. We will start with **basic authentication**, then gradually enhance it using **role-based authorization**, and finally introduce **password encoding** to follow security best practices.

The goal of this lesson is not to secure everything at once, but to **understand the security model clearly** and apply it incrementally in a way that students can follow and extend confidently.

---

## Lesson Duration

**Approximate Duration:** 3 hours

- Conceptual explanations and walkthroughs: ~1.5 hours  
- Guided implementation: ~1 hour  
- Hands-on student activity and discussion: ~30 minutes  

---

## Lesson Objectives

By the end of this lesson, students will be able to:

- Explain why application-level security is required for backend APIs
- Understand the role of Spring Security in a Spring Boot application
- Differentiate between authentication and authorization
- Configure basic authentication using Spring Security
- Apply role-based authorization to a REST endpoint
- Encode passwords securely using a password encoder
- Secure a simple endpoint in the existing `simple-crm` project
- Extend security rules to other endpoints as a hands-on exercise

---

## Prerequisites

Before starting this lesson, students should already be comfortable with:

- Spring Boot project structure
- REST controllers and endpoints
- Service and repository layers
- The `simple-crm` application created in earlier lessons
- Basic Maven dependency management

---

## Part 1: Why Do We Need Security?

So far, our API looks something like this:

```
GET  /customers
POST /customers
PUT  /customers/{id}
DELETE /customers/{id}
```

Anyone can call these endpoints using tools like Postman, curl, or even a browser.

### Problems with an Unsecured API

- Anyone can read sensitive customer data
- Anyone can modify or delete records
- There is no accountability (we don’t know who made a change)
- This violates basic security and compliance standards

In real systems, we need answers to questions like:
- Who is making this request?
- Is this user authenticated?
- Is this user allowed to perform this action?

This is where **Spring Security** comes in.

---

## Part 2: What Is Spring Security?

Spring Security is a powerful and flexible framework that provides:

- Authentication (verifying identity)
- Authorization (verifying permissions)
- Protection against common security vulnerabilities

Spring Security works by placing a **security filter chain** in front of your application. Every incoming HTTP request passes through this chain **before** it reaches your controllers.

At a high level, the flow looks like this:

```
Client Request
   ↓
Spring Security Filter Chain
   ↓
Controller
   ↓
Service → Repository
```

If a request fails security checks, it never reaches the controller.

---

## Part 3: Authentication vs Authorization

These two concepts are often confused, so let’s be very clear.

### Authentication

Authentication answers the question:

**Who are you?**

Examples:
- Username and password
- Token validation
- OAuth login

In this lesson, we will start with **basic authentication using username and password**.

### Authorization

Authorization answers the question:

**Are you allowed to do this?**

Examples:
- Only ADMIN users can delete data
- Only authenticated users can view data
- Different roles have different permissions

In this lesson, we will use **role-based authorization**.

---

## Part 4: Adding Spring Security to `simple-crm`

### Step 1: Add Spring Security Dependency

Open `pom.xml` and add the Spring Security starter:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

After adding this dependency, restart the application.

### What Happens Immediately?

Once Spring Security is added:
- All endpoints are secured by default
- Spring Security generates a default user
- A password is printed in the console at startup

Try accessing:
```
GET http://localhost:8080/customers
```

You will now receive:
```
401 Unauthorized
```

This confirms that Spring Security is active.

---

## Part 5: Default Spring Security Behavior

By default, Spring Security:
- Enables HTTP Basic authentication
- Secures all endpoints
- Creates a default user with a generated password

This default behavior is **not suitable for production**, but it helps us understand how security intercepts requests.

---

## Part 6: Creating a Custom Security Configuration

To control security behavior, we create a **Security Configuration class**.

### Step 1: Create SecurityConfig

Create a new class:

```
src/main/java/.../config/SecurityConfig.java
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

}
```

This class will define:
- Which endpoints are secured
- How authentication works
- What roles are required

---

## Part 7: In-Memory Authentication (For Learning)

For learning purposes, we will use **in-memory users**.

### Why In-Memory Users?

- Simple to set up
- No database required
- Easy to understand authentication flow

This is **not** how production systems work, but it is perfect for learning.

---

## Part 8: Configuring Basic Authentication

Add the following method inside `SecurityConfig`:

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

    http
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/customers").authenticated()
            .anyRequest().permitAll()
        )
        .httpBasic();

    return http.build();
}
```

### Explanation

- CSRF is disabled because we are building a stateless REST API
- `/customers` requires authentication
- All other endpoints are temporarily permitted
- HTTP Basic authentication is enabled

---

## Part 9: Creating Users with Roles

Now let’s define users and roles.

```java
@Bean
public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {

    User user = User.builder()
        .username("user")
        .password(passwordEncoder.encode("password"))
        .roles("USER")
        .build();

    User admin = User.builder()
        .username("admin")
        .password(passwordEncoder.encode("admin123"))
        .roles("ADMIN")
        .build();

    return new InMemoryUserDetailsManager(user, admin);
}
```

---

## Part 10: Password Encoding

Storing passwords as plain text is **dangerous**.

Spring Security requires passwords to be encoded.

### Create a Password Encoder Bean

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### Why BCrypt?

- One-way hashing
- Automatically handles salt
- Industry standard

---

## Part 11: Role-Based Authorization (RBAC)

Now let’s enhance security by applying **role-based rules**.

We will use **ONE simple endpoint** as a demonstration:
```
GET /customers
```

Update the security rules:

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers(HttpMethod.GET, "/customers").hasAnyRole("USER", "ADMIN")
    .requestMatchers(HttpMethod.DELETE, "/customers/**").hasRole("ADMIN")
    .anyRequest().authenticated()
)
```

### What This Means

- Both USER and ADMIN can view customers
- Only ADMIN can delete customers
- All requests require authentication

---

## Part 12: Testing with Postman

### Test as USER
- Username: `user`
- Password: `password`
- Try:
  - GET `/customers` → ✅ Allowed
  - DELETE `/customers/{id}` → ❌ Forbidden

### Test as ADMIN
- Username: `admin`
- Password: `admin123`
- Try:
  - GET `/customers` → ✅ Allowed
  - DELETE `/customers/{id}` → ✅ Allowed

This demonstrates **both authentication and authorization on the same endpoint**.

---

## Part 13: Guided Hands-On Activity

### Activity 1 (Instructor-Guided)

- Secure the `GET /customers/{id}` endpoint
- Allow both USER and ADMIN roles
- Verify behavior using Postman

---

## Part 14: Student Hands-On Activity

### Activity 2 (Student-Driven)

Students should:
- Secure remaining **simple CRM endpoints**
- Apply appropriate role-based rules
- Avoid endpoints involving JPA relationships
- Test using different users

Suggested rules:
- USER: read-only
- ADMIN: full access

---

## Part 15: Key Takeaways

- Spring Security intercepts requests before controllers
- Authentication verifies identity
- Authorization verifies permissions
- Passwords must always be encoded
- Security should be applied incrementally
- Simple examples build strong understanding

---

## What’s Next?

In the next lesson, we will:
- Integrate security with persistent users
- Explore token-based authentication concepts
- Prepare the application for production-grade security

---
