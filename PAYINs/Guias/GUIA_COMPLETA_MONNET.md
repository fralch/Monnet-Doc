# 📚 Guía Completa de Monnet Payments Payin API

> **Audiencia:** Full Stack Web Developers
> **Versión de API:** v3
> **Fecha:** Febrero 2026
> **Idioma:** Español

---

## 📋 Tabla de Contenidos

1. [Introducción](#1-introducción)
2. [¿Qué es Monnet Payments?](#2-qué-es-monnet-payments)
3. [Características Principales](#3-características-principales)
4. [Requisitos Técnicos y Prerrequisitos](#4-requisitos-técnicos-y-prerrequisitos)
5. [Ambientes (CERT y PROD)](#5-ambientes-cert-y-prod)
6. [Credenciales y Autenticación](#6-credenciales-y-autenticación)
7. [Métodos de Pago Disponibles](#7-métodos-de-pago-disponibles)
8. [Flujo Principal — Crear una Transacción](#8-flujo-principal--crear-una-transacción)
9. [Consultar Estado de una Transacción](#9-consultar-estado-de-una-transacción)
10. [Notificaciones de Pago (Webhooks)](#10-notificaciones-de-pago-webhooks)
11. [Yape One Shot](#11-yape-one-shot)
12. [Yape On File (Suscripciones OCP)](#12-yape-on-file-suscripciones-ocp)
13. [Cuentas Virtuales (Virtual Accounts)](#13-cuentas-virtuales-virtual-accounts)
14. [Códigos de Estado](#14-códigos-de-estado)
15. [Códigos de Error](#15-códigos-de-error)
16. [Tipos de Documentos por País](#16-tipos-de-documentos-por-país)
17. [Browsers/Dispositivos Soportados](#17-browsersdispositivos-soportados)
18. [Pasos para Implementar en un Nuevo Proyecto Fintech](#18-pasos-para-implementar-en-un-nuevo-proyecto-fintech)
19. [Ejemplos de Código y Mejores Prácticas](#19-ejemplos-de-código-y-mejores-prácticas)
20. [Patrones de Integración](#20-patrones-de-integración)
21. [Manejo de Errores y Gestión de Estados](#21-manejo-de-errores-y-gestión-de-estados)
22. [Mejores Prácticas](#22-mejores-prácticas)
23. [Flujos de Pago Detallados](#23-flujos-de-pago-detallados)
24. [Estructura Recomendada del Proyecto](#24-estructura-recomendada-del-proyecto)
25. [Flujo de Desarrollo Recomendado](#25-flujo-de-desarrollo-recomendado)
26. [Monitoreo y Mantenimiento](#26-monitoreo-y-mantenimiento)

---

## 1. Introducción

Monnet Payments Payin API es una plataforma integral de procesamiento de pagos que permite a las empresas aceptar pagos en toda Latinoamérica a través de múltiples canales. Funciona como intermediario entre los comerciantes y diversos procesadores de pago, ofreciendo una API unificada para manejar diversos métodos de pago incluyendo tarjetas de crédito/débito, transferencias bancarias, pagos en efectivo, billeteras móviles y cuentas virtuales.

---

## 2. ¿Qué es Monnet Payments?

**Monnet Payments** es una empresa de intermediación de pagos y cobros online para **Latinoamérica**. Actúa como un gateway de pagos que conecta a tus usuarios con los procesadores bancarios y métodos de pago locales de cada país.

### Canales que ofrece:
| Canal | Descripción |
|---|---|
| 💳 **Tarjetas locales** | Crédito y débito (Visa, Mastercard, etc.) |
| 🏦 **Transferencia bancaria** | Online Banking, SPEI (México), CVU (Argentina), CCI (Perú) |
| 📱 **Pagos móviles** | Wallets, Yape, QR |
| 🏪 **Puntos de efectivo** | Farmacias, agentes, tiendas físicas |

### Países disponibles:
| País | Tarjeta Crédito | Tarjeta Débito | Transferencia | Efectivo |
|---|:---:|:---:|:---:|:---:|
| 🇦🇷 Argentina | ✅ | ✅ | ✅ | ✅ |
| 🇨🇱 Chile | ✅ | ✅ | ✅ | ✅ |
| 🇪🇨 Ecuador | ✅ | ✅ | ✅ | ✅ |
| 🇵🇪 Perú | ✅ | ✅ | ✅ | ✅ |
| 🇲🇽 México | ✅ | ✅ | ✅ | ✅ |
| 🇨🇴 Colombia | ✅ | ✅ | ✅ | ✅ |

---

## 3. Características Principales

### Métodos de Pago Soportados

| Método | Países | Descripción |
|--------|-----------|-------------|
| **TCTD** | Todo LATAM | Tarjetas de Crédito y Débito |
| **TC** | Todo LATAM | Tarjetas de Crédito |
| **TD** | Todo LATAM | Tarjetas de Débito |
| **Cash** | Perú, Ecuador, Argentina, Colombia, Guatemala | Pagos en efectivo en puntos físicos |
| **BankTransfer** | Perú, Ecuador, México, Chile, Argentina, Colombia, Guatemala, Brasil | Transferencias bancarias en línea |
| **BankTransfer_Businesses** | Perú | Transferencias bancarias empresariales |
| **Wallet** | Perú, Ecuador, Colombia, Guatemala, Argentina | Pagos mediante billeteras móviles |
| **QR** | Perú, Chile | Pagos QR (uso único) |
| **VA** | México, Argentina, Perú | Cuentas Virtuales |

### Soluciones Especiales
- **Yape One Shot**: Pagos de autorización única via Yape app
- **Yape on File (OCP)**: Pagos por suscripción con consentimiento almacenado
- **Cuentas Virtuales**: Cuentas bancarias específicas por país para seguimiento automatizado de depósitos

---

## 4. Requisitos Técnicos y Prerrequisitos

### URLs de Entorno
- **CERT (Test)**: `https://cert.payin.api.monnetpayments.com/`
- **PROD (Producción)**: `https://payin.api.monnetpayments.com/`

### Requisitos de Autenticación
- **API Key**: Identificador público del comerciante
- **Signature Key**: Clave secreta para firmas HMAC-SHA512
- **Merchant ID**: Identificador único asignado por Monnet

### Requisitos de Seguridad
- Todas las comunicaciones deben usar HTTPS
- Verificación de firmas usando HMAC-SHA512
- Validación de timestamp Unix (ventana de 2 minutos)
- Puntos finales públicos para webhooks

### Prerrequisitos Técnicos
- Servidor capaz de manejar solicitudes HTTPS POST
- Capacidad para generar hashes SHA-512
- Punto final webhook para notificaciones de pago
- Configuración CORS si es necesario para integración frontend
- Soporte para formato JSON request/response

---

## 5. Ambientes (CERT y PROD)

Monnet proporciona **dos ambientes** completamente separados:

### 🧪 CERT (Certificación / Sandbox)
Usado para **desarrollo y pruebas**. Ningún cobro real se efectúa.

| Recurso | URL |
|---|---|
| API Payin | `https://cert.monnetpayments.com/api-payin/v3/online-payments` |
| Status/Consultas | `https://cert.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations` |
| Back Office | `https://cert.payin.monnetpayments.com/` |
| Suscripciones (Yape) | `https://cert.subscriptions.payin.monnet.io/api/v1/subscription` |

### 🚀 PROD (Producción)
Usado una vez que Monnet aprueba tu certificación.

| Recurso | URL |
|---|---|
| API Payin | `https://payin.api.monnetpayments.com/api-payin/v3/online-payments` |
| Status/Consultas | `https://apiin.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations` |
| Back Office | `https://payin.monnetpayments.com/` |
| Suscripciones (Yape) | `https://subscriptions.payin.monnet.io/api/v1/subscription` |

> ⚠️ **Importante:** Todos los ejemplos en la documentación oficial están basados en CERT. Cuando hagas deploy a producción, Monnet realizará un proceso de **Certificación** donde un integrador probará tu integración con pagos reales.

---

## 6. Credenciales y Autenticación

### 6.1 Obtener credenciales

1. Contacta a tu representante de cuenta en Monnet Payments para que te creen un account en el Back Office CERT
2. Accede a: [https://cert.payin.monnetpayments.com/pages/auth/login](https://cert.payin.monnetpayments.com/pages/auth/login)
3. Ve a **Perfil → Admin → Merchant Data**
4. Obtén tus dos claves:

| Credencial | Descripción | Uso |
|---|---|---|
| `payinMerchantID` | Tu ID de comercio asignado por Monnet | Enviado en el body de cada request |
| `KeyMonnet` (Signature Key) | Clave secreta para generar el hash SHA512 | **NUNCA se envía** directamente; usada solo para generar el hash |

> 🔐 **Seguridad:** La `KeyMonnet` jamás debe salir de tu servidor backend. Es equivalente a un secret key.

También debes configurar en el Back Office la **URL de webhook/notificación** donde Monnet enviará las confirmaciones de pago.

### 6.2 Generación del Hash de Verificación (SHA512)

El campo `payinVerification` es obligatorio en cada transacción. Se calcula así:

```
payinVerification = SHA512( payinMerchantID + payinMerchantOperationNumber + payinAmount + payinCurrency + KeyMonnet )
```

#### Ejemplo en JavaScript/Node.js:
```javascript
const crypto = require('crypto');

function generatePayinVerification(merchantId, operationNumber, amount, currency, keyMonnet) {
  const rawString = `${merchantId}${operationNumber}${amount}${currency}${keyMonnet}`;
  return crypto.createHash('sha512').update(rawString.trim()).digest('hex');
}

// Uso:
const hash = generatePayinVerification('674', 'ORDER-12345', '150.00', 'PEN', 'tu-key-monnet-secreta');
console.log(hash); // string hexadecimal de 128 caracteres
```

#### Ejemplo en PHP:
```php
<?php
function generatePayinVerification($merchantId, $operationNumber, $amount, $currency, $keyMonnet) {
    $rawString = $merchantId . $operationNumber . $amount . $currency . $keyMonnet;
    return openssl_digest(trim($rawString), 'sha512');
}

// Uso:
$hash = generatePayinVerification('674', 'ORDER-12345', '150.00', 'PEN', 'tu-key-monnet-secreta');
```

#### Ejemplo en Python:
```python
import hashlib

def generate_payin_verification(merchant_id, operation_number, amount, currency, key_monnet):
    raw_string = f"{merchant_id}{operation_number}{amount}{currency}{key_monnet}".strip()
    return hashlib.sha512(raw_string.encode()).hexdigest()
```

### 6.3 Autenticación para Virtual Accounts (HMAC-SHA512)

Las Cuentas Virtuales usan un sistema de autenticación diferente basado en **HMAC-SHA512** con headers:

```
X-Api-Key: {tu-api-key}
X-Timestamp: {unix-timestamp-en-segundos}
X-Signature: HMAC_SHA512(secret-key, timestamp + api-key)
```

```javascript
const crypto = require('crypto');

function generateVASignature(secretKey, apiKey) {
  const timestamp = Math.floor(Date.now() / 1000); // Unix en segundos
  const message = `${timestamp}${apiKey}`;
  const signature = crypto.createHmac('sha512', secretKey).update(message).digest('hex');
  
  return { timestamp, signature };
}
```

---

## 7. Métodos de Pago Disponibles

El campo `payinMethod` define qué tipo de pago usará el cliente:

| Código | Descripción | Países |
|---|---|---|
| `TCTD` | Tarjeta de Crédito y Débito (ambas) | Chile, Argentina, México, Perú, Colombia, Ecuador |
| `TC` | Solo Tarjeta de Crédito | Chile, México, Perú, Argentina, Colombia, Ecuador |
| `TD` | Solo Tarjeta de Débito | Chile, México, Perú, Argentina, Colombia, Ecuador |
| `Cash` | Pago en Efectivo (farmacias, agentes) | Perú, Ecuador, Argentina, Colombia, Guatemala |
| `BankTransfer` | Transferencia Bancaria Online | Perú, Ecuador, México, Chile, Argentina, Colombia, Guatemala, Brasil |
| `BankTransfer_Businesses` | Transferencia Bancaria para Empresas | Solo Perú |
| `Wallet` | Billetera Digital | Perú, Ecuador, Colombia, Guatemala, Argentina |
| `QR` | Código QR (uso único — no reutilizable) | Perú, Chile |
| `VA` | Cuenta Virtual | México, Argentina, Perú |

---

## 8. Flujo Principal — Crear una Transacción

### 8.1 ¿Cómo funciona?

```
1. Usuario selecciona método de pago en tu sitio
2. Tu backend genera el hash SHA512 y hace POST a Monnet
3. Monnet valida y devuelve una URL de pago
4. Tu frontend redirige al usuario a esa URL
5. Usuario completa el pago en el gateway de Monnet/banco
6. Monnet notifica a tu webhook con el resultado
7. Tu sistema actualiza el estado del pedido
```

### 8.2 Endpoint

```
POST https://cert.monnetpayments.com/api-payin/v3/online-payments
Content-Type: application/json
```

### 8.3 Campos del Request (Perú - ejemplo completo)

> ⚠️ **IMPORTANTE:** Todos los campos deben enviarse aunque sean opcionales. Los campos `payinCustomerName`, `payinCustomerLastName`, `payinCustomerEmail` y `payinCustomerPhone` **NUNCA pueden ir vacíos**.

| Campo | Tipo | Req. | Descripción |
|---|---|:---:|---|
| `payinMerchantID` | String | ✅ | Tu ID de comercio en Monnet |
| `payinAmount` | Decimal | ✅ | Monto con 2 decimales (ej: `"150.00"`) |
| `payinCurrency` | String ISO-4217 | ✅ | Moneda: `PEN`, `CLP`, `MXN`, `ARS`, etc. |
| `payinMerchantOperationNumber` | String (max 50) | ✅ | Tu número de orden único |
| `payinMethod` | String | ✅ | Método de pago: `BankTransfer`, `TCTD`, etc. |
| `payinVerification` | String | ✅ | Hash SHA512 generado en tu backend |
| `payinTransactionOKURL` | String HTTPS | ✅ | URL de éxito (redirección del usuario) |
| `payinTransactionErrorURL` | String HTTPS | ✅ | URL de error (redirección del usuario) |
| `payinExpirationTime` | Integer | ✅ | Minutos para expirar (Online: 30, Efectivo: 120) |
| `payinLanguage` | String ISO-639-1 | ✅ | `ES`, `EN`, `PT` |
| `payinCustomerEmail` | String | ✅ | Email del cliente |
| `payinCustomerName` | String (max 30) | ✅ | Nombre del cliente |
| `payinCustomerLastName` | String (max 30) | ✅ | Apellido del cliente |
| `payinCustomerTypeDocument` | String | ✅ | Tipo de doc: `DNI`, `RUC`, `CE`, etc. |
| `payinCustomerDocument` | String | ✅ | Número de documento |
| `payinCustomerPhone` | String | ✅ | Teléfono (9 dígitos en Perú) |
| `payinCustomerAddress` | String | ✅ | Dirección del cliente |
| `payinCustomerCity` | String | ✅ | Ciudad |
| `payinCustomerRegion` | String | ✅ | Región/Estado (default: `"Lima"` en Perú) |
| `payinCustomerCountry` | String | ✅ | `"Peru"`, `"Chile"`, etc. |
| `payinCustomerZipCode` | String | ✅ | Código postal |
| `payinProductID` | String | ✅ | ID del producto (puede ser `"0"`) |
| `payinProductDescription` | String | ✅ | Descripción del producto |
| `payinProductAmount` | String | ✅ | Monto del producto |
| `payinProductSku` | String | ✅ | SKU del producto (puede ser `"0"`) |
| `payinProductQuantity` | String | ✅ | Cantidad (puede ser `"1"`) |
| `payinDateTime` | String | ✅ | Fecha de la transacción `YYYY-MM-DD` |
| `URLMonnet` | String | ✅ | URL del ambiente Monnet (CERT o PROD) |
| `typePost` | String | ✅ | Siempre: `"json"` |

### 8.4 Ejemplo completo del Request:

```json
{
  "payinMerchantID": "674",
  "payinAmount": "150.00",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "ORDER-20260220-001",
  "payinMethod": "BankTransfer",
  "payinVerification": "a3f8b2...hash_sha512...d9c1",
  "payinCustomerName": "Juan",
  "payinCustomerLastName": "Perez",
  "payinCustomerEmail": "juan.perez@email.com",
  "payinCustomerPhone": "912345678",
  "payinCustomerTypeDocument": "DNI",
  "payinCustomerDocument": "12345678",
  "payinRegularCustomer": "",
  "payinCustomerID": "user_internal_id_123",
  "payinDiscountCoupon": "",
  "payinLanguage": "ES",
  "payinExpirationTime": "30",
  "payinDateTime": "2026-02-20",
  "payinTransactionOKURL": "https://tuapp.com/pago/exito",
  "payinTransactionErrorURL": "https://tuapp.com/pago/error",
  "payinFilterBy": "",
  "payinCustomerAddress": "Av. Larco 1234",
  "payinCustomerCity": "Lima",
  "payinCustomerRegion": "Lima",
  "payinCustomerCountry": "Peru",
  "payinCustomerZipCode": "15036",
  "payinCustomerShippingName": "Juan Perez",
  "payinCustomerShippingPhone": "912345678",
  "payinCustomerShippingAddress": "Av. Larco 1234",
  "payinCustomerShippingCity": "Lima",
  "payinCustomerShippingRegion": "Lima",
  "payinCustomerShippingCountry": "Peru",
  "payinCustomerShippingZipCode": "15036",
  "payinProductID": "PROD-001",
  "payinProductDescription": "Recarga de saldo",
  "payinProductAmount": "150.00",
  "payinProductSku": "SKU-001",
  "payinProductQuantity": "1",
  "URLMonnet": "https://cert.monnetpayments.com/api-payin/v1/online-payments",
  "typePost": "json"
}
```

### 8.5 Response exitoso:

```json
{
  "url": "https://cert.monnetpayments.com/gateway/pay/xxx",
  "payinErrorCode": "0000",
  "payinErrorMessage": "Successfull process",
  "payinTrxOperation": "MONTRX207249992409275755"
}
```

> ⚠️ **Que el usuario sea redirigido a `payinTransactionOKURL` NO garantiza que el pago fue exitoso.** Siempre usa el sistema de notificaciones webhook para confirmar pagos.

---

## 9. Consultar Estado de una Transacción

Puedes consultar el estado de transacciones a través de un `POST`:

### Endpoint:

```
POST https://cert.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations
```

### Header requerido:

```
authorization: SHA256(MerchantID + KeyMonnet)
```

### Request — Por ID de operación:

```json
{
  "payinMerchantOperationNumber": "ORDER-20260220-001"
}
```

### Request — Por rango de fechas:

```json
{
  "payinStartDate": "2026-01-01",
  "payinEndDate": "2026-02-20"
}
```

> ⚠️ No puedes usar la fecha actual como `payinStartDate`; usa como mínimo el día anterior (`hoy - 1`).

### Response:

```json
{
  "payinMerchantID": "674",
  "operations": [
    {
      "payinStateID": "5",
      "payinState": "AUTORIZADO",
      "payinAmount": "150.00",
      "payinCurrency": "PEN",
      "payinMerchantOperationNumber": "ORDER-20260220-001",
      "payinID": "MONTRX207249992409275755"
    }
  ]
}
```

---

## 10. Notificaciones de Pago (Webhooks)

Cuando Monnet recibe confirmación del banco de que un pago fue procesado, hace un **HTTP POST a tu endpoint** con el resultado.

### Tu endpoint debe:
- Ser **HTTPS pública** (no localhost)
- Aceptar POST con `Content-Type: application/json`
- Responder con **HTTP 200** inmediatamente
- Verificar el hash para evitar notificaciones falsas

### Payload recibido:

```json
{
  "payinStateID": "5",
  "payinState": "Autorizado",
  "payinStatusErrorMessage": "",
  "payinStatusErrorCode": "00",
  "payinMerchantID": "674",
  "payinAmount": "150.00",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "ORDER-20260220-001",
  "payinMethod": "BankTransfer",
  "payinVerification": "hash_sha512_para_verificar",
  "additionalInformation": [],
  "errorDetails": null
}
```

### Verificación del webhook (Node.js):

```javascript
const crypto = require('crypto');

app.post('/monnet/webhook', (req, res) => {
  const {
    payinMerchantID,
    payinMerchantOperationNumber,
    payinAmount,
    payinCurrency,
    payinVerification,
    payinStateID
  } = req.body;

  // 1. Verificar el hash para autenticar que viene de Monnet
  const expectedHash = crypto
    .createHash('sha512')
    .update(`${payinMerchantID}${payinMerchantOperationNumber}${payinAmount}${payinCurrency}${process.env.KEY_MONNET}`)
    .digest('hex');

  if (expectedHash !== payinVerification) {
    console.warn('⚠️ Webhook con firma inválida — posible fraude');
    return res.status(200).send('OK'); // Siempre responde 200 a Monnet
  }

  // 2. Procesar el pago según el estado
  if (payinStateID === '5') {
    // Pago AUTORIZADO — acredita saldo al usuario
    updateUserBalance(payinMerchantOperationNumber, payinAmount);
  } else if (payinStateID === '6') {
    // Pago DENEGADO
    markOrderAsFailed(payinMerchantOperationNumber, req.body.errorDetails);
  }

  // 3. SIEMPRE responder HTTP 200
  res.status(200).send('OK');
});
```

---

## 11. Yape One Shot

### ¿Qué es?

**Yape One Shot** es un método de pago para Perú que permite a los usuarios pagar **una sola transacción** a través de la app Yape mediante una **notificación push** o **deeplink**. No almacena consentimiento para cobros futuros.

### Casos de uso:
- Pago puntual sin suscripción
- Compatible con flujos móviles (deeplink a app Yape) y web (instrucciones en pantalla)

### Flujo:

```
1. Usuario selecciona "Yape" como método de pago en tu web/app
2. Tu backend crea una transacción con payinMethod: "Wallet" (o el código específico)
3. Monnet crea el pago y genera un deeplink si es mobile
4. Usuario abre su app Yape y aprueba o rechaza el pago
5. Monnet notifica el resultado a tu webhook
```

### Dispositivos soportados:
- **MOBILE:** Redirige via deeplink a la app Yape
- **WEB:** Muestra instrucciones en pantalla (el usuario ingresa su teléfono en la app)

---

## 12. Yape On File (Suscripciones OCP)

### ¿Qué es?

**Yape On File** (también llamado **One Click Payment — OCP**) permite crear suscripciones donde el usuario da su consentimiento **una sola vez** y el merchant puede cobrarle automáticamente en el futuro sin que el usuario tenga que volver a autorizar cada pago.

### Tipos de suscripción:
| Tipo | Descripción |
|---|---|
| `ON_DEMAND` | El merchant inicia el cobro cuando lo necesita |
| `RECURRENT` | Los cobros se procesan automáticamente en intervalos definidos (ej: mensual) |

### Procesadores disponibles:
| Procesador | Código | Descripción |
|---|---|---|
| Yape OCP | `Yape_on_file` | Primera wallet integrada. Permite pagos on-demand con un solo consentimiento |

### Endpoint — Crear Suscripción:

```
POST https://cert.subscriptions.payin.monnet.io/api/v1/subscription
```

### Header de autorización:

```
Authorization: Bearer SHA512(merchantId + type + customerId + processorCode + keyPayin)
```

### Request (ON_DEMAND):

```json
{
  "merchantId": 674,
  "subscriptionDetails": {
    "type": "ON_DEMAND",
    "device": "MOBILE",
    "customerId": "992212092",
    "processorCode": "Yape_on_file"
  },
  "metadata": [
    {
      "key": "MerchantReference",
      "value": "USER-ID-123"
    }
  ]
}
```

### Response exitoso (device MOBILE):

```json
{
  "subscriptionId": 12345,
  "status": "PENDING",
  "deepLink": "https://yape.com.pe/app/checkout/ocp/subscription?id=xxx"
}
```

> El merchant debe redirigir al usuario al `deepLink` para que apruebe la suscripción en la app Yape.

### Request (RECURRENT — cada 3 meses, 150 soles):

```json
{
  "merchantId": 674,
  "subscriptionDetails": {
    "type": "RECURRENT",
    "device": "WEB",
    "customerId": "992212092",
    "processorCode": "Yape_on_file",
    "periodicity": "3",
    "amount": 150.00
  }
}
```

### Cancelar Suscripción:

```
DELETE https://cert.subscriptions.payin.monnet.io/api/v1/subscription/{subscriptionId}
```

---

## 13. Cuentas Virtuales (Virtual Accounts)

### ¿Qué es?

Una **Cuenta Virtual** es un número de cuenta bancaria único asignado a un usuario específico de tu plataforma. Funciona como una cuenta real para recibir transferencias, pero su propósito es **identificar automáticamente** de quién viene cada depósito.

### Flujo:

```
1. Usuario solicita su cuenta virtual en tu app
2. Tu backend crea la VA via Monnet API
3. Monnet devuelve el número de cuenta único
4. Muestras el número al usuario para que lo use al hacer transferencias
5. Cuando alguien deposita, Monnet detecta el depósito
6. Monnet envía un webhook a tu endpoint con los detalles del depósito
7. Tu sistema acredita el saldo al usuario
```

### Países y tipos de cuenta:
| País | Tipo de Cuenta | Documento Requerido |
|---|---|---|
| 🇦🇷 Argentina | `CVU` | `CUIT` |
| 🇲🇽 México | `CLABE` | `RFC` |
| 🇵🇪 Perú | `CCI` | `DNI` / `RUC` |

### Endpoint:

```
POST {{base_url}}/merchant-payin-accounts/v1/accounts
```

### Headers requeridos:
| Header | Descripción |
|---|---|
| `X-Api-Key` | Tu API Key pública de Monnet |
| `X-Timestamp` | Unix timestamp en segundos |
| `X-Signature` | `HMAC_SHA512(secret-key, timestamp + api-key)` |
| `X-Account-deposit-mode` | `OWNER` (solo el dueño deposita) o `ANY` (cualquiera puede depositar) |

### Request body:

```json
{
  "owner": {
    "referenceId": "USER-INTERNO-123",
    "type": "PERSON",
    "document": {
      "type": "DNI",
      "number": "12345678"
    },
    "firstName": "Juan",
    "lastName": "Perez",
    "email": "juan.perez@email.com",
    "phone": {
      "countryCode": "51",
      "number": "912345678"
    }
  },
  "account": {
    "category": "VIRTUAL",
    "type": "CCI",
    "country": "PER",
    "currency": "PEN",
    "name": "juan.perez"
  },
  "metadata": {
    "segmento": "VIP",
    "plan": "premium"
  }
}
```

### Response exitoso:

```json
{
  "id": "acc_8af98b8c8a4",
  "traceId": "93963JM-38A1A",
  "timestamp": "2026-02-20T15:14:22Z",
  "status": "ACTIVE",
  "owner": { "..." : "..." },
  "account": {
    "category": "VIRTUAL",
    "type": "CCI",
    "number": "00219100123456789012",
    "name": "juan.perez",
    "country": "PER",
    "currency": "PEN"
  }
}
```

### Webhook de depósito recibido:

```json
{
  "version": "1.0",
  "operationId": "7019283",
  "status": {
    "code": "5",
    "description": "Aprobado"
  },
  "merchantId": "674",
  "amount": "200.00",
  "currency": "PEN",
  "payinMethod": "VA",
  "depositDetails": {
    "account": {
      "id": "acc_8af98b8c8a4",
      "number": "00219100123456789012"
    },
    "owner": {
      "fullName": "Juan Perez",
      "documentType": "DNI",
      "documentNumber": "12345678",
      "referenceId": "USER-INTERNO-123"
    },
    "depositor": {
      "fullName": "Maria Garcia",
      "documentType": "DNI",
      "documentNumber": "87654321"
    }
  },
  "receivedAt": "2026-02-20T14:22:05Z"
}
```

---

## 14. Códigos de Estado

Las transacciones pasan por los siguientes estados:

| `payinStateID` | `payinState` | Descripción |
|:---:|---|---|
| `1` | Creado | El pago fue creado (link generado) |
| `2` | Pendiente de pago | El pago está pendiente de confirmación |
| `3` | Expirado | El link de pago expiró |
| `5` | **Autorizado** ✅ | El pago fue completado exitosamente |
| `6` | **Denegado** ❌ | El pago fue rechazado |
| `9` | Liquidado | El pago fue liquidado/settled |
| `10` | Reembolsado | Reembolso realizado (trans. no completadas) |
| `11` | Devuelto | Reembolso realizado (trans. liquidadas) |

---

## 15. Códigos de Error

### Errores de Creación de Transacción:
| Código | Descripción |
|---|---|
| `0000` | ✅ Proceso exitoso |
| `0001` | `payinMerchantID` vacío o inválido |
| `0002` | `payinAmount` vacío o inválido |
| `0003` | `payinCurrency` vacío o inválido |
| `0005` | `payinVerification` inválido (hash incorrecto) |
| `0010` | Error de verificación (hash no coincide) |
| `0015` | Formato de `payinAmount` inválido |
| `0017` | Moneda no válida para este merchant |
| `0025` | Tipo de documento del cliente inválido |
| `0026` | Número de documento del cliente inválido |
| `0099` | Error interno de Payin |

### Errores de Confirmación de Pago (banco):
| Código | Descripción |
|---|---|
| `9012` | Transacción inválida |
| `9013` | Monto inválido |
| `9051` | Fondos insuficientes |
| `9054` | Tarjeta expirada |
| `9055` | PIN incorrecto |
| `9057` | Transacción no permitida para el tarjetahabiente |
| `9017` | El cliente canceló la operación |
| `9097` | Timeout de la operación |

---

## 16. Tipos de Documentos por País

| País | Código | Descripción | Longitud |
|---|---|---|---|
| 🇦🇷 Argentina | `DNI` | Documento de Identidad | 8 dígitos |
| 🇦🇷 Argentina | `CUIT` | ID Tributario | 11 dígitos |
| 🇨🇱 Chile | `RUT` | Rol Único Tributario | 7-9 dígitos |
| 🇵🇪 Perú | `DNI` | Documento Nacional de Identidad | 8 dígitos |
| 🇵🇪 Perú | `CE` | Carné de Extranjería | 8-12 alphanum |
| 🇵🇪 Perú | `RUC` | Registro Único de Contribuyentes | 9-10 dígitos |
| 🇪🇨 Ecuador | `CI` | Cédula de Identidad | 10 dígitos |
| 🇪🇨 Ecuador | `RUC` | Registro Único de Contribuyentes | 13 dígitos |
| 🇲🇽 México | `CURP` | Clave Única de Registro de Población | 13-18 dígitos |
| 🇲🇽 México | `RFC` | Registro Federal de Contribuyentes | 10-13 dígitos |
| 🇨🇴 Colombia | `CC` | Cédula de Ciudadanía | 6-10 dígitos |
| 🇨🇴 Colombia | `NIT` | Número de Identificación Tributaria | 9 dígitos |
| 🇧🇷 Brasil | `CPF` | Cadastro de Pessoas Físicas | 11 dígitos |
| 🇧🇷 Brasil | `CNPJ` | Cadastro Nacional da Pessoa Jurídica | 14 dígitos |

---

## 17. Browsers/Dispositivos Soportados

Para el voucher/gateway de Monnet, los navegadores mínimos requeridos son:

| Navegador | Versión Mínima |
|---|---|
| Safari | 17 o superior |
| Google Chrome | 130 o superior |
| Microsoft Edge | 130 o superior |
| Mozilla Firefox | 133 o superior |

---

## 18. Pasos para Implementar en un Nuevo Proyecto Fintech

### PASO 1: Contactar a Monnet y Obtener Acceso Sandbox

1. Contacta a tu representante de Monnet Payments para solicitar acceso al ambiente CERT
2. Recibirás acceso al Back Office CERT: [https://cert.payin.monnetpayments.com/](https://cert.payin.monnetpayments.com/)
3. Guarda tus credenciales:
   - `MONNET_MERCHANT_ID`
   - `MONNET_KEY` (KeyMonnet / Signature Key)

### PASO 2: Configurar el Back Office CERT

1. Ir a **Admin → Merchant Data**
2. Configurar la **URL de notificaciones** (tu webhook de producción/staging):
   ```
   https://tuapp.com/api/webhooks/monnet
   ```
3. Para Virtual Accounts: solicitar al equipo técnico de Monnet las credenciales adicionales (`X-Api-Key`, `secret-key`)

### PASO 3: Estructura del Proyecto Recomendada

Para un proyecto **Node.js/Express + Next.js** (o similar):

```
tu-proyecto-fintech/
├── backend/
│   ├── services/
│   │   └── monnet.service.js     # Lógica del SDK de Monnet
│   ├── controllers/
│   │   └── payment.controller.js # Endpoints de tu API
│   ├── routes/
│   │   └── payment.routes.js
│   └── config/
│       └── monnet.config.js      # Variables de entorno
├── frontend/
│   └── pages/
│       ├── checkout.js           # Formulario de pago
│       ├── pago/
│       │   ├── exito.js          # Página de éxito
│       │   └── error.js          # Página de error
└── .env
```

### PASO 4: Variables de Entorno

```env
# .env (NUNCA commitear con datos reales)
MONNET_MERCHANT_ID=674
MONNET_KEY=tu_key_monnet_secreta
MONNET_ENV=cert  # "cert" o "prod"

# URLs por ambiente
MONNET_API_CERT=https://cert.monnetpayments.com/api-payin/v3/online-payments
MONNET_API_PROD=https://payin.api.monnetpayments.com/api-payin/v3/online-payments

# Para Virtual Accounts
MONNET_VA_API_KEY=tu_api_key_va
MONNET_VA_SECRET=tu_secret_va

# Tu dominio
APP_URL=https://tuapp.com
```

### PASO 5: Implementar el Servicio de Monnet (Backend)

```javascript
// services/monnet.service.js
const crypto = require('crypto');
const axios = require('axios');

class MonnetService {
  constructor() {
    this.merchantId = process.env.MONNET_MERCHANT_ID;
    this.keyMonnet = process.env.MONNET_KEY;
    this.baseUrl = process.env.MONNET_ENV === 'prod'
      ? process.env.MONNET_API_PROD
      : process.env.MONNET_API_CERT;
  }

  /**
   * Genera el hash SHA512 para verificación
   */
  generateVerification(operationNumber, amount, currency) {
    const raw = `${this.merchantId}${operationNumber}${amount}${currency}${this.keyMonnet}`;
    return crypto.createHash('sha512').update(raw.trim()).digest('hex');
  }

  /**
   * Verifica el hash recibido en el webhook
   */
  verifyWebhookSignature(data) {
    const { payinMerchantID, payinMerchantOperationNumber, payinAmount, payinCurrency, payinVerification } = data;
    const expected = crypto
      .createHash('sha512')
      .update(`${payinMerchantID}${payinMerchantOperationNumber}${payinAmount}${payinCurrency}${this.keyMonnet}`)
      .digest('hex');
    return expected === payinVerification;
  }

  /**
   * Crea una transacción de pago
   */
  async createTransaction({ 
    orderNumber, amount, currency, method, 
    customer, product, language = 'ES', expirationTime = 30 
  }) {
    const verification = this.generateVerification(orderNumber, amount, currency);
    
    const payload = {
      payinMerchantID: this.merchantId,
      payinAmount: amount,
      payinCurrency: currency,
      payinMerchantOperationNumber: orderNumber,
      payinMethod: method,
      payinVerification: verification,
      payinCustomerName: customer.firstName,
      payinCustomerLastName: customer.lastName,
      payinCustomerEmail: customer.email,
      payinCustomerPhone: customer.phone,
      payinCustomerTypeDocument: customer.documentType,
      payinCustomerDocument: customer.documentNumber,
      payinRegularCustomer: '',
      payinCustomerID: customer.id || '',
      payinDiscountCoupon: '',
      payinLanguage: language,
      payinExpirationTime: String(expirationTime),
      payinDateTime: new Date().toISOString().split('T')[0],
      payinTransactionOKURL: `${process.env.APP_URL}/pago/exito?order=${orderNumber}`,
      payinTransactionErrorURL: `${process.env.APP_URL}/pago/error?order=${orderNumber}`,
      payinFilterBy: '',
      payinCustomerAddress: customer.address || 'N/A',
      payinCustomerCity: customer.city || 'Lima',
      payinCustomerRegion: customer.region || 'Lima',
      payinCustomerCountry: customer.country || 'Peru',
      payinCustomerZipCode: customer.zipCode || '15000',
      payinCustomerShippingName: `${customer.firstName} ${customer.lastName}`,
      payinCustomerShippingPhone: customer.phone,
      payinCustomerShippingAddress: customer.address || 'N/A',
      payinCustomerShippingCity: customer.city || 'Lima',
      payinCustomerShippingRegion: customer.region || 'Lima',
      payinCustomerShippingCountry: customer.country || 'Peru',
      payinCustomerShippingZipCode: customer.zipCode || '15000',
      payinProductID: product.id || '0',
      payinProductDescription: product.description || 'N/A',
      payinProductAmount: product.amount || amount,
      payinProductSku: product.sku || '0',
      payinProductQuantity: String(product.quantity || 1),
      URLMonnet: this.baseUrl,
      typePost: 'json'
    };

    const response = await axios.post(this.baseUrl, payload, {
      headers: { 'Content-Type': 'application/json' }
    });

    return response.data;
  }
}

module.exports = new MonnetService();
```

### PASO 6: Implementar los Endpoints (Controller)

```javascript
// controllers/payment.controller.js
const monnetService = require('../services/monnet.service');
const Order = require('../models/Order'); // Tu modelo de BD

// Iniciar pago
exports.createPayment = async (req, res) => {
  try {
    const { amount, currency, method, productId } = req.body;
    const user = req.user; // Auth middleware

    // 1. Generar número de orden único
    const orderNumber = `ORD-${Date.now()}-${user.id}`;

    // 2. Guardar orden en BD con estado PENDING
    const order = await Order.create({
      number: orderNumber,
      userId: user.id,
      amount,
      currency,
      status: 'PENDING',
      paymentMethod: method
    });

    // 3. Llamar a Monnet
    const result = await monnetService.createTransaction({
      orderNumber,
      amount: parseFloat(amount).toFixed(2),
      currency,
      method,
      customer: {
        firstName: user.firstName,
        lastName: user.lastName,
        email: user.email,
        phone: user.phone,
        documentType: user.documentType,
        documentNumber: user.documentNumber,
        id: String(user.id),
        country: user.country
      },
      product: {
        id: productId,
        description: 'Recarga de saldo',
        amount: parseFloat(amount).toFixed(2)
      }
    });

    // 4. Retornar la URL de pago al frontend
    if (result.payinErrorCode === '0000') {
      await Order.update({ monnetTrxId: result.payinTrxOperation }, { where: { number: orderNumber } });
      return res.json({ success: true, redirectUrl: result.url });
    } else {
      return res.status(400).json({ success: false, error: result.payinErrorMessage });
    }

  } catch (error) {
    console.error('Error creando pago:', error);
    res.status(500).json({ success: false, error: 'Error interno del servidor' });
  }
};

// Webhook de notificación de Monnet
exports.handleWebhook = async (req, res) => {
  // Siempre responder 200 primero (Monnet reintenta si no recibe 200)
  res.status(200).send('OK');

  try {
    const data = req.body;

    // 1. Verificar firma SHA512
    if (!monnetService.verifyWebhookSignature(data)) {
      console.warn('⚠️ Webhook con firma inválida:', data.payinMerchantOperationNumber);
      return;
    }

    const { payinMerchantOperationNumber, payinStateID, payinAmount, errorDetails } = data;

    // 2. Buscar la orden en BD
    const order = await Order.findOne({ where: { number: payinMerchantOperationNumber } });
    if (!order) return;

    // 3. Actualizar según estado
    if (payinStateID === '5') {
      // Pago AUTORIZADO
      await order.update({ status: 'COMPLETED' });
      await creditUserBalance(order.userId, payinAmount); // Tu lógica de negocio
      
    } else if (payinStateID === '6') {
      // Pago DENEGADO
      await order.update({ 
        status: 'FAILED', 
        errorCode: errorDetails?.codeErrorTrx,
        errorMessage: errorDetails?.messageErrorTrx
      });
    }

  } catch (error) {
    console.error('Error procesando webhook:', error);
  }
};
```

### PASO 7: Configurar Rutas

```javascript
// routes/payment.routes.js
const express = require('express');
const router = express.Router();
const { createPayment, handleWebhook } = require('../controllers/payment.controller');
const authMiddleware = require('../middleware/auth');

// Ruta para iniciar pago (requiere auth)
router.post('/create', authMiddleware, createPayment);

// Webhook de Monnet (SIN auth, pero verificamos el hash internamente)
router.post('/webhook/monnet', handleWebhook);

module.exports = router;
```

### PASO 8: Páginas del Frontend

```jsx
// frontend: Checkout component (React)
import { useState } from 'react';
import axios from 'axios';

export default function Checkout() {
  const [loading, setLoading] = useState(false);
  const [method, setMethod] = useState('BankTransfer');

  const handlePay = async (amount) => {
    setLoading(true);
    try {
      const { data } = await axios.post('/api/payments/create', {
        amount: '150.00',
        currency: 'PEN',
        method,
        productId: 'RECARGA-SALDO'
      });

      if (data.success) {
        // Redirigir al gateway de Monnet
        window.location.href = data.redirectUrl;
      }
    } catch (err) {
      alert('Error al procesar el pago');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <h2>Selecciona método de pago</h2>
      <select value={method} onChange={e => setMethod(e.target.value)}>
        <option value="BankTransfer">Banca Online</option>
        <option value="TCTD">Tarjeta Crédito/Débito</option>
        <option value="Cash">Efectivo</option>
        <option value="Wallet">Yape</option>
      </select>
      <button onClick={() => handlePay('150.00')} disabled={loading}>
        {loading ? 'Procesando...' : 'Pagar S/ 150.00'}
      </button>
    </div>
  );
}
```

### PASO 9: Pruebas en Sandbox

Áreas que debes probar antes de ir a producción:

- [ ] ✅ **Creación de transacción** — verificar que recibes la URL correctamente
- [ ] ✅ **Flujo de checkout** — redireccionamiento al gateway de Monnet
- [ ] ✅ **Webhook exitoso** — simular un pago con `payinStateID: "5"`
- [ ] ✅ **Webhook denegado** — simular un pago fallido con `payinStateID: "6"`
- [ ] ✅ **Verificación de hash** — asegurar que rechazas webhooks con hash inválido
- [ ] ✅ **Notificación duplicada** — idempotencia (procesar misma orden 2 veces no debe duplicar saldo)
- [ ] ✅ **Expiración** — transacción con tiempo corto de expiración
- [ ] ✅ **Errores de validación** — enviar campos inválidos y manejar errores `0001–0099`

### PASO 10: Checklist de Seguridad

- [ ] 🔐 `KeyMonnet` guardada en variable de entorno, nunca hardcodeada ni en frontend
- [ ] 🔐 Verificar el hash SHA512 en TODOS los webhooks recibidos
- [ ] 🔐 Usar HTTPS en todas las URLs (webhook, OK, Error)
- [ ] 🔐 Implementar idempotencia en el webhook (verificar si ya procesaste esa orden)
- [ ] 🔐 Responder siempre HTTP 200 al webhook (incluso si hay error interno, procesar asíncronamente)
- [ ] 🔐 Rate limiting en tu endpoint de creación de transacciones
- [ ] 🔐 Para VA: `X-Api-Key` y `secret-key` también en variables de entorno
- [ ] 🔐 Validar que el `payinAmount` del webhook coincida con el de tu BD antes de acreditar

### PASO 11: Ir a Producción

1. **Contactar a Monnet** para programar la **Certificación**
2. Un integrador de Monnet probará tu integración con pagos reales en estas áreas:
   - Checkout completo
   - Flujo de pago
   - Creación de transacción
   - Recepción de notificaciones
3. Una vez aprobado, Monnet generará nuevas credenciales para **PROD**
4. Cambiar tus variables de entorno a las de PROD:
   ```env
   MONNET_KEY=nueva_key_real_produccion
   MONNET_MERCHANT_ID=id_real_produccion
   MONNET_ENV=prod
   ```
5. Cambiar la URL del webhook en el Back Office PROD a tu URL real

---

## 19. Ejemplos de Código y Mejores Prácticas

### 19.1 Ejemplo de Creación de Transacción (Node.js)

```javascript
const crypto = require('crypto');
const axios = require('axios');

async function createTransaction(transactionData) {
  const timestamp = Math.floor(Date.now() / 1000);
  const message = `${timestamp}${config.apiKey}${transactionData.payinMerchantOperationNumber}`;
  const signature = crypto.createHmac('sha512', config.signatureKey).update(message).digest('hex');

  try {
    const response = await axios.post(
      `${config.baseUrl}api-payin/v3/online-payments`,
      transactionData,
      {
        headers: {
          'Content-Type': 'application/json',
          'X-Api-Key': config.apiKey,
          'X-Timestamp': timestamp,
          'X-Signature': signature
        }
      }
    );
    return response.data;
  } catch (error) {
    console.error('Transaction creation failed:', error.response?.data);
    throw error;
  }
}
```

### 19.2 Verificación de Webhook

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(requestBody, headers) {
  const { 'x-signature': signature, 'x-timestamp': timestamp, 'x-api-key': apiKey } = headers;
  
  // Verificar timestamp está dentro de 2 minutos
  const currentTime = Math.floor(Date.now() / 1000);
  if (Math.abs(currentTime - parseInt(timestamp)) > 120) {
    return false;
  }

  // Verificar API key
  if (apiKey !== config.apiKey) {
    return false;
  }

  // Verificar firma
  const message = `${timestamp}${apiKey}${requestBody}`;
  const expectedSignature = crypto.createHmac('sha512', config.signatureKey).update(message).digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}
```

### 19.3 Creación de Cuenta Virtual

```javascript
async function createVirtualAccount(accountData) {
  const timestamp = Math.floor(Date.now() / 1000);
  const message = `${timestamp}${config.apiKey}${accountData.owner.referenceId}`;
  const signature = crypto.createHmac('sha512', config.signatureKey).update(message).digest('hex');

  try {
    const response = await axios.post(
      `${config.baseUrl}merchant-payin-accounts/v1/accounts`,
      accountData,
      {
        headers: {
          'Content-Type': 'application/json',
          'X-Api-Key': config.apiKey,
          'X-Timestamp': timestamp,
          'X-Signature': signature,
          'X-Account-deposit-mode': 'OWNER' // o 'ANY'
        }
      }
    );
    return response.data;
  } catch (error) {
    console.error('Virtual account creation failed:', error.response?.data);
    throw error;
  }
}
```

### 19.4 Creación de Suscripción Yape

```javascript
async function createYapeSubscription(subscriptionData) {
  const timestamp = Math.floor(Date.now() / 1000);
  const message = `${timestamp}${config.apiKey}${subscriptionData.customerId}`;
  const signature = crypto.createHmac('sha512', config.signatureKey).update(message).digest('hex');

  try {
    const response = await axios.post(
      'https://cert.subscriptions.payin.monnet.io/api/v1/subscription',
      subscriptionData,
      {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${signature}`
        }
      }
    );
    return response.data;
  } catch (error) {
    console.error('Subscription creation failed:', error.response?.data);
    throw error;
  }
}
```

### 19.5 Verificación de Estado de Pago

```javascript
async function getPaymentStatus(merchantOperationNumber) {
  const timestamp = Math.floor(Date.now() / 1000);
  const message = `${timestamp}${config.apiKey}${merchantOperationNumber}`;
  const signature = crypto.createHmac('sha512', config.signatureKey).update(message).digest('hex');

  try {
    const response = await axios.post(
      `${config.baseUrl}merchant/{MID}/operations`,
      {
        payinMerchantOperationNumber: merchantOperationNumber
      },
      {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${signature}`
        }
      }
    );
    return response.data;
  } catch (error) {
    console.error('Status check failed:', error.response?.data);
    throw error;
  }
}
```

---

## 20. Patrones de Integración

### 20.1 Flujo de Pago Síncrono

```javascript
// Frontend inicia pago
async function initiatePayment(order) {
  try {
    const transaction = await createTransaction({
      payinAmount: order.total,
      payinCurrency: 'USD',
      payinMerchantOperationNumber: order.id,
      payinTransactionOKURL: `${config.webhookUrl}/success`,
      payinTransactionErrorURL: `${config.webhookUrl}/failure`
    });
    
    // Redirigir a pasarela de pago
    window.location.href = transaction.paymentUrl;
  } catch (error) {
    // Manejar error
  }
}
```

### 20.2 Flujo Asíncrono con Webhooks

```javascript
// Handler de webhook backend
app.post('/webhook', async (req, res) => {
  try {
    // Verificar firma
    if (!verifyWebhookSignature(req.body, req.headers)) {
      return res.status(401).send('Invalid signature');
    }

    const { payinStateID, payinMerchantOperationNumber, payinAmount } = req.body;

    // Procesar estado del pago
    switch (payinStateID) {
      case '5': // Autorizado
        await processPaymentSuccess(payinMerchantOperationNumber, payinAmount);
        break;
      case '6': // Denegado
        await processPaymentFailure(payinMerchantOperationNumber);
        break;
      default:
        console.log('Unknown payment status:', payinStateID);
    }

    res.status(200).send('OK');
  } catch (error) {
    console.error('Webhook processing error:', error);
    res.status(500).send('Error processing webhook');
  }
});
```

### 20.3 Patrón de Integración de Cuentas Virtuales

```javascript
// Registro de usuario con cuenta virtual
async function registerUserWithVirtualAccount(userData) {
  try {
    // Crear cuenta virtual
    const virtualAccount = await createVirtualAccount({
      owner: {
        referenceId: userData.id,
        type: 'PERSON',
        document: {
          type: 'DNI',
          number: userData.documentNumber
        },
        firstName: userData.firstName,
        lastName: userData.lastName,
        email: userData.email,
        phone: {
          countryCode: userData.countryCode,
          number: userData.phoneNumber
        }
      },
      account: {
        category: 'VIRTUAL',
        type: 'CCI', // o CVU, CLABE según país
        country: userData.country,
        currency: userData.currency,
        name: `${userData.firstName}.${userData.lastName}`
      }
    });

    // Almacenar detalles de cuenta virtual
    await saveUserVirtualAccount(userData.id, virtualAccount);

    return virtualAccount;
  } catch (error) {
    // Manejar error de creación de cuenta
    throw error;
  }
}
```

### 20.4 Patrón de Suscripción Yape

```javascript
// Pagos basados en suscripción
async function processSubscriptionPayment(subscriptionId, amount) {
  try {
    // Verificar suscripción está activa
    const subscription = await getSubscriptionDetails(subscriptionId);
    
    if (subscription.status !== 'ACTIVE') {
      throw new Error('Subscription not active');
    }

    // Crear pago usando suscripción
    const payment = await createPaymentUsingSubscription({
      subscriptionId: subscriptionId,
      amount: amount,
      currency: subscription.currency
    });

    return payment;
  } catch (error) {
    // Manejar error de pago
    throw error;
  }
}
```

---

## 21. Manejo de Errores y Gestión de Estados

### Códigos de Error Comunes
- **0000**: Éxito
- **0001-0007**: Campos requeridos faltantes
- **0009**: ID de comerciante inválido
- **0010**: Firma de verificación inválida
- **0011**: Comerciante no habilitado
- **9001-9099**: Errores de procesamiento de pago
- **B400-B500**: Errores de validación de negocio

### Códigos de Estado
- **1**: Creado
- **2**: Pendiente de pago
- **3**: Expirado
- **5**: Autorizado/Completado
- **6**: Denegado
- **9**: Liquidado
- **10**: Reembolsado
- **11**: Devuelto

---

## 22. Mejores Prácticas

### Mejores Prácticas de Seguridad
1. Siempre usar HTTPS para todas las comunicaciones
2. Implementar verificación de firma adecuada para webhooks
3. Usar credenciales específicas por entorno (CERT vs PROD)
4. Implementar rate limiting (máximo 10 transacciones por segundo)
5. Almacenar datos sensibles de forma segura y nunca registrarlos

### Mejores Prácticas de Rendimiento
1. Procesar notificaciones webhook de forma asíncrona
2. Implementar caché adecuada para datos frecuentemente accedidos
3. Usar connection pooling para solicitudes API
4. Implementar lógica de reintento con backoff exponencial
5. Monitorear tiempos de respuesta de API y tasas de error

### Mejores Prácticas de Desarrollo
1. Probar exhaustivamente en entorno CERT antes de producción
2. Implementar logging comprehensivo para debugging
3. Usar manejo de errores adecuado y mensajes de error amigables
4. Seguir requisitos específicos por país para formatos de documentos
5. Validar todos los datos de entrada antes de enviar a API de Monnet

### Mejores Prácticas de Despliegue en Producción
1. Realizar testing de certificación antes de ir a vivo
2. Monitorear entrega de webhooks y tiempos de respuesta
3. Implementar mecanismos de fallback para procesamiento de pagos
4. Revisar y actualizar integraciones API regularmente
5. Mantener cumplimiento con regulaciones locales

---

## 23. Flujos de Pago Detallados

### Flujo Completo de Transacción
1. **Inicialización**: Cliente selecciona método de pago en frontend
2. **Creación**: Backend crea transacción con Monnet API
3. **Redirección**: Cliente redirigido a pasarela de pago
4. **Pago**: Cliente completa pago en pasarela
5. **Notificación**: Monnet envía webhook con estado
6. **Verificación**: Backend verifica y procesa estado
7. **Confirmación**: Cliente recibe confirmación

### Flujo de Cuenta Virtual
1. **Registro**: Usuario crea cuenta con información personal
2. **Creación**: Sistema crea cuenta virtual con Monnet
3. **Distribución**: Usuario recibe detalles de cuenta
4. **Depósito**: Usuario realiza transferencia bancaria
5. **Notificación**: Webhook notifica sobre depósito
6. **Procesamiento**: Sistema procesa fondos y actualiza saldo

### Flujo de Suscripción Yape
1. **Suscripción**: Usuario autoriza suscripción via Yape
2. **Creación**: Sistema crea suscripción con Monnet
3. **Pago**: Sistema inicia pagos recurrentes
4. **Notificación**: Webhook actualiza estado de suscripción
5. **Manejo**: Sistema procesa pagos y actualiza facturación

---

## 24. Estructura Recomendada del Proyecto

```
monnet-payments/
├── src/
│   ├── services/
│   │   ├── monnet-api.js          # Servicios principales de API
│   │   ├── monnet-webhook.js      # Manejo de webhooks
│   │   └── monnet-utils.js        # Utilidades y helpers
│   ├── models/
│   │   ├── transaction.js         # Modelo de transacción
│   │   ├── subscription.js        # Modelo de suscripción
│   │   └── virtual-account.js     # Modelo de cuenta virtual
│   ├── controllers/
│   │   ├── payment.js              # Controlador de pagos
│   │   ├── webhook.js              # Controlador de webhooks
│   │   └── subscription.js         # Controlador de suscripciones
│   └── middleware/
│       └── signature-verification.js
├── config/
│   └── monnet.js                  # Configuración de Monnet
├── tests/
│   ├── services/
│   ├── controllers/
│   └── integration/
└── docs/
    └── monnet-api.md               # Documentación interna
```

---

## 25. Flujo de Desarrollo Recomendado

1. **Setup Inicial**: Configurar entorno CERT y obtener credenciales
2. **Implementación Básica**: Crear servicios API fundamentales
3. **Integración Frontend**: Implementar flujo de pago en frontend
4. **Webhooks**: Crear y probar endpoint de webhooks
5. **Testing**: Realizar pruebas exhaustivas en CERT
6. **Certificación**: Probar con Monnet para certificación
7. **Producción**: Migrar a entorno PROD con monitoreo

---

## 26. Monitoreo y Mantenimiento

- **Logs**: Implementar logging estructurado para todas las operaciones
- **Métricas**: Monitorear tasas de éxito, tiempos de respuesta, errores
- **Alertas**: Configurar alertas para fallos críticos
- **Actualizaciones**: Mantenerse informado sobre cambios en API
- **Cumplimiento**: Revisar regularmente cumplimiento regulatorio

---

*Documentación generada el 22 de Febrero de 2026 basada en la documentación oficial de Monnet Payments Payin API v3.*

*Consolida: ANALISIS_COMPLETO_MONNET.md + GUIA_DEFINITIVA_MONNET.md + Guia_Integracion_Monnet.md + Guia_Integracion_Monnet2.md + IMPLEMENTACION_GUIDE.md*