# TLS Termination Guide for Azure App Service

---

## What is TLS Termination at Proxy/Ingress Level?

```
Client (HTTPS) → [Proxy/Ingress: decrypts SSL] → Your App (plain HTTP)
```

- **The platform handles all HTTPS/SSL** — certificates, encryption, renewal.
- **Your app may or may not receive plain HTTP** — depends on the hosting model.
- **You do NOT need** to configure SSL certificates inside your app.

> Think of it like a security guard at the building entrance — visitors (clients) show credentials (HTTPS) to the guard (proxy). Inside the building, everyone moves freely (HTTP).

---

## Azure App Service — Three Scenarios

Azure App Service behaves **differently** depending on how your app is hosted.  
Understanding the hosting model explains why the same code works in one scenario but fails in another.

---

## Scenario 1: Code Deployment — IIS In-Process Model

### How It Works

```
Internet
   │ HTTPS (TLS 1.2+)
   ▼
Azure Front End (ARR Proxy)
   │ Terminates TLS externally
   │ Forwards to IIS
   ▼
IIS (w3wp.exe worker process)
   │ In-Process: Your app runs INSIDE IIS worker process
   │ IIS natively preserves original HTTPS scheme
   │ No separate proxy hop
   ▼
Your ASP.NET Core App
   │ request.Scheme = "https"  ← Already correct!
   │ No forwarded headers needed
```

- Your app is hosted **inside the IIS worker process** (`w3wp.exe`) via **ASP.NET Core Module (ANCM)**.
- IIS is **not an external proxy** — it is the **same process** as your app.
- IIS natively passes the **original HTTPS scheme** directly to your app.
- `request.Scheme` is already `"https"` — `UseHttpsRedirection()` sees HTTPS and **never triggers a redirect**.

### Awareness

| Topic | Detail |
|-------|--------|
| Hosting | App runs **inside** IIS worker process |
| Scheme your app sees | `"https"` — preserved natively by IIS |
| `UseHttpsRedirection()` | ✅ Safe — no redirect triggered |
| `UseForwardedHeaders()` | ❌ Not required |
| `X-Forwarded-Proto` | Present but not needed |
| `X-ARR-SSL` | Present (App Service specific) |
| Risk | ⚠️ Switching to Out-of-Process will break this |

### web.config

```xml
<!-- ✅ In-Process (default for code deployment) -->
<aspNetCore processPath="dotnet"
            arguments=".\MyApi.dll"
            hostingModel="InProcess"
            stdoutLogEnabled="false" />
```

### Code: Wrong vs Good

```csharp
// ❌ "Wrong by cloud-native standards" but technically works here
// Only safe because IIS In-Process preserves https scheme natively
// Will BREAK if you switch to Out-of-Process or Container deployment

var app = builder.Build();

// UseHttpsRedirection() checks request.Scheme
// IIS passes "https" → condition is false → no redirect triggered
// Works but is NOT portable
app.UseHttpsRedirection();

app.MapControllers();
app.Run();
```

```csharp
// ✅ Good — works now AND portable to other hosting models
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor |
                               ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Clear();
    options.KnownProxies.Clear();
});

var app = builder.Build();

app.UseForwardedHeaders(); // ✅ Safe to add even for In-Process

if (app.Environment.IsDevelopment())
{
    app.UseHttpsRedirection();
}

app.MapControllers();
app.Run();
```

> ✅ **Why it "works" without fix:** IIS In-Process is not a proxy hop — your app sees HTTPS directly.  
> ⚠️ **Why you should still fix it:** The moment you switch to Out-of-Process or containers, it breaks.

---

## Scenario 2: Code Deployment — IIS Out-of-Process Model

### How It Works

```
Internet
   │ HTTPS (TLS 1.2+)
   ▼
Azure Front End (ARR Proxy)
   │ Terminates TLS
   │ Adds X-Forwarded-Proto: https
   │ Adds X-Forwarded-For: <client-ip>
   │ Adds X-ARR-SSL: <ssl-info>
   │ HTTP (plain)
   ▼
IIS (reverse proxy role)
   │ Forwards HTTP request to Kestrel
   │ Out-of-Process: Your app runs in a SEPARATE process
   │ HTTP (plain)
   ▼
Kestrel (your ASP.NET Core process)
   │ request.Scheme = "http"  ← Sees HTTP, not HTTPS!
   │ UseHttpsRedirection() triggers → redirect loop!
```

