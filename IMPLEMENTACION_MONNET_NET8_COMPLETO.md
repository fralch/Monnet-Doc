# Guía Completa: Implementación de Monnet Payments API en .NET 8

¡Bienvenido a la guía definitiva para integrar la API de Monnet Payments en proyectos .NET 8! Esta guía te proporcionará una implementación paso a paso, desde la configuración inicial hasta la implementación completa de todas las funcionalidades clave.

## 📋 Tabla de Contenidos

1. [Introducción](#introducción)
2. [Requisitos Previos](#requisitos-previos)
3. [Configuración del Proyecto](#configuración-del-proyecto)
4. [Configuración de Autenticación](#configuración-de-autenticación)
5. [Implementación de Transacciones](#implementación-de-transacciones)
6. [Manejo de Webhooks](#manejo-de-webhooks)
7. [Cuentas Virtuales](#cuentas-virtuales)
8. [Yape One Shot y Suscripciones](#yape-one-shot-y-suscripciones)
9. [Manejo de Errores](#manejo-de-errores)
10. [Pruebas y Despliegue](#pruebas-y-despliegue)
11. [Ejemplo Completo](#ejemplo-completo)
12. [Buenas Prácticas](#buenas-prácticas)

## 1. Introducción 🚀

Esta guía te mostrará cómo implementar la API de Monnet Payments en una aplicación .NET 8, aprovechando las últimas características del framework para crear una integración robusta, segura y mantenible.

Monnet Payments ofrece una plataforma de pagos para Latinoamérica con múltiples métodos de pago, incluyendo tarjetas, transferencias bancarias, billeteras digitales y más.

## 2. Requisitos Previos 🛠️

Antes de comenzar, asegúrate de tener:

- **.NET 8 SDK** instalado ([Descargar .NET 8](https://dotnet.microsoft.com/download/dotnet/8.0))
- **Visual Studio 2022** (versión 17.8+) o **VS Code** con extensión de C#
- **Credenciales de Monnet**: `MerchantID`, `KeyMonnet`, `ApiKey` y `SecretKey`
- **Cuenta en entorno CERT** de Monnet para pruebas
- Conocimientos básicos de C# y ASP.NET Core

## 3. Configuración del Proyecto 🔧

### Crear un nuevo proyecto

```bash
# Crear un nuevo proyecto Web API
dotnet new webapi -n MonnetPaymentIntegration
cd MonnetPaymentIntegration

# Agregar paquetes necesarios
dotnet add package Microsoft.Extensions.Http
dotnet add package Newtonsoft.Json
dotnet add package Polly
dotnet add package Serilog.AspNetCore
```

### Estructura recomendada del proyecto

```
MonnetPaymentIntegration/
├── Controllers/
│   ├── MonnetController.cs
│   └── WebhookController.cs
├── Services/
│   ├── MonnetAuthService.cs
│   ├── MonnetTransactionService.cs
│   ├── MonnetVirtualAccountService.cs
│   └── MonnetSubscriptionService.cs
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
│   └── MonnetNotification.cs
├── appsettings.json
├── Program.cs
└── MonnetPaymentIntegration.csproj
```

## 4. Configuración de Autenticación 🔐

### Servicio de Autenticación

Crea el archivo `Services/MonnetAuthService.cs`:

```csharp
using System.Security.Cryptography;
using System.Text;

namespace MonnetPaymentIntegration.Services
{
    public class MonnetAuthService
    {
        private readonly string _merchantId;
        private readonly string _keyMonnet;
        private readonly string _apiKey;
        private readonly string _secretKey;

        public string MerchantId => _merchantId;

        public MonnetAuthService(string merchantId, string keyMonnet, string apiKey = null, string secretKey = null)
        {
            _merchantId = merchantId;
            _keyMonnet = keyMonnet;
            _apiKey = apiKey;
            _secretKey = secretKey;
        }

        /// <summary>
        /// Genera firma SHA-512 para API estándar de Monnet
        /// </summary>
        public string GenerateStandardSignature(string operationNumber, string amount, string currency)
        {
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
            string expectedSignature = GenerateStandardSignature(operationNumber, amount, currency);
            return expectedSignature.Equals(verification, StringComparison.OrdinalIgnoreCase);
        }
    }
}
```

### Configuración en appsettings.json

```json
{
  "Monnet": {
    "MerchantId": "tu_merchant_id",
    "KeyMonnet": "tu_llave_secreta_para_firmas",
    "ApiKey": "tu_api_key_para_cuentas_virtuales",
    "SecretKey": "tu_secret_key_para_cuentas_virtuales",
    "BaseUrl": "https://cert.payin.api.monnetpayments.com",
    "VirtualAccountBaseUrl": "https://cert-api.monnetpayments.com/merchant-payin-accounts/v1",
    "SubscriptionBaseUrl": "https://cert.subscriptions.payin.monnet.io/api/v1",
    "IsProduction": false
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

## 5. Implementación de Transacciones 💳

### Modelos de Transacción

Crea `Models/Requests/TransactionRequest.cs`:

```csharp
namespace MonnetPaymentIntegration.Models.Requests
{
    public class TransactionRequest
    {
        public string PayinMerchantID { get; set; }
        public string PayinAmount { get; set; }
        public string PayinCurrency { get; set; }
        public string PayinMerchantOperationNumber { get; set; }
        public string PayinProcessorCode { get; set; }
        public string PayinTransactionOKURL { get; set; }
        public string PayinTransactionErrorURL { get; set; }
        public string PayinCustomerTypeDocument { get; set; }
        public string PayinCustomerDocument { get; set; }
        public string PayinCustomerEmail { get; set; }?
        public string PayinCustomerName { get; set; }?
    }
}
```

Crea `Models/Responses/TransactionResponse.cs`:

```csharp
namespace MonnetPaymentIntegration.Models.Responses
{
    public class TransactionResponse
    {
        public string PayinID { get; set; }
        public string PayinURL { get; set; }
        public string Status { get; set; }
        public string Message { get; set; }
    }

    public class TransactionStatusResponse
    {
        public string PayinMerchantID { get; set; }
        public List<TransactionOperation> Operations { get; set; }
    }

    public class TransactionOperation
    {
        public string PayinStateID { get; set; }
        public string PayinState { get; set; }
        public string PayinAmount { get; set; }
        public string PayinCurrency { get; set; }
        public string PayinMerchantOperationNumber { get; set; }
        public string PayinID { get; set; }
    }
}
```

### Servicio de Transacciones

Crea `Services/MonnetTransactionService.cs`:

```csharp
using System.Net.Http.Headers;
using System.Text;
using Newtonsoft.Json;
using Polly;
using Polly.Retry;

namespace MonnetPaymentIntegration.Services
{
    public class MonnetTransactionService
    {
        private readonly HttpClient _httpClient;
        private readonly MonnetAuthService _authService;
        private readonly IConfiguration _configuration;
        private readonly ILogger<MonnetTransactionService> _logger;
        private readonly AsyncRetryPolicy _retryPolicy;

        public MonnetTransactionService(
            HttpClient httpClient,
            MonnetAuthService authService,
            IConfiguration configuration,
            ILogger<MonnetTransactionService> logger)
        {
            _httpClient = httpClient;
            _authService = authService;
            _configuration = configuration;
            _logger = logger;

            // Configurar política de reintento
            _retryPolicy = Policy
                .Handle<HttpRequestException>()
                .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
                .WaitAndRetryAsync(3, retryAttempt => 
                    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
                    onRetry: (response, delay, retryCount, context) => 
                    {
                        _logger.LogWarning($