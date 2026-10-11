# Spring Security — Notes

## 1. What is Spring Security?

Spring Security is a framework that provides authentication, authorization, and protection against common security attacks in Spring applications.

**Authentication:** Verifies who the user is.

Example: Checking a username and password during login.

**Authorization:** Determines what an authenticated user is allowed to access.

Example: A normal user can view their own transactions, but an administrator can manage all users.

Spring Security also provides protection against attacks such as Cross-Site Request Forgery (CSRF).

## 2. How an HTTP Request Flows Through Spring Security

In a typical Spring Boot application using Spring MVC and Spring Security, the request flow is:

```text
Client / Browser / Postman
          |
          v
Spring Security Filter Chain
          |
          v
Servlet Container (Tomcat)
          |
          v
DispatcherServlet
          |
          v
Controller
          |
          v
Service
          |
          v
Repository
          |
          v
Database
```

The Spring Security filter chain is applied within the servlet-based request processing infrastructure, before the request reaches the Spring MVC controller.

The filters can inspect credentials, authenticate users, check authorization, handle CSRF protection, and reject requests.

If a request is rejected by a security filter, it may never reach the controller.

### Important components

**Servlet Container:** Manages servlets and their request-processing lifecycle. Tomcat is a common servlet container.

**DispatcherServlet:** Spring MVC's Front Controller. It dispatches requests to the appropriate controller handler.

**SecurityFilterChain:** Defines the security filters and rules applied to matching requests.

**Controller:** Handles application requests after they pass the relevant security checks.

## 3. Servlet-Based and Reactive Applications

Spring supports two main web programming models.

### Servlet-based (Spring MVC)

- Uses the Servlet API.
- Commonly runs on Tomcat.
- Uses the Spring Security servlet filter chain.
- Suitable for conventional Spring Boot REST APIs and web applications.

### Reactive (Spring WebFlux)

- Uses reactive programming APIs.
- Commonly uses Reactor and non-blocking request processing.
- Spring Security uses a reactive security web filter chain rather than the servlet-based `SecurityFilterChain`.

For your current Spring Boot MVC application, focus on the servlet-based approach.

## 4. What Happens When Spring Security Is Added?

When Spring Boot detects Spring Security on the classpath, it applies default security configuration unless you provide your own configuration.

By default, a typical Spring Boot application with Spring Security:

- Requires authentication for requests.
- Supports form-based login.
- Supports HTTP Basic authentication when configured.
- Generates a development password and prints it in the application startup logs when the default user is configured.
- Creates a default username, commonly `user`, unless configured otherwise.

You can configure the default user in `application.properties`:

```properties
spring.security.user.name=sekhar
spring.security.user.password=Password
```

These settings are useful for learning and simple development scenarios. For production, use an appropriate user store and secure password management.

## 5. Security Configuration Using SecurityFilterChain

You can define a configuration class to customize Spring Security.

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(
            HttpSecurity http) throws Exception {

        http.authorizeHttpRequests(request ->
                request.anyRequest().authenticated()
        );

        http.formLogin(Customizer.withDefaults());
        http.httpBasic(Customizer.withDefaults());

        return http.build();
    }
}
```

### Important annotations and methods

**`@Configuration`**

Marks a class that declares Spring configuration.

**`@Bean`**

Registers the object returned by the method as a Spring-managed bean.

**`SecurityFilterChain`**

Represents a configured set of security filters and rules for matching requests.

**`HttpSecurity`**

Provides methods to configure web security, including authorization, login mechanisms, CSRF protection, and session management.

**`http.build()`**

Builds the configured `SecurityFilterChain`.

**`Customizer.withDefaults()`**

Uses the default configuration for the relevant feature.

### Why create this configuration?

It allows you to customize the security behavior rather than relying entirely on Spring Boot's default configuration.

`@EnableWebSecurity` is available to enable web security configuration explicitly, but it is generally not necessary merely to customize Spring Security in a typical Spring Boot application that already has the security starter.

## 6. Authentication Flow

Suppose a user submits:

```text
Username: sekhar
Password: entered password
```

The general authentication flow is:

```text
Login Request
      |
      v
Authentication Filter
      |
      v
AuthenticationManager
      |
      v
AuthenticationProvider
      |
      v
UserDetailsService
      |
      v
Load User Details
      |
      v
Password Verification
      |
   +--+--+
   |     |
Valid   Invalid
   |     |
