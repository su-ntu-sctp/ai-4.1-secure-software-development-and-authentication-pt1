# Self Studies: Spring Security Fundamentals & Securing Simple-CRM

## Overview

In this lesson you will secure the `simple-crm` application for the first time — adding authentication, authorization, and password hashing. These concepts are new and conceptually different from anything covered so far. The self-study materials below will help you arrive with a clear mental model of how Spring Security works so you can focus on the code-along rather than catching up on theory.

**Estimated Prep Time:** 60–80 minutes

---

## Task 1: Spring Security Tutorial for Beginners

This video covers the core Spring Security concepts you will use in the lesson — the filter chain, authentication vs authorization, Basic Auth, role-based access control, and password encoding. Watch it in full before class.

**Watch:** Spring Security Tutorial for Beginners
🎬 https://www.youtube.com/watch?v=2tf0UY6gV3Y

**Then read:** Lesson 4.1 — Parts 1 to 3

**Guiding Questions:**
- What is the difference between authentication and authorization?
- What does the Spring Security Filter Chain do before a request reaches your controller?
- What is the difference between a `401 Unauthorized` and a `403 Forbidden` response?
- Why should passwords never be stored in plain text?
- What does BCrypt do to a password and why is it considered secure?

---

## Task 2: Read — UserDetailsService and RBAC

This is a read-only task to solidify your understanding of how Spring Security manages users and roles before the code-along.

**Read:** Lesson 4.1 — Part 3 (SecurityConfig and UserDetailsService sections)

**Guiding Questions:**
- What is the role of `UserDetailsService` in the authentication process?
- What does `InMemoryUserDetailsManager` do and when would you use it?
- What is Role-Based Access Control and how does `.hasRole("ADMIN")` enforce it?
- What is the difference between `.permitAll()` and `.authenticated()`?

---

## Task 3: Prepare Your Environment

Before class, make sure your `simple-crm` project is in a working state with all endpoints functioning correctly. The lesson builds directly on top of it.

**Checklist:**
- [ ] `simple-crm` runs without errors on `mvn spring-boot:run`
- [ ] `GET /customers`, `POST /customers`, `DELETE /customers/{id}` all return expected responses in Postman
- [ ] PostgreSQL is running and connected

---

## Active Engagement Strategies

- As you watch the video, pause whenever a new annotation is introduced (`@EnableWebSecurity`, `@Bean`, etc.) and write down in one sentence what it does
- After reading Part 1, try to draw the filter chain on paper — from incoming HTTP request to your controller — and label each checkpoint
- Think about the `simple-crm` project: which endpoints should be public, which should require login, and which should be admin-only? Bring your thoughts to class

---

## Additional Reading Material

- [Spring Security Architecture — Spring Docs](https://spring.io/guides/topicals/spring-security-architecture)
- [Introduction to Spring Security — Baeldung](https://www.baeldung.com/spring-security-basic-authentication)
- [BCrypt Password Encoding — Baeldung](https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
- [Role-Based Access Control in Spring — Baeldung](https://www.baeldung.com/role-and-privilege-for-spring-security-registration)