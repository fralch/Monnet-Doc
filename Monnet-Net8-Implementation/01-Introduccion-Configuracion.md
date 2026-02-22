# Implementación de Monnet Payments en .NET 8

## 📋 Parte 1: Introducción y Configuración Básica

## 1. Introducción 🚀

Guía completa para integrar la API de Monnet Payments en proyectos .NET 8, aprovechando las últimas características del framework para crear una integración robusta, segura y mantenible.

### ¿Qué es Monnet Payments?

Monnet Payments es una plataforma de pagos para Latinoamérica que ofrece:

- **Múltiples métodos de pago**: Tarjetas, transferencias, billeteras digitales, efectivo
- **Cobertura regional**: Argentina, Chile, Perú, México, Colombia, Ecuador y más
- **Soluciones avanzadas**: Cuentas virtuales, suscripciones recurrentes
- **API moderna**: Basada en REST con autenticación segura

### Beneficios de usar .NET 8 con Monnet

| Característica | Beneficio |
|--------------|----------|
| **Rendimiento** | Uno de los frameworks más rápidos para APIs |
| **Seguridad** | Manejo nativo de criptografía (SHA-512, HMAC) |
| **Concurrencia** | Soporte nativo para operaciones asíncronas |
| **DI integrado** | Arquitectura limpia y mantenible |
| **Logging** | Integración fácil con Serilog y otros |

## 2. Requisitos Previos 🛠️

### Herramientas necesarias

```bash
# Verificar versión de .NET
dotnet --version
# Debería mostrar: 8.0.100 o superior
```

**Software requerido:**
- .NET 8 SDK ([Descargar](https://dotnet.microsoft.com/download/dotnet/8.0))
- Visual Studio 2022 (17.8+) o VS Code con extensión C#
- Postman o similar para pruebas de API

### Credenciales de Monnet

Contacta a tu representante para obtener:

1. **MerchantID**: Identificador único de tu comercio
2. **KeyMonnet**: Llave secreta para firmas estándar
3. **ApiKey**: Clave pública para APIs de cuentas virtuales
4. **SecretKey**: Llave secreta para cuentas virtuales

### Entornos

- **CERT (Pruebas)**: `https://cert.payin.api.monnetpayments.com/`
- **PROD (Producción)**: `https://payin.api.monnetpayments.com/`
- **Back Office CERT**: [https://cert.payin.monnetpayments.com/](https://cert.payin.monnetpayments.com/)

### Conocimientos recomendados

- C# y ASP.NET Core
- HTTP/HTTPS y REST APIs
- JSON y serialización
- Autenticación con firmas digitales
- Manejo de webhooks

## 3. Configuración del Proyecto 🔧

### Crear nuevo proyecto

```bash
# Crear proyecto Web API
dotnet new webapi -n MonnetPaymentIntegration
cd MonnetPaymentIntegration

# Agregar paquetes necesarios
dotnet add package Microsoft.Extensions.Http
dotnet add package Newtonsoft.Json
dotnet add package Polly
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File

# Restaurar paquetes
dotnet restore
```

### Estructura recomendada

```
MonnetPaymentIntegration/
├── Controllers/
│   ├── MonnetController.cs          # Endpoints para transacciones
│   └── WebhookController.cs         # Manejo de notificaciones
├── Services/
│   ├── MonnetAuthService.cs         # Autenticación y firmas
│   ├── MonnetTransactionService.cs  # Servicios de transacciones
│   ├── MonnetVirtualAccountService.cs # Cuentas virtuales
│   └── MonnetSubscriptionService.cs # Suscripciones
├── Models/
│   ├── Requests/
│   │   ├── TransactionRequest.cs
│   │   ├── VirtualAccountRequest.cs
│   │   └── SubscriptionRequest.cs
│   └── Responses/
│       ├── TransactionResponse.cs
│       ├── VirtualAccountResponse.cs
│       └── SubscriptionResponse.cs
├── DTOs/
│   └── MonnetNotification.cs        # Notificaciones entrantes
├── appsettings.json
├── appsettings.Development.json
├── Program.cs
└── MonnetPaymentIntegration.csproj
```

### Configurar csproj

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  
  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Http" Version="8.0.0" />
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
    <PackageReference Include="Polly" Version="7.2.3" />
    <PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
    <PackageReference Include="Serilog.Sinks.Console" Version="5.0.0" />
    <PackageReference Include="Serilog.Sinks.File" Version="5.0.0" />
  </ItemGroup>
</Project>
```

### Configuración de appsettings.json

```json
{
  "Monnet": {
    "MerchantId": "999999",
    "KeyMonnet": "tu_llave_secreta_para_firmas",
    "ApiKey": "mk_test_abc123xyz456",
    "SecretKey": "sk_test_secret123456789",
    "BaseUrl": "https://cert.payin.api.monnetpayments.com",
    "VirtualAccountBaseUrl": "https://cert-api.monnetpayments.com/merchant-payin-accounts/v1",
    "SubscriptionBaseUrl": "https://cert.subscriptions.payin.monnet.io/api/v1",
    "IsProduction": false,
    "TimeoutSeconds": 30,
    "MaxRetryAttempts": 3
  },
  
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.Hosting.Lifetime": "Information"
      }
    },
    "WriteTo": [
      {"Name": "Console"},
      {
        "Name": "File",
        "Args": {
          "path": "Logs/monnet-.log",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 7
        }
      }
    ]
  },
  
  "AllowedHosts": "*"
}
```

### Configuración de Program.cs

```csharp
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configurar Serilog
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .CreateLogger();