Success  Authentication Failure
```

The exact filter and processing path depend on the authentication mechanism being used.

### AuthenticationManager

Coordinates authentication by delegating to an appropriate `AuthenticationProvider`.

### AuthenticationProvider

Performs the actual authentication logic for a particular authentication mechanism.

For username/password authentication backed by a user store, a common implementation is `DaoAuthenticationProvider`.

### UserDetailsService

Loads user information using a username.

Its key method is:

```java
UserDetails loadUserByUsername(String username);
```

Spring Security uses the returned `UserDetails` to obtain information such as the username, encoded password, and authorities.

`UserDetailsService` normally loads the user; it does not itself compare the submitted password with the stored password.

### UserDetails

Represents the information Spring Security needs about a user, including:

- Username
- Password
- Authorities
- Account status

### PasswordEncoder

Verifies a submitted password against the stored password representation.

For BCrypt, the stored password is a hash, not the original plain-text password.

## 7. DaoAuthenticationProvider

`DaoAuthenticationProvider` is a common `AuthenticationProvider` implementation for username/password authentication.

It works with:

- `UserDetailsService`
- `PasswordEncoder`

Example:

```java
@Bean
public AuthenticationProvider authenticationProvider(
        UserDetailsService userDetailsService) {

    DaoAuthenticationProvider provider =
            new DaoAuthenticationProvider(userDetailsService);

    provider.setPasswordEncoder(
            new BCryptPasswordEncoder(12)
    );

    return provider;
}
```

The constructor shown is available in current Spring Security versions; older versions may use a different constructor and setter combination.

### What happens internally?

1. Spring Security receives the username and password.
2. The provider asks `UserDetailsService` to load the user.
3. The returned `UserDetails` contains the stored encoded password.
4. The provider uses `PasswordEncoder` to verify the submitted password.
5. If the credentials are valid and the account checks pass, authentication succeeds.
6. Otherwise, authentication fails.

## 8. Custom UserDetailsService

Your implementation:

```java
@Service
public class MyUserDetailsService
        implements UserDetailsService {

    @Autowired
    private UserRepo repo;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        Users user = repo.findUserByUsername(username);

        if (user == null) {
            throw new UsernameNotFoundException("User not found");
        }

        return new UserPrincipal(user);
    }
}
```

### How it works

1. Spring Security calls `loadUserByUsername()`.
2. The service queries the database through `UserRepo`.
3. If the user exists, it wraps the database entity in `UserPrincipal`.
4. `UserPrincipal` provides the user information through the `UserDetails` interface.
5. Spring Security uses that information to finish authentication.

The repository method:

```java
Users findUserByUsername(String username);
```

is a Spring Data JPA derived query method.

## 9. UserPrincipal and UserDetails

Your `UserPrincipal` implements `UserDetails` and adapts your application-specific `Users` entity to the format expected by Spring Security.

For example:

```java
@Override
public String getUsername() {
    return user.getUsername();
}

@Override
public String getPassword() {
    return user.getPassword();
}
```

The other methods describe account status:

- `isAccountNonExpired()`
- `isAccountNonLocked()`
- `isCredentialsNonExpired()`
- `isEnabled()`

Returning `true` for all these methods means your implementation does not currently enforce expiration, lockout, or disabled-account checks.

### Authorities

Authorities represent the permissions associated with an authenticated user.

For example:

```java
return Collections.singleton(
        new SimpleGrantedAuthority("ROLE_USER")
);
```

Spring Security can use these authorities to make authorization decisions.

## 10. Password Encoding with BCrypt

Do not store user passwords in plain text.

For registration, encode the submitted password before saving it:

```java
user.setPassword(passwordEncoder.encode(user.getPassword()));
repo.save(user);
```

Use a configured `PasswordEncoder` bean rather than creating separate encoders in different services.

For example:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12);
}
```

Then inject this bean into the registration service and configure the authentication provider to use the same encoding algorithm.

### Important distinction

Registration:

```text
Plain-text password
       |
       v
PasswordEncoder.encode()
       |
       v
Stored BCrypt hash
```

Login:

```text
Submitted password + Stored BCrypt hash
                 |
                 v
       PasswordEncoder.matches()
                 |
                 v
        Authentication result
```

BCrypt uses a salt as part of its hashing process. Do not encode the submitted login password and compare the two hash strings directly; use the encoder's matching mechanism.

## 11. Registering a User

Your registration flow is:

```text
POST /register
      |
      v
UserController
      |
      v
UserService
      |
      v
Encode Password
      |
      v
UserRepo.save(user)
      |
      v
Database
```

Your controller accepts the user details:

```java
@PostMapping("/register")
public Users register(@RequestBody Users user) {
    return service.register(user);
}
```

The service encodes the password and persists the user.

Important production considerations:

- Never return the stored password hash in an API response.
- Validate user input.
- Enforce username uniqueness.
- Avoid exposing sensitive user fields through entity serialization.
- Configure authorization for registration endpoints appropriately.

## 12. Stateful vs Stateless Authentication

### Stateful

The server maintains authentication state, commonly using an HTTP session.

Typical flow:

```text
Login
  |
  v
Server authenticates user
  |
  v
Security context stored in session
  |
  v
Session identifier sent in a cookie
  |
  v
Browser sends cookie on later requests
  |
  v
Server retrieves authentication state
```

Advantages:

- Convenient for traditional web applications.
- The server can invalidate a session.
- Works naturally with session-based form login.

Disadvantages:

- The server must maintain session state.
- Distributed applications may need shared session storage or session affinity.

### Stateless

The server does not use an HTTP session to maintain the security context between requests.

Each request must carry credentials or a token that allows the server to authenticate the request again.

Common examples include APIs using bearer tokens.

Advantages:

- Easier to scale without sharing authentication sessions.
- Appropriate for many APIs and service-to-service communication.

Disadvantages:

- Each request must provide valid authentication information.
- Token expiration, revocation, and refresh need to be designed appropriately.

### Spring configuration

```java
http.sessionManagement(session ->
        session.sessionCreationPolicy(
                SessionCreationPolicy.STATELESS
        )
);
```

`STATELESS` tells Spring Security not to create an HTTP session for storing the security context and not to use an existing session to obtain it.

**Important:** Stateless does not automatically mean more secure. It is a session-management choice, not a replacement for authentication, authorization, CSRF protection, or secure token handling.

## 13. Form Login vs HTTP Basic

### Form login

```java
http.formLogin(Customizer.withDefaults());
```

Spring Security provides a default login form unless you configure a custom one.

This is commonly used for browser-based applications and typically works with session-based authentication.

### HTTP Basic

```java
http.httpBasic(Customizer.withDefaults());
```

The client sends credentials using the HTTP `Authorization` header with the Basic scheme.

Postman can be configured to send HTTP Basic credentials.

Basic authentication does not itself create a login session. The client generally sends credentials on each request.

Use HTTPS because Basic credentials are only Base64-encoded, not encrypted by the Basic scheme itself.

Form login and HTTP Basic can be enabled together, but the correct combination depends on your application. Setting `STATELESS` while using form login changes the expected session behavior; for a typical form-login website, session-based configuration is more appropriate.

## 14. Cross-Site Request Forgery (CSRF)

CSRF is an attack in which a malicious website tricks a user's browser into sending an unwanted request to another website where the user is already authenticated.

### Example

Suppose you are logged in to a banking website.

Your browser holds a session cookie for that website.

You visit a malicious website in another tab. That website attempts to submit an unwanted request to the banking website.

Because browsers can automatically attach applicable cookies to requests, the banking server may receive the request with your existing session cookie.

Without suitable protection, the server might mistake the request for one intentionally made by you.

### How a CSRF token helps

A CSRF token is an unpredictable value that the server expects on requests that can change state.

The general process is:

1. The application makes a CSRF token available to the legitimate client.
2. The client includes that token in a submitted form or request header.
3. Spring Security validates the token on protected requests.
4. A request with a missing or invalid token is rejected.

The malicious website may cause the browser to send a request, but it generally cannot read the legitimate site's token because of browser same-origin restrictions.

This makes it harder to forge an authenticated request.

### CSRF token vs session cookie

These serve different purposes.

- **Session cookie:** Identifies the user's session.
- **CSRF token:** Helps verify that a state-changing request originated from a client that could obtain the expected token.

A CSRF token is not a replacement for the session cookie.

### Where is the token created and handled?

Spring Security provides CSRF token support when CSRF protection is enabled.

In a conventional server-rendered form, the token is included as a hidden form field.

For a JavaScript client, an application may expose the token and have the client send it in a configured request header.

Spring Security also provides mechanisms for exposing the token through request attributes. Your example:

```java
@GetMapping("/csrf-token")
public CsrfToken getCsrfToken(HttpServletRequest request) {
    return (CsrfToken) request.getAttribute("_csrf");
}
```

demonstrates one way to access the token for learning purposes.

The exact token handling depends on the Spring Security version and configuration. In current versions, token loading can be deferred, and the application must ensure that its chosen client integration actually obtains and sends the token.

### Where should the client send it?

For example, a JavaScript client might send:

```http
X-CSRF-TOKEN: <token-value>
```

The actual header name must match the application's Spring Security configuration and token handling.

A server-rendered form commonly submits the token as a hidden form field instead.

### Why CSRF is important

CSRF protection helps prevent a third-party site from using the victim's existing browser authentication to perform unwanted state-changing actions.

It is especially important when authentication credentials, such as session cookies, are automatically attached by the browser.

### Does stateless authentication eliminate CSRF?

No, not automatically.

CSRF risk depends on how credentials are transmitted. Browser-automatically-sent cookies can create CSRF risk even if an application describes itself as stateless.

A bearer token explicitly added to the `Authorization` header by application code is generally less exposed to traditional CSRF attacks, because a malicious site normally cannot read the token and add it to the request. However, token storage and XSS risks still need attention.

### Should CSRF be disabled?

Your configuration:

```java
http.csrf(customizer -> customizer.disable());
```

disables CSRF protection for the configured security chain.

This can be appropriate for certain APIs that use non-automatically-sent bearer tokens and have a carefully considered security design. It should not be disabled merely because you want to implement authentication yourself.

For a normal browser application using session cookies and form login, keep CSRF protection enabled unless you have a well-understood alternative.

## 15. Authorization Rules

Your example:

```java
http.authorizeHttpRequests(request ->
        request.anyRequest().authenticated()
);
```

means every request matching this security chain requires authentication.

You can define more specific rules:

```java
http.authorizeHttpRequests(request -> request
        .requestMatchers("/register").permitAll()
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
);
```

Here:

- `permitAll()` allows access without authentication.
- `authenticated()` requires an authenticated user.
- `hasRole("ADMIN")` checks for the `ROLE_ADMIN` authority.
- `hasAuthority("REPORT_READ")` checks for that exact authority.

Rule ordering matters: place more specific rules before broad catch-all rules.

## 16. Authentication vs Authorization

| Authentication                        | Authorization                      |
| ------------------------------------- | ---------------------------------- |
| Verifies identity                     | Checks permissions                 |
| Is the user valid?                    | Can this user perform this action? |
| Username/password or token validation | Role/authority checks              |
| Produces an authenticated identity    | Allows or denies access            |

A user may be authenticated but still not be authorized to access a particular resource.

## 17. Important Security Concepts to Learn Next

### Authentication and authorization failures

- `401 Unauthorized`: Authentication is missing or invalid in common API scenarios.
- `403 Forbidden`: The request is understood, but access is denied. This can occur due to insufficient permissions or a CSRF failure.

Exact responses depend on the configured authentication mechanism and exception handling.

### PasswordEncoder

Understand encoding, password matching, and why plain-text password storage is unsafe.

### Roles and authorities

Understand `GrantedAuthority`, `SimpleGrantedAuthority`, `hasRole()`, and `hasAuthority()`.

### Exception handling

Understand how Spring Security handles authentication failures and access-denied situations, including `AuthenticationEntryPoint` and `AccessDeniedHandler`.

### CORS

Cross-Origin Resource Sharing controls which browser origins are permitted to make cross-origin requests. CORS and CSRF solve different problems.

### JWT and OAuth 2.0

After understanding session-based authentication and the security filter chain, learn bearer-token authentication, JWT validation, and the purpose of OAuth 2.0 and OpenID Connect.

### Method-level security

Learn how to protect service methods using annotations such as `@PreAuthorize`, after enabling method security.

## 18. Overall Mental Model

```text
Request
   |
   v
Spring Security Filter Chain
   |
   v
Authentication
   |
   v
UserDetailsService
   |
   v
AuthenticationProvider
   |
   v
Password Verification
   |
   v
Authorization
   |
   v
DispatcherServlet
   |
   v
Controller → Service → Repository → Database
```

This is a simplified mental model. Authentication may already exist in a session or may be established through another mechanism, and not every request performs a username/password lookup.

The key idea is that Spring Security intercepts requests, establishes or retrieves authentication, and enforces the applicable access rules before allowing protected operations to proceed.
