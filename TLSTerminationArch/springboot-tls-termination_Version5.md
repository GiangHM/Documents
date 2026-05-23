# TLS Termination Guide for Java Spring Boot

---

## What is TLS Termination at Proxy/Ingress Level?

```
Client (HTTPS) → [Proxy/Ingress: decrypts SSL] → Your Container (plain HTTP)
```

- **The platform (Azure / Cloud Run) handles all HTTPS/SSL** — certificates, encryption, renewal.
- **Your container only receives plain HTTP** internally.
- **You do NOT need** to configure SSL/TLS inside your Spring Boot app.

> Think of it like a security guard at the building entrance — visitors (clients) show credentials (HTTPS) to the guard (proxy). Inside the building, everyone moves freely (HTTP).

---

## Azure Container Apps

### How It Applies the Architecture

```
Internet
   │ HTTPS (TLS 1.2+)
   ▼
Azure Container Apps Ingress Controller
   │ Terminates TLS
   │ Adds X-Forwarded-Proto: https
   │ Adds X-Forwarded-For: <client-ip>
   │ HTTP (plain)
   ▼
Your Spring Boot Container (port 8080)
```

- Azure provisions and renews **managed TLS certificates** automatically.
- Ingress forwards requests to your container over **plain HTTP**.
- Azure attaches **forwarded headers** so your app knows the original request was HTTPS.

### Awareness

| Topic | Detail |
|-------|--------|
| Your app receives | Plain **HTTP** on the port you configure |
| Forwarded headers | `X-Forwarded-Proto: https`, `X-Forwarded-For: <ip>` |
| Port | Configured via `--target-port` in Azure CLI |
| `requiresSecure()` risk | Redirect loop if forwarded headers are not configured |
| `forward-headers-strategy` | **Must be set to `framework`** in `application.yml` |
| Spring Security | Do NOT call `requiresChannel().requiresSecure()` without forwarded headers |
| HATEOAS links | Will use `http://` instead of `https://` without forwarded headers |
| `request.getScheme()` | Returns `"http"` (wrong) without forwarded headers configured |

---

### Code: Wrong vs Good

#### application.yml: Wrong vs Good

```yaml
# ❌ Wrong — no forwarded headers strategy, SSL enabled in container
server:
  port: 8080
  ssl:
    enabled: true                      # ❌ Don't configure SSL in container
    key-store: classpath:keystore.p12
    key-store-password: changeit
# Missing: forward-headers-strategy
```

```yaml
# ✅ Good — forwarded headers enabled, no SSL in container
server:
  port: ${PORT:8080}                   # ✅ Read PORT from environment
  forward-headers-strategy: framework  # ✅ Trust X-Forwarded-Proto header
  ssl:
    enabled: false                     # ✅ Platform handles TLS

spring:
  application:
    name: myapi
```

#### Spring Security: Wrong vs Good

```java
// ❌ Wrong — forces HTTPS redirect without forwarded headers
// Causes redirect loop in Azure Container Apps
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .requiresChannel(channel -> channel
                .anyRequest().requiresSecure() // ❌ Redirect loop!
            );
        return http.build();
    }
}
```

```java
// ✅ Good — platform handles HTTPS, no channel redirect needed
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated()
            );
            // ✅ No requiresSecure() — Azure ingress handles HTTPS

        return http.build();
    }
}
```

#### ForwardedHeaderFilter (Programmatic): Wrong vs Good

```java
// ❌ Wrong — no ForwardedHeaderFilter registered
// request.getScheme() returns "http" even for HTTPS requests
// HATEOAS links, redirects, and OAuth2 callbacks will use http://
@Configuration
public class WebConfig {
    // Nothing configured for forwarded headers
}
```

```java
// ✅ Good — ForwardedHeaderFilter registered with highest priority
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean<ForwardedHeaderFilter> forwardedHeaderFilter() {
        FilterRegistrationBean<ForwardedHeaderFilter> bean =
            new FilterRegistrationBean<>();
        bean.setFilter(new ForwardedHeaderFilter());
        bean.setOrder(0); // ✅ Must be first filter in chain
        return bean;
    }
}
```

---

## Google Cloud Run

### How It Applies the Architecture

```
Internet
   │ HTTPS (TLS 1.2+)
   ▼
Cloud Run Frontend (Google-managed proxy)
   │ Terminates TLS
   │ Adds X-Forwarded-Proto: https
   │ Adds X-Forwarded-For: <client-ip>
   │ HTTP (plain)
   ▼
Your Spring Boot Container (PORT env variable)
```

- Google provisions and renews **Google-managed TLS certificates** automatically.
- Cloud Run injects a **`PORT` environment variable** (default: `8080`) — your app must listen on it.
- Cloud Run attaches **forwarded headers** so your app knows the original protocol.