builder.Host.UseSerilog();

// Configurar HttpClients
builder.Services.AddHttpClient("Monnet", client =>
{
    client.BaseAddress = new Uri(builder.Configuration["Monnet:BaseUrl"]);
    client.Timeout = TimeSpan.FromSeconds(30);
});

builder.Services.AddHttpClient("MonnetVirtualAccounts", client =>
{
    client.BaseAddress = new Uri(builder.Configuration["Monnet:VirtualAccountBaseUrl"]);
    client.Timeout = TimeSpan.FromSeconds(30);
});

builder.Services.AddHttpClient("MonnetSubscriptions", client =>
{
    client.BaseAddress = new Uri(builder.Configuration["Monnet:SubscriptionBaseUrl"]);
    client.Timeout = TimeSpan.FromSeconds(30);
});

// Registrar servicios
builder.Services.AddSingleton<MonnetAuthService>(provider =>
    new MonnetAuthService(
        builder.Configuration["Monnet:MerchantId"],
        builder.Configuration["Monnet:KeyMonnet"],
        builder.Configuration["Monnet:ApiKey"],
        builder.Configuration["Monnet:SecretKey"]));

builder.Services.AddScoped<MonnetTransactionService>();
builder.Services.AddScoped<MonnetVirtualAccountService>();
builder.Services.AddScoped<MonnetSubscriptionService>();

// Configurar controladores
builder.Services.AddControllers()
    .AddNewtonsoftJson(options =>
    {
        options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore;
        options.SerializerSettings.NullValueHandling = Newtonsoft.Json.NullValueHandling.Ignore;
    });

// Configurar Swagger
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new() { Title = "Monnet Payment Integration API", Version = "v1" });
});

var app = builder.Build();

// Configurar pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## 4. Servicio de Autenticación 🔐

### MonnetAuthService.cs

