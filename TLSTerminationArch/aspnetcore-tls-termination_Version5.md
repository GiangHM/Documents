# TLS Termination Guide for ASP.NET Core

---

## What is TLS Termination at Proxy/Ingress Level?

```
Client (HTTPS) → [Proxy/Ingress: decrypts SSL] → Your Container (plain HTTP)
```

- **The platform (Azure / Cloud Run) handles all HTTPS/SSL** — certificates, encryption, renewal.
- **Your container only receives plain HTTP** internally.
- **You do NOT need** to configure SSL certificates or HTTPS inside your app.

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
Your ASP.NET Core Container (port 8080)
```

- Azure provisions and renews **managed TLS certificates** automatically.
- Ingress forwards requests to your container over **plain HTTP**.
- Azure attaches **forwarded headers** so your app knows the original request was HTTPS.

### Awareness

| Topic | Detail |
|-------|--------|
| Your app receives | Plain **HTTP** on the port you specify |
| Forwarded headers | `X-Forwarded-Proto: https`, `X-Forwarded-For: <ip>` |
| Port | Configured via `--target-port` in Azure CLI |
| `UseHttpsRedirection()` risk | Redirect loop if forwarded headers are not configured |
| `UseForwardedHeaders()` | **Must be called first** before any HTTPS middleware |
| `KnownNetworks` / `KnownProxies` | Must be cleared to trust Azure ingress |

---

### Code: Wrong vs Good

#### ❌ Wrong — Redirect Loop Risk

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

// ❌ No forwarded headers configured
// ❌ App always sees HTTP → always redirects → infinite loop
app.UseHttpsRedirection();

app.UseAuthorization();
app.MapControllers();
app.Run();
```

#### ✅ Good — Forwarded Headers Configured

```csharp
var builder = WebApplication.CreateBuilder(args);

// ✅ Read port from environment variable
var port = Environment.GetEnvironmentVariable("PORT") ?? "8080";
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(int.Parse(port));
});

// ✅ Configure forwarded headers to trust Azure ingress
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor |
                               ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Clear();  // Trust Azure ingress proxy
    options.KnownProxies.Clear();
});

builder.Services.AddControllers();

var app = builder.Build();

// ✅ Must be FIRST — reads X-Forwarded-Proto: https
app.UseForwardedHeaders();

// ✅ Only redirect in local development
if (app.Environment.IsDevelopment())
{
    app.UseHttpsRedirection();
}

app.UseAuthorization();
app.MapControllers();
app.Run();
```

#### application-settings: Wrong vs Good

```json
// ❌ Wrong — hardcoded HTTPS port, SSL enabled in container
{
  "Kestrel": {
    "Endpoints": {
      "Https": {
        "Url": "https://0.0.0.0:443"
      }
    }
  }
}
```

```json
// ✅ Good — plain HTTP, port from environment
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://0.0.0.0:8080"
      }
    }
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
Your ASP.NET Core Container (PORT env variable)
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
| `UseHttpsRedirection()` risk | Redirect loop if forwarded headers are not configured |
| `UseForwardedHeaders()` | **Must be called first** before any HTTPS middleware |
| `KnownNetworks` / `KnownProxies` | Must be cleared to trust Cloud Run proxy |

---

### Code: Wrong vs Good

#### ❌ Wrong — Hardcoded Port & Redirect Loop

```csharp
var builder = WebApplication.CreateBuilder(args);

// ❌ Hardcoded port — Cloud Run uses dynamic PORT env variable
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(5000); // Wrong port!
});

builder.Services.AddControllers();

var app = builder.Build();

// ❌ No forwarded headers — redirect loop will occur
app.UseHttpsRedirection();

app.UseAuthorization();
app.MapControllers();
app.Run();
```

#### ✅ Good — PORT Variable & Forwarded Headers

```csharp
var builder = WebApplication.CreateBuilder(args);

// ✅ Read PORT from Cloud Run environment variable
var port = Environment.GetEnvironmentVariable("PORT") ?? "8080";
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(int.Parse(port));
});

// ✅ Configure forwarded headers to trust Cloud Run proxy
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor |
                               ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Clear();  // Trust Cloud Run proxy
    options.KnownProxies.Clear();
});

builder.Services.AddControllers();

var app = builder.Build();

// ✅ Must be FIRST — reads X-Forwarded-Proto: https
app.UseForwardedHeaders();

// ✅ Only redirect in local development
if (app.Environment.IsDevelopment())
{
    app.UseHttpsRedirection();
}

app.UseAuthorization();
app.MapControllers();
app.Run();
```

#### Dockerfile: Wrong vs Good

```dockerfile
# ❌ Wrong — wrong port, ASPNETCORE_URLS uses HTTPS
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_URLS=https://+:443   # ❌ HTTPS inside container
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

```dockerfile
# ✅ Good — HTTP only, correct port
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_URLS=http://+:8080   # ✅ HTTP inside container
EXPOSE 8080
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

---

## Comparison: Azure Container Apps vs Google Cloud Run

### Summary

Both platforms use the same TLS termination pattern — **your code is nearly identical** on both.  
The difference is in **networking model, pricing, and ecosystem**.

### Comparison Table

| Feature | Azure Container Apps | Google Cloud Run |
|---------|---------------------|-----------------|
| **TLS termination** | ✅ Ingress Controller | ✅ Cloud Run Proxy |
| **Managed certificates** | ✅ Auto (Let's Encrypt) | ✅ Auto (Google-managed) |
| **Forwarded headers** | `X-Forwarded-Proto`, `X-Forwarded-For` | `X-Forwarded-Proto`, `X-Forwarded-For` |
| **Port configuration** | `--target-port` in CLI | `PORT` environment variable |
| **ASP.NET Core config change** | `UseForwardedHeaders()` | `UseForwardedHeaders()` |
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
Using Azure ecosystem already?     → Azure Container Apps
Using Google Cloud already?        → Cloud Run
Need Dapr / service mesh?          → Azure Container Apps
Need true scale-to-zero (savings)? → Cloud Run
Multi-container shared network?    → Azure Container Apps
Simple stateless API?              → Either (both work great)
```