- Your app runs in a **separate process** (Kestrel) outside of IIS.
- IIS acts as a **reverse proxy** between ARR and Kestrel.
- Your app receives **plain HTTP** — `request.Scheme = "http"`.
- `UseHttpsRedirection()` **triggers a redirect** → **infinite loop**.

### Awareness

| Topic | Detail |
|-------|--------|
| Hosting | App runs in **separate Kestrel process** outside IIS |
| Scheme your app sees | `"http"` — downgraded by IIS proxy hop |
| `UseHttpsRedirection()` | ❌ Causes redirect loop |
| `UseForwardedHeaders()` | ✅ Required — same as Container deployment |
| `X-Forwarded-Proto` | Must be read to detect original HTTPS |
| `X-ARR-SSL` | Present but use `X-Forwarded-Proto` for portability |
| Risk | ⚠️ Easy to miss — same code deploy, different behavior |

### web.config

```xml
<!-- ⚠️ Out-of-Process — requires forwarded headers in your code -->
<aspNetCore processPath="dotnet"
            arguments=".\MyApi.dll"
            hostingModel="OutOfProcess"
            stdoutLogEnabled="false" />
```

### Code: Wrong vs Good

```csharp
// ❌ Wrong — redirect loop in Out-of-Process
// IIS is now a proxy hop → app sees HTTP
// UseHttpsRedirection() sees HTTP → redirects to HTTPS → loop forever

var app = builder.Build();

app.UseHttpsRedirection(); // ❌ Infinite redirect loop!

app.MapControllers();
app.Run();
```

```csharp
// ✅ Good — configure forwarded headers, same as container deployment
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor |
                               ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Clear();  // Trust IIS / ARR proxy
    options.KnownProxies.Clear();
});

var app = builder.Build();

// ✅ Must be FIRST — reads X-Forwarded-Proto: https
app.UseForwardedHeaders();

// ✅ Only redirect in local development
if (app.Environment.IsDevelopment())
{
    app.UseHttpsRedirection();
}

app.MapControllers();
app.Run();
```

---

## Scenario 3: Container Deployment

### How It Works

```
Internet
   │ HTTPS (TLS 1.2+)
   ▼
Azure Front End (ARR Proxy)
   │ Terminates TLS
   │ Adds X-Forwarded-Proto: https
   │ Adds X-Forwarded-For: <client-ip>
   │ Adds X-ARR-SSL: <ssl-info>
   │ HTTP (plain)
   ▼
Azure Container Runtime
   │ Routes HTTP to your container
   │ HTTP (plain)
   ▼
Your ASP.NET Core Container (PORT env variable)
   │ request.Scheme = "http"  ← Sees HTTP, not HTTPS!
   │ UseHttpsRedirection() triggers → redirect loop!
```

- Your app runs in an **isolated container** — completely outside IIS.
- No IIS involvement at all.
- Your container receives **plain HTTP** on the `PORT` environment variable.
- Behavior is **identical to Out-of-Process** — `UseForwardedHeaders()` is required.

### Awareness

| Topic | Detail |
|-------|--------|
| Hosting | Isolated Docker container — no IIS |
| Scheme your app sees | `"http"` — TLS terminated externally |
| `UseHttpsRedirection()` | ❌ Causes redirect loop |
| `UseForwardedHeaders()` | ✅ Required |
| Port | Must read from `PORT` environment variable |
| `X-Forwarded-Proto` | Must be read to detect original HTTPS |
| SSL in Dockerfile | ❌ Never configure SSL inside container |

### Code: Wrong vs Good

```csharp
// ❌ Wrong — no forwarded headers, redirect loop
var app = builder.Build();

app.UseHttpsRedirection(); // ❌ Loops forever

app.MapControllers();
app.Run();
```