### Awareness

| Topic | Detail |
|-------|--------|
| Your app receives | Plain **HTTP** on `PORT` env variable |
| Forwarded headers | `X-Forwarded-Proto: https`, `X-Forwarded-For: <ip>` |
| Port | **Must read from `PORT` environment variable** |
| `requiresSecure()` risk | Redirect loop if forwarded headers are not configured |
| `forward-headers-strategy` | **Must be set to `framework`** in `application.yml` |
| Spring Security | Do NOT call `requiresChannel().requiresSecure()` without forwarded headers |
| HATEOAS links | Will generate `http://` URLs without forwarded headers |
| Reactive (WebFlux) | Same `forward-headers-strategy: framework` applies |

---

### Code: Wrong vs Good

#### application.yml: Wrong vs Good

```yaml
# ❌ Wrong — hardcoded port, SSL enabled, no forwarded headers
server:
  port: 8443                            # ❌ Hardcoded — ignores PORT env var
  ssl:
    enabled: true                       # ❌ Don't configure SSL in container
    key-store: classpath:keystore.p12
    key-store-password: changeit
```

```yaml
# ✅ Good — reads PORT from Cloud Run, forwarded headers enabled
server:
  port: ${PORT:8080}                    # ✅ Reads Cloud Run PORT env variable
  forward-headers-strategy: framework   # ✅ Trust X-Forwarded-Proto header
  ssl:
    enabled: false                      # ✅ Cloud Run handles TLS

spring:
  application:
    name: myapi
```

#### Spring Security: Wrong vs Good

```java
// ❌ Wrong — forces HTTPS redirect without forwarded headers
// Causes redirect loop in Cloud Run
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .requiresChannel(channel -> channel
                .anyRequest().requiresSecure() // ❌ Redirect loop!
            );
        return http.build();
    }
}
```

```java
// ✅ Good — platform handles HTTPS, no channel redirect needed
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated()
            );
            // ✅ No requiresSecure() — Cloud Run handles HTTPS

        return http.build();
    }
}
```

#### Dockerfile: Wrong vs Good

```dockerfile
# ❌ Wrong — hardcoded port, ignores PORT env variable from Cloud Run
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 443                              # ❌ Wrong port
ENV SERVER_PORT=443                     # ❌ Ignores Cloud Run PORT
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```dockerfile
# ✅ Good — uses PORT env variable, HTTP only
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080                             # ✅ Default port
# PORT env variable is injected by Cloud Run at runtime
ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-jar", "app.jar"]
```

---

## Comparison: Azure Container Apps vs Google Cloud Run

### Summary

Both platforms use the same TLS termination pattern — **`server.forward-headers-strategy=framework` is the key setting for both**.  
The difference is in **networking model, pricing, and ecosystem**.

### Comparison Table

| Feature | Azure Container Apps | Google Cloud Run |
|---------|---------------------|-----------------|
| **TLS termination** | ✅ Ingress Controller | ✅ Cloud Run Proxy |
| **Managed certificates** | ✅ Auto (Let's Encrypt) | ✅ Auto (Google-managed) |
| **Forwarded headers** | `X-Forwarded-Proto`, `X-Forwarded-For` | `X-Forwarded-Proto`, `X-Forwarded-For` |
| **Port configuration** | `--target-port` in CLI | `PORT` environment variable (required) |
| **Spring Boot key config** | `forward-headers-strategy: framework` | `forward-headers-strategy: framework` |
| **Spring Security risk** | Redirect loop without forwarded headers | Redirect loop without forwarded headers |
| **HATEOAS links** | Wrong `http://` without forwarded headers | Wrong `http://` without forwarded headers |
| **Scale to zero** | ✅ Configurable | ✅ Default behavior |
| **Pricing model** | Per vCPU/memory per second | Per request + CPU/memory |
| **Internal networking** | Shared environment (VNET) | VPC via connector |
| **Multi-service environment** | ✅ Built-in (same environment) | Individual services |
| **Dapr support** | ✅ Native | ❌ Manual setup |
| **Custom domains** | ✅ Yes | ✅ Yes |
| **gRPC / HTTP2** | ✅ Via `--transport http2` | ✅ Automatic |
| **Ecosystem** | Azure / Microsoft | Google Cloud |

### Quick Decision Guide

```
Using Azure ecosystem already?      → Azure Container Apps
Using Google Cloud already?         → Cloud Run
Need Dapr / service mesh?           → Azure Container Apps
Need true scale-to-zero (savings)?  → Cloud Run
Multi-container shared network?     → Azure Container Apps
Simple stateless Spring Boot API?   → Either (both work great)
```