```csharp
using System.Security.Cryptography;
using System.Text;

namespace MonnetPaymentIntegration.Services
{
    /// <summary>
    /// Servicio para manejo de autenticación y firmas con Monnet Payments
    /// </summary>
    public class MonnetAuthService
    {
        private readonly string _merchantId;
        private readonly string _keyMonnet;
        private readonly string _apiKey;
        private readonly string _secretKey;

        public string MerchantId => _merchantId;

        public MonnetAuthService(string merchantId, string keyMonnet, string apiKey = null, string secretKey = null)
        {
            _merchantId = merchantId ?? throw new ArgumentNullException(nameof(merchantId));
            _keyMonnet = keyMonnet ?? throw new ArgumentNullException(nameof(keyMonnet));
            _apiKey = apiKey;
            _secretKey = secretKey;
        }

        /// <summary>
        /// Genera firma SHA-512 para API estándar
        /// </summary>
        public string GenerateStandardSignature(string operationNumber, string amount, string currency)
        {
            if (string.IsNullOrEmpty(operationNumber))
                throw new ArgumentException("Operation number cannot be null or empty", nameof(operationNumber));
            
            if (string.IsNullOrEmpty(amount))
                throw new ArgumentException("Amount cannot be null or empty", nameof(amount));
            
            if (string.IsNullOrEmpty(currency))
                throw new ArgumentException("Currency cannot be null or empty", nameof(currency));

            string stringToHash = _merchantId + operationNumber + amount + currency + _keyMonnet;
            using var sha512 = SHA512.Create();
            byte[] bytes = sha512.ComputeHash(Encoding.UTF8.GetBytes(stringToHash));
            return BitConverter.ToString(bytes).Replace("-", "").ToLower();
        }

        /// <summary>
        /// Genera firma HMAC-SHA512 para Cuentas Virtuales
        /// </summary>
        public string GenerateHmacSignature(string message)
        {
            if (string.IsNullOrEmpty(_secretKey))
                throw new InvalidOperationException("SecretKey is not configured");

            if (string.IsNullOrEmpty(message))
                throw new ArgumentException("Message cannot be null or empty", nameof(message));

            using var hmac = new HMACSHA512(Encoding.UTF8.GetBytes(_secretKey));
            byte[] hashBytes = hmac.ComputeHash(Encoding.UTF8.GetBytes(message));
            return BitConverter.ToString(hashBytes).Replace("-", "").ToLower();
        }

        /// <summary>
        /// Genera headers para APIs de Cuentas Virtuales
        /// </summary>
        public Dictionary<string, string> GenerateVirtualAccountHeaders(string referenceId = null)
        {
            var timestamp = DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString();
            var message = $"{timestamp}{_apiKey}{referenceId}";
            var signature = GenerateHmacSignature(message);

            return new Dictionary<string, string>
            {
                { "X-Api-Key", _apiKey },
                { "X-Timestamp", timestamp },
                { "X-Signature", signature }
            };
        }

        /// <summary>
        /// Valida notificaciones entrantes
        /// </summary>
        public bool ValidateNotification(string merchantId, string operationNumber, 
                                        string amount, string currency, string verification)
        {
            if (string.IsNullOrEmpty(verification))
                return false;

            string expectedSignature = GenerateStandardSignature(operationNumber, amount, currency);
            return expectedSignature.Equals(verification, StringComparison.OrdinalIgnoreCase);
        }
    }
}
```

### Uso del servicio de autenticación

```csharp
// En tu controlador o servicio
var authService = new MonnetAuthService(
    "999999",
    "tu_key_monnet",
    "tu_api_key",
    "tu_secret_key");

// Generar firma para transacción
string signature = authService.GenerateStandardSignature(
    "ORD-12345",
    "100.00",
    "PEN");

// Generar headers para cuenta virtual
var headers = authService.GenerateVirtualAccountHeaders("CLIENTE-001");

// Validar notificación
bool isValid = authService.ValidateNotification(
    "999999",
    "ORD-12345",
    "100.00",
    "PEN",
    "firma_recibida");
```

## 🎯 Próximos Pasos

En el siguiente archivo (`02-Autenticacion-Servicios.md`) cubriremos:

1. **Modelos de transacción** (Request/Response)
2. **Servicio de transacciones completo** con manejo de errores
3. **Controladores para transacciones** con ejemplos prácticos
4. **Manejo de políticas de reintento** con Polly
5. **Logging avanzado** con Serilog

¿Te gustaría que continúe con la siguiente parte o prefieres revisar algún aspecto específico de esta primera parte? 😊