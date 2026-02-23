#  GUÍA DEFINITIVA  MONNET PAYMENTS PAYIN API
> **Audiencia:** Full-Stack Web Developer | **API Version:** v3 | **Fecha:** Febrero 2026 | **Idioma:** Español

---

##  TABLA DE CONTENIDOS

1. [Qué es Monnet Payments?](#1-qué-es-monnet-payments)
2. [Arquitectura General](#2-arquitectura-general)
3. [Ambientes CERT y PROD](#3-ambientes-cert-y-prod)
4. [Credenciales y Autenticación](#4-credenciales-y-autenticación)
5. [Métodos de Pago Disponibles](#5-métodos-de-pago-disponibles)
6. [Flujo Principal  Crear Transacción](#6-flujo-principal--crear-transacción)
7. [Consultar Estado de Transacción](#7-consultar-estado-de-transacción)
8. [Webhooks  Notificaciones de Pago](#8-webhooks--notificaciones-de-pago)
9. [Yape One Shot](#9-yape-one-shot)
10. [Yape On File  Suscripciones OCP](#10-yape-on-file--suscripciones-ocp)
11. [Cuentas Virtuales (Virtual Accounts)](#11-cuentas-virtuales-virtual-accounts)
12. [Códigos de Estado](#12-códigos-de-estado)
13. [Códigos de Error](#13-códigos-de-error)
14. [Documentos de Identificación por País](#14-documentos-de-identificación-por-país)
15. [Browsers Soportados](#15-browsers-soportados)
16. [Estructura de Proyecto Recomendada](#16-estructura-de-proyecto-recomendada)
17. [Servicio Monnet Completo (Node.js)](#17-servicio-monnet-completo-nodejs)
18. [Controlador y Rutas](#18-controlador-y-rutas)
19. [Frontend  Checkout Component (React)](#19-frontend--checkout-component-react)
20. [Troubleshooting Común](#20-troubleshooting-común)
21. [Checklist de Implementación](#21-checklist-de-implementación)
22. [Recursos y Soporte](#22-recursos-y-soporte)

---

## 1. Qué es Monnet Payments?

**Monnet Payments** es una plataforma de intermediación de pagos y cobros online para **Latinoamérica**. Actúa como gateway que conecta tu plataforma con procesadores bancarios y métodos de pago locales de cada país.

Con **una sola integración** puedes:
- Aceptar tarjetas crédito/débito en 6 países LATAM
- Aceptar transferencias bancarias online
- Aceptar pagos en efectivo (farmacias, agentes)
- Integrar billeteras digitales (Yape, etc.)
- Crear **Cuentas Virtuales** para depósitos automáticos
- Gestionar **suscripciones recurrentes** con Yape On File

### Ventajas técnicas
-  No requiere SDK propietario  HTTPS + JSON puro
-  Compatible con cualquier lenguaje backend (Node.js, PHP, Python, Java, etc.)
-  Ambiente de pruebas (CERT/Sandbox) gratuito
-  Volumen: hasta **10 transacciones por segundo**
-  Server-to-Server: el navegador del cliente nunca maneja datos sensibles

### Países y Métodos Disponibles

| País | Tarjeta Crédito | Tarjeta Débito | Transferencia | Efectivo | Wallet | QR |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
|  Perú |  |  |  |  |  |  |
|  Chile |  |  |  |  |  |  |
|  Argentina |  |  |  |  |  |  |
|  México |  |  |  |  |  |  |
|  Colombia |  |  |  |  |  |  |
|  Ecuador |  |  |  |  |  |  |

---

## 2. Arquitectura General

```

                       TU PLATAFORMA FINTECH                          
                                                                       
   Frontend (React/Next.js)          Backend (Node/Laravel/Django)    
               
    Formulario de checkout    Genera SHA512                  
    Redirige al gateway              POST a Monnet API              
    Muestra resultado                Recibe URL de pago             
                                      Expone endpoint webhook        

                                      
              
                            MONNET PAYMENTS API               
                 Valida transacción    Genera link de pago  
                 Reenvía al banco      Notifica resultado    
              
                                     
                  
                BANCO / WALLET                    
                (BCP, Visa,         
                 Yape, SPEI...)        TU WEBHOOK (HTTPS POST)    
                     Verifica SHA512           
                                         Actualiza BD              
                                         Responde HTTP 200         
                                      
```

### Flujo en 7 pasos
1. Usuario selecciona método de pago en tu sitio
2. Tu backend genera el hash SHA512 y hace POST a Monnet
3. Monnet valida y devuelve una URL de pago
4. Tu frontend redirige al usuario a esa URL
5. Usuario completa el pago en el gateway de Monnet/banco
6. Monnet envía un POST asíncrono a tu webhook con el resultado
7. Tu sistema actualiza el estado del pedido/saldo

>  **CRÍTICO:** Que el usuario llegue a tu `payinTransactionOKURL` **NO garantiza** que el pago fue exitoso. Siempre usa el webhook para confirmar.

---

## 3. Ambientes CERT y PROD

###  CERT (Sandbox  Desarrollo y Pruebas)
Ningún cobro real se efectúa.

| Recurso | URL |
|---|---|
| API Payin | `https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments` |
| Consulta de Estado | `https://cert.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations` |
| Back Office | `https://cert.payin.monnetpayments.com/` |
| Suscripciones Yape | `https://cert.subscriptions.payin.monnet.io/api/v1/subscription` |

###  PROD (Producción  Post Certificación)

| Recurso | URL |
|---|---|
| API Payin | `https://payin.api.monnetpayments.com/api-payin/v3/online-payments` |
| Consulta de Estado | `https://apiin.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations` |
| Back Office | `https://payin.monnetpayments.com/` |
| Suscripciones Yape | `https://subscriptions.payin.monnet.io/api/v1/subscription` |

>  Solo puedes usar PROD después de que Monnet apruebe tu **proceso de Certificación**.

---

## 4. Credenciales y Autenticación

### 4.1 Obtener Credenciales

1. Contactar a tu representante de Monnet para acceso al Back Office CERT
2. Iniciar sesión en: `https://cert.payin.monnetpayments.com/`
3. Ir a **Perfil  Admin  Merchant Data**
4. Obtener:

| Credencial | Descripción | Uso |
|---|---|---|
| `payinMerchantID` | Tu ID de comercio asignado por Monnet | Enviado en el body de cada request |
| `KeyMonnet` (Signature Key) | Clave secreta | **NUNCA se envía directamente**  solo para generar hashes |

5. Configurar tu **URL de webhook** en Back Office  Admin  Merchant Data

>  **La `KeyMonnet` jamás debe salir de tu backend.** Trátala como un secret de base de datos.

### 4.2 Variables de Entorno (Configuración)

```env
# .env  NUNCA commitear con datos reales
MONNET_MERCHANT_ID=674
MONNET_KEY=tu_key_monnet_secreta_aqui
MONNET_ENV=cert

# URLs de API
MONNET_API_CERT=https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments
MONNET_API_PROD=https://payin.api.monnetpayments.com/api-payin/v3/online-payments
MONNET_STATUS_CERT=https://cert.monnetpayments.com/ms-experience-payin/merchant
MONNET_STATUS_PROD=https://apiin.monnetpayments.com/ms-experience-payin/merchant

# Para Virtual Accounts (credenciales separadas  pedirlas a Monnet)
MONNET_VA_API_KEY=tu_api_key_va
MONNET_VA_SECRET=tu_secret_va

# Tu dominio
APP_URL=https://tuapp.com
```

### 4.3 Autenticación Standard  SHA-512

Usada para **crear transacciones** y **verificar webhooks**.

**Fórmula:**
```
payinVerification = SHA512(payinMerchantID + payinMerchantOperationNumber + payinAmount + payinCurrency + KeyMonnet)
```

**Implementación por lenguaje:**

```javascript
// Node.js
const crypto = require('crypto');
function generateVerification(merchantId, operationNumber, amount, currency, keyMonnet) {
  const raw = `${merchantId}${operationNumber}${amount}${currency}${keyMonnet}`;
  return crypto.createHash('sha512').update(raw.trim()).digest('hex');
}
```

```python
# Python
import hashlib
def generate_verification(merchant_id, operation_number, amount, currency, key_monnet):
    raw = f"{merchant_id}{operation_number}{amount}{currency}{key_monnet}".strip()
    return hashlib.sha512(raw.encode()).hexdigest()
```

```php
<?php
// PHP
function generateVerification($merchantId, $operationNumber, $amount, $currency, $keyMonnet) {
    $raw = $merchantId . $operationNumber . $amount . $currency . $keyMonnet;
    return openssl_digest(trim($raw), 'sha512');
}
```

```java
// Java
import java.security.MessageDigest;
import org.apache.commons.codec.binary.Hex;

public static String generateVerification(String merchantId, String operationNumber, 
    String amount, String currency, String keyMonnet) throws Exception {
    String raw = (merchantId + operationNumber + amount + currency + keyMonnet).trim();
    MessageDigest md = MessageDigest.getInstance("SHA-512");
    md.update(raw.getBytes("UTF-8"));
    return new String(Hex.encodeHex(md.digest()));
}
```

### 4.4 Autenticación Virtual Accounts  HMAC-SHA512

Usada SOLO para endpoints de **Cuentas Virtuales**.

**Headers requeridos:**
```
X-Api-Key: {tu-api-key-de-va}
X-Timestamp: {unix-timestamp-en-segundos}
X-Signature: HMAC_SHA512(secret-key, timestamp + api-key)
X-Account-deposit-mode: OWNER | ANY
```

```javascript
// Node.js  Generar headers para Virtual Accounts
const crypto = require('crypto');
function generateVAHeaders(apiKey, secretKey) {
  const timestamp = Math.floor(Date.now() / 1000);
  const message = `${timestamp}${apiKey}`;
  const signature = crypto.createHmac('sha512', secretKey).update(message).digest('hex');
  return { 'X-Api-Key': apiKey, 'X-Timestamp': timestamp, 'X-Signature': signature };
}
```

### 4.5 Autenticación Consulta de Estado  SHA-256

Usada SOLO para el endpoint de **consulta de estado**.

```
authorization: SHA256(MerchantID + KeyMonnet)
```

```javascript
// Node.js
const authHeader = crypto.createHash('sha256')
  .update(`${merchantId}${keyMonnet}`)
  .digest('hex');
```

---

## 5. Métodos de Pago Disponibles

El campo `payinMethod` (también llamado `payinProcessorCode`) define el tipo de pago:

| Código | Descripción | Países |
|---|---|---|
| `TCTD` | Tarjeta de Crédito y Débito (ambas) | CL, AR, MX, PE, CO, EC |
| `TC` | Solo Tarjeta de Crédito | CL, MX, PE, AR, CO, EC |
| `TD` | Solo Tarjeta de Débito | CL, MX, PE, AR, CO, EC |
| `Cash` | Pago en Efectivo (farmacias, agentes) | PE, EC, AR, CO, GT |
| `BankTransfer` | Transferencia Bancaria Online | PE, EC, MX, CL, AR, CO, GT, BR |
| `BankTransfer_Businesses` | Transferencia Bancaria para Empresas | Solo PE |
| `Wallet` | Billetera Digital (Yape, etc.) | PE, EC, CO, GT, AR |
| `QR` | Código QR (**uso único  no reutilizable**) | PE, CL |
| `VA` | Cuenta Virtual | MX, AR, PE |

### Consideraciones especiales
- **QR:** De un solo uso. No se puede reutilizar después de ser escaneado.
- **BankTransfer Argentina:** Requiere `payinCustomerCBU` y `payinCustomerCUIT`.
- **Wallet/Yape Perú:** Soporta pagos puntuales (One Shot) y recurrentes (On File).
- **Tiempo de expiración recomendado:** Online/Tarjetas  30 min | Efectivo  120 min

---

## 6. Flujo Principal  Crear Transacción

### 6.1 Endpoint

```
POST https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments
Content-Type: application/json
```

### 6.2 Campos del Request  Tabla Completa

| Campo | Tipo | Req. | Descripción |
|---|---|:---:|---|
| `payinMerchantID` | String |  | Tu ID de comercio en Monnet |
| `payinAmount` | String decimal |  | Monto con 2 decimales: `"150.00"` |
| `payinCurrency` | ISO-4217 |  | `PEN`, `CLP`, `MXN`, `ARS`, `USD`, etc. |
| `payinMerchantOperationNumber` | String max 50 |  | Tu número de orden único |
| `payinMethod` | String |  | `BankTransfer`, `TCTD`, `Cash`, `Wallet`, `QR`, `VA` |
| `payinVerification` | String |  | Hash SHA512 generado en tu backend |
| `payinTransactionOKURL` | String HTTPS |  | URL de redirección cuando el pago es exitoso |
| `payinTransactionErrorURL` | String HTTPS |  | URL de redirección cuando el pago falla |
| `payinExpirationTime` | String Integer |  | Minutos para expirar (30 para online, 120 para efectivo) |
| `payinLanguage` | ISO-639-1 |  | `ES`, `EN`, `PT` |
| `payinCustomerEmail` | String |  | Email del cliente  **nunca vacío** |
| `payinCustomerName` | String max 30 |  | Nombre del cliente  **nunca vacío** |
| `payinCustomerLastName` | String max 30 |  | Apellido del cliente  **nunca vacío** |
| `payinCustomerTypeDocument` | String |  | Tipo de documento: `DNI`, `RUC`, `CE`, `RUT`, etc. |
| `payinCustomerDocument` | String |  | Número de documento |
| `payinCustomerPhone` | String |  | Teléfono  **nunca vacío** |
| `payinCustomerAddress` | String |  | Dirección del cliente |
| `payinCustomerCity` | String |  | Ciudad |
| `payinCustomerRegion` | String |  | Región/Estado (default: `"Lima"` en Perú) |
| `payinCustomerCountry` | String |  | `"Peru"`, `"Chile"`, `"Argentina"`, etc. |
| `payinCustomerZipCode` | String |  | Código postal |
| `payinCustomerID` | String | Opcional | ID interno del cliente en tu sistema |
| `payinRegularCustomer` | String | Opcional | Dejar vacío si no aplica |
| `payinDiscountCoupon` | String | Opcional | Dejar vacío si no aplica |
| `payinFilterBy` | String | Opcional | Dejar vacío |
| `payinProductID` | String |  | ID del producto (puede ser `"0"`) |
| `payinProductDescription` | String |  | Descripción del producto |
| `payinProductAmount` | String decimal |  | Monto del producto |
| `payinProductSku` | String |  | SKU del producto (puede ser `"0"`) |
| `payinProductQuantity` | String |  | Cantidad (puede ser `"1"`) |
| `payinDateTime` | String YYYY-MM-DD |  | Fecha de la transacción |
| `payinCustomerShippingName` | String |  | Nombre de envío (puede ser igual al cliente) |
| `payinCustomerShippingPhone` | String |  | Teléfono de envío |
| `payinCustomerShippingAddress` | String |  | Dirección de envío |
| `payinCustomerShippingCity` | String |  | Ciudad de envío |
| `payinCustomerShippingRegion` | String |  | Región de envío |
| `payinCustomerShippingCountry` | String |  | País de envío |
| `payinCustomerShippingZipCode` | String |  | CP de envío |
| `URLMonnet` | String |  | URL del ambiente Monnet (CERT o PROD) |
| `typePost` | String |  | Siempre: `"json"` |

>  **`payinCustomerName`, `payinCustomerLastName`, `payinCustomerEmail` y `payinCustomerPhone` NUNCA pueden ir vacíos.**

### 6.3 Ejemplo Completo del Request (Perú  Transferencia Bancaria)

```json
{
  "payinMerchantID": "674",
  "payinAmount": "150.00",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "ORDER-20260220-001",
  "payinMethod": "BankTransfer",
  "payinVerification": "a3f8b2...hash_sha512_de_128_chars...d9c1",
  "payinCustomerName": "Juan",
  "payinCustomerLastName": "Perez",
  "payinCustomerEmail": "juan.perez@email.com",
  "payinCustomerPhone": "912345678",
  "payinCustomerTypeDocument": "DNI",
  "payinCustomerDocument": "12345678",
  "payinCustomerID": "user_internal_id_123",
  "payinRegularCustomer": "",
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
  "URLMonnet": "https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments",
  "typePost": "json"
}
```

### 6.4 Response Exitoso

```json
{
  "url": "https://cert.monnetpayments.com/gateway/pay/xxx",
  "payinErrorCode": "0000",
  "payinErrorMessage": "Successfull process",
  "payinTrxOperation": "MONTRX207249992409275755"
}
```

- `payinErrorCode: "0000"`  Éxito. El cliente debe ser redirigido a `url`.
- `payinTrxOperation`  ID de la transacción en Monnet. Guárdalo en tu BD.

### 6.5 Redirigir al Cliente

```javascript
// Frontend: redirigir al gateway de Monnet
if (data.payinErrorCode === '0000') {
  window.location.href = data.url;
}
```

---

## 7. Consultar Estado de Transacción

Endpoint para verificar el estado de una transacción (útil para fallbacks y reconciliación).

### 7.1 Endpoint

```
POST https://cert.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations
authorization: SHA256(MerchantID + KeyMonnet)
Content-Type: application/json
```

>  **El header `authorization` aquí usa SHA256, NO SHA512.**

### 7.2 Request  Por Número de Operación

```json
{ "payinMerchantOperationNumber": "ORDER-20260220-001" }
```

### 7.3 Request  Por Rango de Fechas

```json
{
  "payinStartDate": "2026-01-01",
  "payinEndDate": "2026-02-19"
}
```

>  No usar la fecha actual como `payinStartDate`. Usar como mínimo `hoy - 1`.

### 7.4 Response

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

```javascript
// Ejemplo Node.js
const crypto = require('crypto');
const axios = require('axios');

async function checkPaymentStatus(merchantId, keyMonnet, operationNumber) {
  const authHeader = crypto.createHash('sha256')
    .update(`${merchantId}${keyMonnet}`)
    .digest('hex');

  const response = await axios.post(
    `https://cert.monnetpayments.com/ms-experience-payin/merchant/${merchantId}/operations`,
    { payinMerchantOperationNumber: operationNumber },
    { headers: { 'authorization': authHeader, 'Content-Type': 'application/json' } }
  );
  return response.data;
}
```

---

## 8. Webhooks  Notificaciones de Pago

Cuando Monnet confirma un pago, hace un **HTTP POST asíncrono a tu endpoint**.

### 8.1 Requisitos del Endpoint Webhook

- URL **pública HTTPS** (no localhost)
- Acepta POST con `Content-Type: application/json`
- **Responde HTTP 200 lo antes posible** (antes de procesar la lógica)
- Verifica el hash SHA512 para autenticar la notificación

### 8.2 Payload Recibido

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

### 8.3 Implementación Completa del Webhook (Node.js/Express)

```javascript
const crypto = require('crypto');
const express = require('express');
const app = express();

app.post('/api/webhooks/monnet', express.json(), async (req, res) => {
  // PASO 1: Responder HTTP 200 INMEDIATAMENTE (Monnet reintenta si no recibe 200)
  res.status(200).send('OK');

  try {
    const data = req.body;
    const {
      payinMerchantID, payinMerchantOperationNumber,
      payinAmount, payinCurrency, payinVerification, payinStateID, errorDetails
    } = data;

    // PASO 2: Verificar la firma SHA512
    const expectedHash = crypto
      .createHash('sha512')
      .update(`${payinMerchantID}${payinMerchantOperationNumber}${payinAmount}${payinCurrency}${process.env.MONNET_KEY}`)
      .digest('hex');

    if (expectedHash !== payinVerification) {
      console.warn(' Webhook con firma inválida  posible fraude:', payinMerchantOperationNumber);
      return; // Ignorar pero ya respondimos 200
    }

    // PASO 3: Verificar idempotencia (evitar procesar misma orden dos veces)
    const order = await Order.findOne({ where: { number: payinMerchantOperationNumber } });
    if (!order) return;
    if (order.status === 'COMPLETED') return; // Ya procesado

    // PASO 4: Actuar según el estado
    if (payinStateID === '5') {
      //  PAGO AUTORIZADO  acreditar saldo o liberar servicio
      await order.update({ status: 'COMPLETED' });
      await creditUserBalance(order.userId, parseFloat(payinAmount));
      // Enviar email de confirmación, etc.

    } else if (payinStateID === '6') {
      //  PAGO DENEGADO
      await order.update({
        status: 'FAILED',
        errorCode: errorDetails?.codeErrorTrx,
        errorMessage: errorDetails?.messageErrorTrx
      });
    } else if (payinStateID === '3') {
      //  EXPIRADO
      await order.update({ status: 'EXPIRED' });
    }

  } catch (error) {
    console.error('Error procesando webhook Monnet:', error);
    // No relanzar  ya respondimos 200
  }
});
```

### 8.4 Verificación del Webhook (PHP)

```php
<?php
$rawInput = file_get_contents('php://input');
$data = json_decode($rawInput, true);

// Calcular hash esperado
$expectedHash = hash('sha512',
    $data['payinMerchantID'] .
    $data['payinMerchantOperationNumber'] .
    $data['payinAmount'] .
    $data['payinCurrency'] .
    getenv('MONNET_KEY')
);

if ($expectedHash !== $data['payinVerification']) {
    http_response_code(200); // igual responder 200
    exit;
}

http_response_code(200);
echo 'OK';

// Procesar según payinStateID...
?>
```

---

## 9. Yape One Shot

**Yape One Shot** permite a usuarios en Perú realizar **un pago puntual** vía la app Yape. No almacena consentimiento para cobros futuros.

### 9.1 Flujo

```
1. Usuario selecciona "Yape" en tu web/app
2. Tu backend crea transacción con payinMethod: "Wallet" o código Yape específico
3. Monnet genera el link + deeplink
4. Usuario abre su app Yape y aprueba el pago
5. Monnet notifica el resultado a tu webhook
```

### 9.2 Dispositivos Soportados

| Dispositivo | Comportamiento |
|---|---|
| **MOBILE (iOS/Android)** | Redirige automáticamente vía deeplink a la app Yape |
| **WEB (Desktop)** | Muestra instrucciones en pantalla para que el usuario ingrese su número en la app |

### 9.3 Endpoint

```
POST /api-payin/v3/online-payments
Con: "payinMethod": "Wallet"  (o el código específico de Yape One Shot según tu configuración)
```

---

## 10. Yape On File  Suscripciones OCP

**Yape On File** (One Click Payment) permite crear suscripciones donde el usuario da su consentimiento **una sola vez** y el merchant puede cobrarle automáticamente después.

### 10.1 Tipos de Suscripción

| Tipo | Descripción | Caso de Uso |
|---|---|---|
| `ON_DEMAND` | El merchant inicia cada cobro cuando lo necesita | Recargas, pagos bajo demanda |
| `RECURRENT` | Cobros automáticos en intervalos definidos | Membresías, suscripciones mensuales |

### 10.2 Endpoint  Crear Suscripción

```
POST https://cert.subscriptions.payin.monnet.io/api/v1/subscription
Authorization: Bearer SHA512(merchantId + type + customerId + processorCode + keyPayin)
```

### 10.3 Request  ON_DEMAND (Mobile)

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
    { "key": "MerchantReference", "value": "USER-ID-123" }
  ]
}
```

### 10.4 Request  RECURRENT (cada 3 meses, 150 PEN)

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

### 10.5 Response Exitoso (MOBILE)

```json
{
  "subscriptionId": 12345,
  "status": "PENDING",
  "deepLink": "https://yape.com.pe/app/checkout/ocp/subscription?id=xxx"
}
```

> Redirige al usuario al `deepLink` para que apruebe en su app Yape.

### 10.6 Cancelar Suscripción

```
DELETE https://cert.subscriptions.payin.monnet.io/api/v1/subscription/{subscriptionId}
```

---

## 11. Cuentas Virtuales (Virtual Accounts)

Una **Cuenta Virtual** es un número de cuenta bancaria único asignado a cada usuario. Permite recibir transferencias bancarias locales e identificar automáticamente quién realizó cada depósito.

### 11.1 Países y Tipos de Cuenta

| País | Tipo de Cuenta | Documento Requerido |
|---|---|---|
|  Argentina | `CVU` | `CUIT` |
|  México | `CLABE` | `RFC` |
|  Perú | `CCI` | `DNI` / `RUC` |

### 11.2 Modos de Depósito

| Modo | Descripción |
|---|---|
| `OWNER` | Solo el dueño de la cuenta puede depositar |
| `ANY` | Cualquier persona puede depositar |

### 11.3 Endpoint  Crear Cuenta Virtual

```
POST {base_url}/merchant-payin-accounts/v1/accounts
X-Api-Key: {tu-api-key-va}
X-Timestamp: {unix-timestamp}
X-Signature: HMAC_SHA512(secret-key, timestamp + api-key)
X-Account-deposit-mode: OWNER
Content-Type: application/json
```

### 11.4 Request Body

```json
{
  "owner": {
    "referenceId": "USER-INTERNO-123",
    "type": "PERSON",
    "document": { "type": "DNI", "number": "12345678" },
    "firstName": "Juan",
    "lastName": "Perez",
    "email": "juan.perez@email.com",
    "phone": { "countryCode": "51", "number": "912345678" }
  },
  "account": {
    "category": "VIRTUAL",
    "type": "CCI",
    "country": "PER",
    "currency": "PEN",
    "name": "juan.perez"
  },
  "metadata": { "segmento": "VIP", "plan": "premium" }
}
```

### 11.5 Response Exitoso

```json
{
  "id": "acc_8af98b8c8a4",
  "traceId": "93963JM-38A1A",
  "timestamp": "2026-02-20T15:14:22Z",
  "status": "ACTIVE",
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

### 11.6 Webhook de Depósito en Cuenta Virtual

```json
{
  "version": "1.0",
  "operationId": "7019283",
  "status": { "code": "5", "description": "Aprobado" },
  "merchantId": "674",
  "amount": "200.00",
  "currency": "PEN",
  "payinMethod": "VA",
  "depositDetails": {
    "account": { "id": "acc_8af98b8c8a4", "number": "00219100123456789012" },
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

### 11.7 Otros Endpoints de Cuentas Virtuales

| Operación | Método | Path |
|---|---|---|
| Crear cuenta | POST | `/merchant-payin-accounts/v1/accounts` |
| Obtener detalles | GET | `/merchant-payin-accounts/v1/accounts/{id}` |
| Actualizar información | PATCH | `/merchant-payin-accounts/v1/accounts/{id}` |
| Cambiar estado | PATCH | `/merchant-payin-accounts/v1/accounts/{id}/status` |

---

## 12. Códigos de Estado

| `payinStateID` | `payinState` | Descripción | Acción |
|:---:|---|---|---|
| `1` | Creado | Link de pago generado | Esperar acción del usuario |
| `2` | Pendiente de pago | Usuario en proceso de pago | Esperar |
| `3` | Expirado | Link de pago expiró | Crear nuevo link si necesario |
| `5` | **Autorizado**  | Pago completado exitosamente | **Procesar pedido / acreditar saldo** |
| `6` | **Denegado**  | Pago rechazado | Informar al cliente, ofrecer reintento |
| `9` | Liquidado  | Pago liquidado en banco | Dinero disponible en cuenta |
| `10` | Reembolsado  | Reembolso (transacción no completada) | Notificar cliente |
| `11` | Devuelto  | Reembolso (transacción liquidada) | Notificar cliente |

### Ciclo de vida

```
CREADO (1)  PENDIENTE (2)  AUTORIZADO (5)  LIQUIDADO (9)
                           EXPIRADO (3)
                           DENEGADO (6)
              LIQUIDADO (9)  DEVUELTO (11)
              AUTORIZADO (5)  REEMBOLSADO (10)
```

---

## 13. Códigos de Error

### 13.1 Errores de Creación de Transacción

| Código | Descripción | Solución |
|---|---|---|
| `0000` |  Éxito |  |
| `0001` | `payinMerchantID` vacío | Verificar que lo envías |
| `0002` | `payinAmount` vacío | Verificar que lo envías |
| `0003` | `payinCurrency` vacía | Verificar que la envías |
| `0004` | `payinMerchantOperationNumber` vacío | Verificar que lo envías |
| `0005` | `payinVerification` vacío | Verificar que lo envías |
| `0006` | `payinTransactionErrorURL` vacía | Verificar URL de error |
| `0007` | `payinTransactionOKURL` vacía | Verificar URL de éxito |
| `0009` | `payinMerchantID` incorrecto | Verificar MID en Back Office |
| `0010` | `payinVerification` incorrecto | Revisar cálculo SHA512, orden de parámetros |
| `0011` | Comerciante no habilitado | Contactar soporte Monnet |
| `0015` | Formato de `payinAmount` inválido | Usar formato `"150.00"` con 2 decimales |
| `0017` | Moneda no válida para este merchant | Verificar monedas habilitadas |
| `0022` | Tipo de documento del cliente no existe | Verificar código de documento |
| `0025` | Tipo de documento inválido | Ver tabla de documentos por país |
| `0026` | Número de documento inválido | Verificar longitud y formato |
| `0030` | No cumple reglas de pre-autorización (AR) | Solo Argentina |
| `0040` | CBU requerido (Argentina) | Enviar `payinCustomerCBU` |
| `0041` | CUIT requerido (Argentina) | Enviar `payinCustomerCUIT` |
| `0043` | CBU y CUIT no coinciden | Verificar datos del cliente |
| `0044` | Suscripción inactiva | Verificar estado de suscripción |
| `0048` | `collectorCode` inválido | Verificar código de colector |
| `0099` | Error interno de Payin | Reintentar; si persiste contactar soporte |

### 13.2 Errores de Confirmación de Pago (Banco/Procesador)

| Código | Descripción |
|---|---|
| `9000` |  Éxito |
| `9005` | No honrar  rechazado por banco |
| `9012` | Transacción inválida |
| `9013` | Monto inválido |
| `9014` | Número de tarjeta no existe |
| `9017` | Cliente canceló la operación |
| `9034` | Sospecha de fraude |
| `9039` | Sin cuenta de crédito |
| `9051` | Fondos insuficientes |
| `9054` | Tarjeta expirada |
| `9055` | PIN incorrecto |
| `9057` | Transacción no permitida para titular |
| `9065` | Excedidas transacciones diarias |
| `9097` | Timeout de la operación |
| `9099` | Error genérico |

### 13.3 Errores de Cuentas Virtuales

| Código | Descripción |
|---|---|
| `ERR_MISSING_HEADER` | Header requerido faltante |
| `ERR_INVALID_SIGNATURE` | Firma inválida |
| `ERR_INVALID_DOCUMENT_TYPE` | Tipo de documento no soportado |

---

## 14. Documentos de Identificación por País

| País | Código | Descripción | Longitud | Ejemplo |
|---|---|---|---|---|
|  Argentina | `DNI` | Documento Nacional de Identidad | 8 dígitos | `12345678` |
|  Argentina | `CUIT` | ID Tributario | 11 dígitos | `20123456789` |
|  Argentina | `CUIL` | ID Laboral | 11 dígitos | `27987654321` |
|  Chile | `RUT` | Rol Único Tributario | 7-9 dígitos | `12345678-9` |
|  Chile | `PP` | Pasaporte | 9-10 dígitos | `1234567890` |
|  Perú | `DNI` | Documento Nacional de Identidad | 8 dígitos | `12345678` |
|  Perú | `CE` | Carné de Extranjería | 8-12 alfanum. | `PE123456789` |
|  Perú | `RUC` | Registro Único de Contribuyentes | 9-10 dígitos | `1234567890` |
|  Perú | `PAS` | Pasaporte | 7-12 dígitos | `AB123456` |
|  Ecuador | `CI` | Cédula de Identidad | 10 dígitos | `1234567890` |
|  Ecuador | `RUC` | Registro Único de Contribuyentes | 13 dígitos | `1234567890001` |
|  Ecuador | `PP` | Pasaporte (Cash) | 13 dígitos | `1234567890ABC` |
|  Ecuador | `PAS` | Pasaporte (BankTransfer) | 9-10 dígitos | `12345678` |
|  México | `CURP` | Clave Única Registro Población | 13-18 chars | `ABCD123456HMCMRN09` |
|  México | `RFC` | Registro Federal de Contribuyentes | 10-13 chars | `ABC123456XYZ` |
|  Colombia | `CC` | Cédula de Ciudadanía | 6-10 dígitos | `1234567` |
|  Colombia | `NIT` | Número Identificación Tributaria | 9 dígitos | `123456789` |
|  Guatemala | `DPI` | Documento Personal de Identidad | 13 dígitos | `1234567890123` |
|  Brasil | `CPF` | Cadastro de Pessoas Físicas | 11 dígitos | `12345678901` |
|  Brasil | `CNPJ` | Cadastro Nacional Pessoa Jurídica | 14 dígitos | `12345678901234` |

---

## 15. Browsers Soportados

Para el gateway/voucher de Monnet (donde el usuario realiza el pago):

| Navegador | Versión Mínima |
|---|---|
| Safari | 17 o superior |
| Google Chrome | 130 o superior |
| Microsoft Edge | 130 o superior |
| Mozilla Firefox | 133 o superior |

> Recomiendar a tus usuarios mantener su navegador actualizado.

---

## 16. Estructura de Proyecto Recomendada

```
tu-proyecto-fintech/
 backend/
    src/
       services/
          monnet.service.js          # SDK/Servicio principal de Monnet
          monnet-va.service.js       # Servicio para Cuentas Virtuales
       controllers/
          payment.controller.js      # Crear pago, webhook
          subscription.controller.js # Yape On File
       routes/
          payment.routes.js
       models/
          Order.js                   # Modelo de transacción
          Subscription.js            # Modelo de suscripción Yape
          VirtualAccount.js          # Modelo de cuenta virtual
       middleware/
           auth.middleware.js
    config/
       monnet.config.js               # Configuración centralizada
    .env
 frontend/
    pages/ (o components/)
        Checkout.jsx                   # Formulario de pago
        PagoExito.jsx                  # Página de éxito
        PagoError.jsx                  # Página de error
 tests/
     services/
     controllers/
     integration/
```

## 17. Servicio Monnet Completo (Node.js)

```javascript
// services/monnet.service.js
const crypto = require('crypto');
const axios = require('axios');

class MonnetService {
  constructor() {
    this.merchantId = process.env.MONNET_MERCHANT_ID;
    this.keyMonnet = process.env.MONNET_KEY;
    this.isProd = process.env.MONNET_ENV === 'prod';

    this.apiUrl = this.isProd
      ? 'https://payin.api.monnetpayments.com/api-payin/v3/online-payments'
      : 'https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments';

    this.statusUrl = this.isProd
      ? `https://apiin.monnetpayments.com/ms-experience-payin/merchant/${this.merchantId}/operations`
      : `https://cert.monnetpayments.com/ms-experience-payin/merchant/${this.merchantId}/operations`;
  }

  //  FIRMAS Y VERIFICACIONES 

  /**
   * SHA512 para crear transacciones y verificar webhooks
   */
  generateVerification(operationNumber, amount, currency) {
    const raw = `${this.merchantId}${operationNumber}${amount}${currency}${this.keyMonnet}`;
    return crypto.createHash('sha512').update(raw.trim()).digest('hex');
  }

  /**
   * SHA256 para consultas de estado
   */
  generateStatusAuth() {
    return crypto.createHash('sha256')
      .update(`${this.merchantId}${this.keyMonnet}`)
      .digest('hex');
  }

  /**
   * Verifica el hash SHA512 recibido en el webhook
   */
  verifyWebhookSignature(data) {
    const { payinMerchantID, payinMerchantOperationNumber, payinAmount, payinCurrency, payinVerification } = data;
    const expected = crypto
      .createHash('sha512')
      .update(`${payinMerchantID}${payinMerchantOperationNumber}${payinAmount}${payinCurrency}${this.keyMonnet}`)
      .digest('hex');
    return expected === payinVerification;
  }

  //  CREAR TRANSACCIÓN 

  /**
   * Crea una transacción de pago en Monnet
   * @param {Object} params
   * @param {string} params.orderNumber - Número de orden único
   * @param {string} params.amount      - Monto: "150.00"
   * @param {string} params.currency    - ISO-4217: "PEN", "CLP", etc.
   * @param {string} params.method      - "BankTransfer", "TCTD", "Cash", "Wallet", "QR", "VA"
   * @param {Object} params.customer    - Datos del cliente
   * @param {Object} params.product     - Datos del producto
   * @param {string} params.language    - "ES" | "EN" | "PT"
   * @param {number} params.expirationTime - Minutos para expirar
   */
  async createTransaction({ orderNumber, amount, currency, method, customer, product = {}, language = 'ES', expirationTime = 30 }) {
    const verification = this.generateVerification(orderNumber, amount, currency);
    const today = new Date().toISOString().split('T')[0];

    const payload = {
      payinMerchantID: this.merchantId,
      payinAmount: String(parseFloat(amount).toFixed(2)),
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
      payinCustomerID: String(customer.id || ''),
      payinRegularCustomer: '',
      payinDiscountCoupon: '',
      payinLanguage: language,
      payinExpirationTime: String(expirationTime),
      payinDateTime: today,
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
      payinProductID: String(product.id || '0'),
      payinProductDescription: product.description || 'Pago',
      payinProductAmount: String(parseFloat(amount).toFixed(2)),
      payinProductSku: String(product.sku || '0'),
      payinProductQuantity: String(product.quantity || '1'),
      URLMonnet: this.apiUrl,
      typePost: 'json'
    };

    try {
      const response = await axios.post(this.apiUrl, payload, {
        headers: { 'Content-Type': 'application/json' },
        timeout: 30000
      });
      return response.data;
    } catch (error) {
      console.error('[Monnet] Error creando transacción:', error.response?.data || error.message);
      throw error;
    }
  }

  //  CONSULTAR ESTADO 

  /**
   * Consulta el estado de una transacción por número de operación
   */
  async getTransactionStatus(operationNumber) {
    const authHeader = this.generateStatusAuth();
    try {
      const response = await axios.post(
        this.statusUrl,
        { payinMerchantOperationNumber: operationNumber },
        { headers: { 'authorization': authHeader, 'Content-Type': 'application/json' }, timeout: 15000 }
      );
      return response.data;
    } catch (error) {
      console.error('[Monnet] Error consultando estado:', error.response?.data || error.message);
      throw error;
    }
  }

  /**
   * Consulta transacciones por rango de fechas
   * NOTA: startDate debe ser al menos ayer (hoy - 1)
   */
  async getTransactionsByDate(startDate, endDate) {
    const authHeader = this.generateStatusAuth();
    try {
      const response = await axios.post(
        this.statusUrl,
        { payinStartDate: startDate, payinEndDate: endDate },
        { headers: { 'authorization': authHeader, 'Content-Type': 'application/json' } }
      );
      return response.data;
    } catch (error) {
      console.error('[Monnet] Error consultando por fecha:', error.response?.data || error.message);
      throw error;
    }
  }
}

module.exports = new MonnetService();
```

---

## 18. Controlador y Rutas

```javascript
// controllers/payment.controller.js
const monnetService = require('../services/monnet.service');

/**
 * POST /api/payments/create
 * Inicia un pago  requiere autenticación de usuario
 */
exports.createPayment = async (req, res) => {
  try {
    const { amount, currency, method, productDescription } = req.body;
    const user = req.user; // inyectado por middleware de auth

    // 1. Generar número de orden único
    const orderNumber = `ORD-${Date.now()}-${user.id}`;

    // 2. Guardar orden en BD con estado PENDING
    const order = await Order.create({
      number: orderNumber,
      userId: user.id,
      amount: parseFloat(amount).toFixed(2),
      currency,
      paymentMethod: method,
      status: 'PENDING'
    });

    // 3. Crear transacción en Monnet
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
        country: user.country,
        city: user.city,
        region: user.region,
        address: user.address,
        zipCode: user.zipCode
      },
      product: {
        id: '0',
        description: productDescription || 'Pago',
        amount: parseFloat(amount).toFixed(2)
      }
    });

    // 4. Verificar respuesta de Monnet
    if (result.payinErrorCode !== '0000') {
      await order.update({ status: 'ERROR', errorMessage: result.payinErrorMessage });
      return res.status(400).json({ success: false, error: result.payinErrorMessage, code: result.payinErrorCode });
    }

    // 5. Guardar ID de transacción Monnet
    await order.update({ monnetTrxId: result.payinTrxOperation });

    return res.json({ success: true, redirectUrl: result.url });
  } catch (error) {
    console.error('[Payment] Error creando pago:', error);
    res.status(500).json({ success: false, error: 'Error interno del servidor' });
  }
};

/**
 * POST /api/payments/webhook/monnet
 * Recibe notificaciones de pago de Monnet  SIN autenticación de usuario
 */
exports.handleWebhook = async (req, res) => {
  // SIEMPRE responder 200 primero (Monnet reintenta si no recibe 200 rápido)
  res.status(200).send('OK');

  try {
    const data = req.body;

    // 1. Verificar firma SHA512
    if (!monnetService.verifyWebhookSignature(data)) {
      console.warn('[Webhook] Firma inválida  posible fraude:', data.payinMerchantOperationNumber);
      return;
    }

    const { payinMerchantOperationNumber, payinStateID, payinAmount, errorDetails } = data;

    // 2. Buscar orden en BD (idempotencia)
    const order = await Order.findOne({ where: { number: payinMerchantOperationNumber } });
    if (!order || order.status === 'COMPLETED') return;

    // 3. Verificar que el monto coincide (seguridad extra)
    if (parseFloat(payinAmount) !== parseFloat(order.amount)) {
      console.error('[Webhook] Monto no coincide:', { received: payinAmount, expected: order.amount });
      return;
    }

    // 4. Procesar según estado
    switch (payinStateID) {
      case '5': // AUTORIZADO
        await order.update({ status: 'COMPLETED', completedAt: new Date() });
        await creditUserBalance(order.userId, parseFloat(payinAmount));
        // await sendConfirmationEmail(order.userId);
        break;

      case '6': // DENEGADO
        await order.update({
          status: 'FAILED',
          errorCode: errorDetails?.codeErrorTrx,
          errorMessage: errorDetails?.messageErrorTrx
        });
        break;

      case '3': // EXPIRADO
        await order.update({ status: 'EXPIRED' });
        break;

      default:
        console.log('[Webhook] Estado recibido:', payinStateID, 'para orden:', payinMerchantOperationNumber);
    }

  } catch (error) {
    console.error('[Webhook] Error procesando:', error);
  }
};

/**
 * GET /api/payments/status/:operationNumber
 * Consulta manual del estado (para admin o fallbacks)
 */
exports.getPaymentStatus = async (req, res) => {
  try {
    const { operationNumber } = req.params;
    const result = await monnetService.getTransactionStatus(operationNumber);
    res.json({ success: true, data: result });
  } catch (error) {
    res.status(500).json({ success: false, error: 'Error consultando estado' });
  }
};
```

```javascript
// routes/payment.routes.js
const express = require('express');
const router = express.Router();
const { createPayment, handleWebhook, getPaymentStatus } = require('../controllers/payment.controller');
const authMiddleware = require('../middleware/auth');

// Rutas protegidas (requieren login)
router.post('/create', authMiddleware, createPayment);
router.get('/status/:operationNumber', authMiddleware, getPaymentStatus);

// Webhook de Monnet  SIN auth de usuario, la seguridad es el hash SHA512
router.post('/webhook/monnet', handleWebhook);

module.exports = router;
```

---

## 19. Frontend  Checkout Component (React)

```jsx
// components/Checkout.jsx
import { useState } from 'react';
import axios from 'axios';

const PAYMENT_METHODS = [
  { value: 'BankTransfer', label: 'Banca Online' },
  { value: 'TCTD',         label: 'Tarjeta Crédito/Débito' },
  { value: 'Cash',         label: 'Efectivo (farmacias/agentes)' },
  { value: 'Wallet',       label: 'Yape' },
  { value: 'QR',           label: 'Código QR' }
];

export default function Checkout({ amount = '150.00', currency = 'PEN' }) {
  const [method, setMethod] = useState('BankTransfer');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const handlePay = async () => {
    setLoading(true);
    setError(null);

    try {
      const { data } = await axios.post('/api/payments/create', {
        amount,
        currency,
        method,
        productDescription: 'Recarga de saldo'
      });

      if (data.success && data.redirectUrl) {
        // Redirigir al gateway de Monnet
        window.location.href = data.redirectUrl;
      } else {
        setError(data.error || 'Error al iniciar el pago');
      }
    } catch (err) {
      setError('Error de conexión. Intenta de nuevo.');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="checkout">
      <h2>Método de pago</h2>
      <select value={method} onChange={e => setMethod(e.target.value)} disabled={loading}>
        {PAYMENT_METHODS.map(m => (
          <option key={m.value} value={m.value}>{m.label}</option>
        ))}
      </select>

      <div className="amount-display">
        Total: {currency} {amount}
      </div>

      {error && <p className="error">{error}</p>}

      <button onClick={handlePay} disabled={loading}>
        {loading ? 'Procesando...' : `Pagar ${currency} ${amount}`}
      </button>

      <p className="disclaimer">
        Serás redirigido al portal seguro de Monnet Payments para completar tu pago.
      </p>
    </div>
  );
}
```

```jsx
// pages/pago/exito.jsx
import { useEffect } from 'react';
import { useSearchParams } from 'react-router-dom';

export default function PagoExito() {
  const [searchParams] = useSearchParams();
  const orderNumber = searchParams.get('order');

  // IMPORTANTE: Esta página NO confirma el pago.
  // La confirmación real llega via webhook.
  // Muestra una pantalla de "procesando" y consulta el estado.

  return (
    <div>
      <h1> Pago en proceso</h1>
      <p>Tu pago está siendo procesado. Recibirás una confirmación.</p>
      <p>Número de referencia: {orderNumber}</p>
    </div>
  );
}
```

---

## 20. Troubleshooting Común

###  Error 0010: Verificación Incorrecta

**Síntoma:** `"Error in payinVerification (it's wrong)"`

**Causas y soluciones:**
- **Orden de parámetros incorrecto:** Debe ser exactamente `MerchantID + OperationNumber + Amount + Currency + KeyMonnet`
- **Espacios en blanco:** Aplicar `.trim()` a todos los valores antes de concatenar
- **Monto con formato diferente:** Si envías `"150.00"` en el request, usar exactamente `"150.00"` en el hash (no `"150"` o `150.0`)
- **Usando SHA256 en vez de SHA512:** La verificación de transacciones usa SHA512
- **Encoding incorrecto:** Asegurar UTF-8
- **KeyMonnet con espacios:** Verificar que no tenga espacios al inicio o al final

###  Error 0009: MerchantID Inválido

**Causas:**
- Typo en el MerchantID
- Usando credenciales CERT en PROD (o viceversa)
- Cuenta no activa

**Solución:** Verificar MerchantID en Back Office  Perfil

###  Error 0011: Comerciante No Habilitado

**Causas:** Cuenta suspendida o método de pago no habilitado para tu cuenta

**Solución:** Contactar soporte de Monnet para verificar estado y métodos habilitados

###  Webhook No Llega

**Checklist:**
- [ ] URL de webhook configurada en Back Office  Admin  Merchant Data
- [ ] URL es pública y HTTPS (no localhost)
- [ ] El servidor responde HTTP 200 (verificar logs del servidor)
- [ ] Firewall/security groups permiten requests desde IPs de Monnet
- [ ] Certificado SSL válido y no expirado
- [ ] Usar `webhook.site` para debugging rápido durante desarrollo

###  Cliente Ve Página en Blanco/Error en Gateway Monnet

**Causas:**
- Navegador desactualizado (ver sección de browsers soportados)
- JavaScript deshabilitado
- Extensiones de bloqueo (AdBlock, etc.)
- Conexión inestable

**Solución:** Solicitar al cliente abrir con Chrome 130+, Safari 17+ en modo incógnito

###  Pago Exitoso pero No Llega el Webhook

**Acción:** Implementar un endpoint de consulta de estado como fallback:
1. Crear un cron job que consulte transacciones pendientes usando el endpoint de estado
2. La consulta usa SHA256 para autenticación (no SHA512)
3. Usar la fecha de ayer como `payinStartDate` mínimo

###  Webhook Llega Dos Veces (Duplicado)

Monnet puede reenviar el webhook si no recibe HTTP 200 a tiempo.

**Solución:** Implementar **idempotencia** en tu webhook:
```javascript
// Verificar si ya existe la orden y está completada
const order = await Order.findOne({ where: { number: payinMerchantOperationNumber } });
if (!order || order.status === 'COMPLETED') return; // ignorar duplicado
```

---

## 21. Checklist de Implementación

###  Fase 1: Preparación
- [ ] Contactar representante Monnet para acceso CERT
- [ ] Obtener `payinMerchantID` y `KeyMonnet` desde el Back Office
- [ ] Configurar URL de webhook en Back Office (Admin  Merchant Data)
- [ ] Guardar credenciales en `.env` (NUNCA hardcodear ni commitear)
- [ ] Configurar HTTPS en tu servidor (SSL válido y vigente)

###  Fase 2: Backend
- [ ] Implementar función de generación de hash SHA512
- [ ] Implementar endpoint `POST /api/payments/create`
- [ ] Implementar endpoint `POST /api/webhooks/monnet`
- [ ] Implementar validación de firma en webhook
- [ ] Implementar idempotencia en el webhook
- [ ] Crear modelo de BD para transacciones (state machine)
- [ ] Implementar endpoint de consulta de estado (fallback SHA256)
- [ ] Configurar logging detallado de todas las operaciones

###  Fase 3: Frontend
- [ ] Crear UI de selección de método de pago
- [ ] Implementar llamada al backend para crear pago
- [ ] Implementar redirección a la URL de pago de Monnet
- [ ] Crear página de éxito (con aviso de que es "en proceso")
- [ ] Crear página de error (con opción de reintento)

###  Fase 4: Pruebas en CERT
- [ ] Probar flujo completo con cada método de pago (TCTD, BankTransfer, Cash, Wallet)
- [ ] Probar flujo de error (datos incorrectos, fondos insuficientes)
- [ ] Verificar que el webhook llega y se procesa correctamente
- [ ] Verificar validación de firma (enviar webhook con hash incorrecto  debe ignorarse)
- [ ] Probar idempotencia (webhook duplicado  no duplicar saldo)
- [ ] Verificar que el monto acreditado coincide con el del webhook
- [ ] Probar expiración de transacciones

###  Fase 5: Seguridad
- [ ] `KeyMonnet` guardada en variable de entorno, nunca en frontend
- [ ] Verificar firma SHA512 en TODOS los webhooks
- [ ] HTTPS en todas las URLs (webhook, OK URL, Error URL)
- [ ] Rate limiting en endpoint de creación de pagos (máx. 10 TPS)
- [ ] No loguear datos sensibles (números de tarjeta, keys)
- [ ] Validar que el monto del webhook coincide con el de la BD antes de acreditar
- [ ] Implementar CORS correctamente en el backend

###  Fase 6: Monitoreo
- [ ] Logging estructurado de todas las transacciones
- [ ] Alertas para errores críticos (webhook no procesado, firma inválida)
- [ ] Dashboard con métricas de pagos (éxitos, errores, expirados)
- [ ] Healthcheck para el endpoint de webhook

###  Fase 7: Certificación
- [ ] Contactar representante Monnet para agendar sesión de certificación
- [ ] Un integrador de Monnet probará:
  - [ ] Flujo de checkout completo
  - [ ] Creación de transacciones
  - [ ] Recepción de notificaciones
  - [ ] Manejo de errores
- [ ] Obtener aprobación de certificación (requisito para acceder a PROD)

###  Fase 8: Go to Producción
- [ ] Obtener credenciales PROD de Monnet
- [ ] Actualizar variables de entorno a PROD
- [ ] Cambiar URLs: `cert.payin.api.*`  `payin.api.*`
- [ ] Configurar URL de webhook en Back Office PROD
- [ ] Realizar test final con monto pequeño real
- [ ] Monitorear intensivamente las primeras 24-48 horas
- [ ] Tener plan de rollback listo (vuelta a CERT)

---

## 22. Recursos y Soporte

###  Resumen de Endpoints por Ambiente

| Operación | Método | CERT | PROD |
|---|---|---|---|
| Crear transacción | POST | `https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments` | `https://payin.api.monnetpayments.com/api-payin/v3/online-payments` |
| Consultar estado | POST | `https://cert.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations` | `https://apiin.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations` |
| Suscripción Yape (crear) | POST | `https://cert.subscriptions.payin.monnet.io/api/v1/subscription` | `https://subscriptions.payin.monnet.io/api/v1/subscription` |
| Suscripción Yape (cancelar) | DELETE | `https://cert.subscriptions.payin.monnet.io/api/v1/subscription/{id}` | `https://subscriptions.payin.monnet.io/api/v1/subscription/{id}` |
| Cuenta Virtual (crear) | POST | `{base_url}/merchant-payin-accounts/v1/accounts` | `{base_url}/merchant-payin-accounts/v1/accounts` |
| Cuenta Virtual (detalles) | GET | `{base_url}/merchant-payin-accounts/v1/accounts/{id}` | `{base_url}/merchant-payin-accounts/v1/accounts/{id}` |
| Cuenta Virtual (actualizar) | PATCH | `{base_url}/merchant-payin-accounts/v1/accounts/{id}` | `{base_url}/merchant-payin-accounts/v1/accounts/{id}` |
| Cuenta Virtual (estado) | PATCH | `{base_url}/merchant-payin-accounts/v1/accounts/{id}/status` | `{base_url}/merchant-payin-accounts/v1/accounts/{id}/status` |
| Back Office | Web | `https://cert.payin.monnetpayments.com/` | `https://payin.monnetpayments.com/` |

###  Resumen de Autenticación

| Operación | Tipo de Hash | Campos |
|---|---|---|
| Crear transacción | SHA512 en body `payinVerification` | `MerchantID + OperationNumber + Amount + Currency + KeyMonnet` |
| Verificar webhook | SHA512 comparar `payinVerification` | `MerchantID + OperationNumber + Amount + Currency + KeyMonnet` |
| Consulta de estado | SHA256 en header `authorization` | `MerchantID + KeyMonnet` |
| Suscripción Yape | SHA512 en header `Authorization: Bearer` | `merchantId + type + customerId + processorCode + keyPayin` |
| Virtual Accounts | HMAC-SHA512 en header `X-Signature` | `HMAC(secret-key, timestamp + api-key)` |

###  Dependencias Recomendadas (Node.js)

```json
{
  "dependencies": {
    "axios": "^1.6.0",
    "express": "^4.18.0",
    "dotenv": "^16.0.0"
  },
  "note": "crypto es módulo built-in de Node.js, no necesita instalación"
}
```

###  Contacto y Soporte

```
 Email:    support@monnetpayments.com
 Web:      https://www.monnetpayments.com/
 Docs:    https://payinmonnetpayments.readme.io/
 Back Office CERT: https://cert.payin.monnetpayments.com/
 Back Office PROD: https://payin.monnetpayments.com/
```

Al contactar soporte, incluir siempre:
- Tu `payinMerchantID`
- El error específico (código y mensaje)
- Logs relevantes del request/response
- Pasos para reproducir el problema
- Ambiente (CERT o PROD)

---

###  Reglas de Oro  NUNCA Olvidar

```
OBLIGATORIO:
 HTTPS en todos los endpoints (webhook, OK URL, Error URL)
 Guardar KeyMonnet SOLO en variables de entorno del backend
 Verificar firma SHA512 en CADA webhook recibido
 Responder HTTP 200 al webhook INMEDIATAMENTE (procesar de forma asíncrona)
 Implementar idempotencia en el webhook
 Certificar con Monnet antes de ir a producción
 Verificar que el monto del webhook coincide con tu BD

PROHIBIDO:
 Hardcodear credenciales en el código
 Exponer KeyMonnet en el frontend
 Confiar en la URL de éxito como confirmación de pago
 Loguear datos sensibles
 Usar credenciales CERT en PROD (o viceversa)
 Almacenar números completos de tarjeta
```

---

*Guía Definitiva generada el 22 de Febrero de 2026*  
*Consolida: ANALISIS_COMPLETO_MONNET.md + GUIA_COMPLETA_IMPLEMENTACION_MONNET.md + Guia_Integracion_Monnet.md + Guia_Integracion_Monnet2.md + IMPLEMENTACION_GUIDE.md*