```csharp
// ✅ Good — forwarded headers + read PORT from environment
var port = Environment.GetEnvironmentVariable("PORT") ?? "8080";
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(int.Parse(port)); // ✅ Read PORT env var
});

builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor |
                               ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Clear();  // Trust ARR proxy
    options.KnownProxies.Clear();
});

var app = builder.Build();

app.UseForwardedHeaders(); // ✅ Must be FIRST

if (app.Environment.IsDevelopment())
{
    app.UseHttpsRedirection();
}

app.MapControllers();
app.Run();
```

#### Dockerfile: Wrong vs Good

```dockerfile
# ❌ Wrong — HTTPS inside container, wrong port
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_URLS=https://+:443   # ❌ HTTPS inside container
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

```dockerfile
# ✅ Good — HTTP only, reads PORT from environment
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_URLS=http://+:8080   # ✅ HTTP only
EXPOSE 8080
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

---

## Full Comparison: All Three Scenarios

### Architecture Summary

| | **Code: In-Process** | **Code: Out-of-Process** | **Container** |
|--|---------------------|------------------------|--------------|
| **Runs inside** | IIS worker process | Separate Kestrel process | Isolated Docker container |
| **IIS role** | Host (same process) | Reverse proxy | Not involved |
| **Scheme app sees** | `"https"` ✅ | `"http"` ⚠️ | `"http"` ⚠️ |
| **TLS terminated at** | ARR (but IIS preserves scheme) | ARR + IIS proxy hop | ARR |
| **Proxy hops** | 1 (ARR only, transparent) | 2 (ARR → IIS → Kestrel) | 1 (ARR → Container) |

### Configuration Requirements

| | **Code: In-Process** | **Code: Out-of-Process** | **Container** |
|--|---------------------|------------------------|--------------|
| **`UseForwardedHeaders()`** | Optional (but recommended) | ✅ Required | ✅ Required |
| **Clear `KnownNetworks`** | Optional | ✅ Required | ✅ Required |
| **`UseHttpsRedirection()`** | ✅ Safe (but move to dev only) | ❌ Causes loop | ❌ Causes loop |
| **Read `PORT` env var** | ❌ Not needed | ❌ Not needed | ✅ Required |
| **`hostingModel` in web.config** | `InProcess` | `OutOfProcess` | N/A |
| **SSL in app** | ❌ Not needed | ❌ Not needed | ❌ Not needed |

### Risk Level Without Proper Config

| | **Code: In-Process** | **Code: Out-of-Process** | **Container** |
|--|---------------------|------------------------|--------------|
| **Risk without fix** | 🟡 Low — works accidentally | 🔴 High — redirect loop | 🔴 High — redirect loop |
| **Portability** | ❌ Breaks if model changes | ✅ Portable | ✅ Portable |
| **Recommended fix** | Add forwarded headers proactively | Add forwarded headers | Add forwarded headers |

---

## One Config That Works Across All Three Scenarios

```csharp name=Program.cs
var builder = WebApplication.CreateBuilder(args);

// ✅ Read PORT — required for container, harmless for code deploy
var port = Environment.GetEnvironmentVariable("PORT") ?? "8080";
builder.WebHost.ConfigureKestrel(options =>
{
    options.ListenAnyIP(int.Parse(port));
});

// ✅ Configure forwarded headers — required for Out-of-Process & Container
builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor |
                               ForwardedHeaders.XForwardedProto;
    options.KnownNetworks.Clear();
    options.KnownProxies.Clear();
});

builder.Services.AddControllers();

var app = builder.Build();

// ✅ Must be FIRST
app.UseForwardedHeaders();

// ✅ Only redirect in local development — platform handles HTTPS
if (app.Environment.IsDevelopment())
{
    app.UseHttpsRedirection();
}

app.UseAuthorization();
app.MapControllers();
app.Run();
```

> This single configuration is **safe and portable** across all three App Service hosting models AND Azure Container Apps AND Google Cloud Run.

---

## Quick Decision Guide

```
Code deployment + InProcess (default)?
  → Works without fix, but add UseForwardedHeaders() for portability

Code deployment + OutOfProcess?
  → UseForwardedHeaders() is required — redirect loop without it

Container deployment?
  → UseForwardedHeaders() is required + read PORT env var

Not sure which model you're using?
  → Always add UseForwardedHeaders() — it is safe in all scenarios
```