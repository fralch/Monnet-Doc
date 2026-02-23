# 📚 ANÁLISIS COMPLETO - MONNET PAYMENTS PAYIN API

**Fecha:** 20 de Febrero de 2026  
**Documento:** Guía técnica completa y detallada para la integración de Monnet Payments Payin API  
**Audiencia:** Full Stack Web Developers

---

## 📑 Tabla de Contenidos

1. [Introducción General](#introducción-general)
2. [¿Qué es Monnet Payments?](#qué-es-monnet-payments)
3. [Características Principales](#características-principales)
4. [Preliminares: ¿Qué Necesitas?](#preliminares-qué-necesitas)
5. [Guía de Inicio](#guía-de-inicio)
6. [Métodos de Pago Soportados](#métodos-de-pago-soportados)
7. [Arquitectura General de la API](#arquitectura-general-de-la-api)
8. [Flujo de Transacciones](#flujo-de-transacciones)
9. [Códigos de Estado](#códigos-de-estado)
10. [Códigos de Error](#códigos-de-error)
11. [Documentos de Identificación por País](#documentos-de-identificación-por-país)
12. [Verificación y Seguridad](#verificación-y-seguridad)
13. [Funcionalidades Avanzadas](#funcionalidades-avanzadas)
14. [Especificaciones Técnicas](#especificaciones-técnicas)
15. [Checklist de Implementación](#checklist-de-implementación)

---

## 🌟 Introducción General

### Propósito del Documento

Este documento es una guía **técnica y práctica** para que entiendas completamente qué es Monnet Payments, cómo funciona, y qué necesitas para implementarlo como Full Stack Developer. Incluye explicaciones, flujos, códigos de ejemplo y todos los detalles técnicos necesarios.

### Scope de Cobertura

Esta documentación cubre la **API Payin de Monnet Payments**, que es el servicio de recepción de pagos en línea con soporte para múltiples métodos de pago y países en Latinoamérica.

---

## 🏢 ¿Qué es Monnet Payments?

### Definición

**Monnet Payments** es una plataforma de **intermediación de pagos y cobros en línea** que actúa como puente entre:
- **Tu negocio/plataforma** (el comerciante)
- **Tus clientes** (los compradores)
- **Instituciones financieras** (bancos, redes de tarjetas, procesadores de pago)

### Misión

Permitir que empresas puedan **recibir pagos** de clientes en **Latinoamérica** ofreciendo múltiples métodos de pago locales, adaptados a cada país y preferencia del consumidor.

### ¿Por Qué Usar Monnet?

| Beneficio | Descripción |
|-----------|-------------|
| **Cobertura Regional** | Acepta pagos en Perú, Chile, Argentina, México, Colombia, Ecuador |
| **Múltiples Métodos** | Tarjetas, transferencias, efectivo, billeteras digitales, códigos QR |
| **Integración Sencilla** | API HTTPS estándar, funciona con cualquier lenguaje de programación |
| **Sin Aplicaciones** | No necesitas instalar software propietario |
| **Ambiente de Pruebas** | CERT environment para testing sin riesgos |
| **Alta Seguridad** | Comunicación servidor-a-servidor sin intermediarios |
| **Escalabilidad** | Soporta hasta 10 transacciones por segundo |

---

## ✨ Características Principales

### 1. **Aceptación de Múltiples Métodos de Pago**

```
┌─────────────────────────────────────┐
│     MÉTODOS DE PAGO SOPORTADOS      │
├─────────────────────────────────────┤
│ • Tarjetas Crédito/Débito           │
│ • Transferencias Bancarias Online   │
│ • Billeteras Digitales (Wallet)     │
│ • Pagos en Efectivo                 │
│ • Códigos QR                         │
│ • Cuentas Virtuales                 │
│ • Yape (Billetera Peruana)          │
└─────────────────────────────────────┘
```

### 2. **Cobertura Geográfica**

| País | Tarjetas | Transferencia | Efectivo | Billeteras | Códigos QR |
|------|----------|---------------|----------|-----------|-----------|
| 🇵🇪 Perú | ✅ | ✅ | ✅ | ✅ | ✅ |
| 🇨🇱 Chile | ✅ | ✅ | ✅ | ✅ | ✅ |
| 🇦🇷 Argentina | ✅ | ✅ | ✅ | ✅ | ❌ |
| 🇲🇽 México | ✅ | ✅ | ❌ | ✅ | ❌ |
| 🇨🇴 Colombia | ✅ | ✅ | ✅ | ✅ | ❌ |
| 🇪🇨 Ecuador | ✅ | ✅ | ✅ | ✅ | ❌ |

### 3. **Modelo de Arquitectura**

- **Sin Aplicaciones Propietarias:** HTTPS puro
- **Server-to-Server:** Mayor seguridad
- **Browser-Free:** El navegador del cliente no maneja datos sensibles
- **Estándar HTTP:** POST/GET convencionales

### 4. **Flexibilidad por Lenguaje**

```
Soporta:
✅ ASP / ASP.NET
✅ Java
✅ PHP
✅ Python
✅ Node.js
✅ Ruby
✅ Go
✅ Cualquier lenguaje que haga HTTP HTTPS
```

---

## 🔧 Preliminares: ¿Qué Necesitas?

### Para Full Stack Developers

#### **Backend (Servidor)**

```javascript
// Requisitos técnicos
✅ Capacidad de hacer HTTP POST/GET requests
✅ Manejo de JSON
✅ Validación de signatures (SHA-256, SHA-512, HMAC-SHA512)
✅ Implementar webhooks/listeners
✅ Persistencia de data (bases de datos)
✅ Manejo de transacciones asincrónicas
```

#### **Frontend (Cliente)**

```javascript
// Requisitos técnicos
✅ Formularios para seleccionar método de pago
✅ Manejo de redirecciones (POST)
✅ Integración con Yape (deeplinks para móvil)
✅ Manejo de URLs de éxito/error
✅ UX para pagos por QR o transferencias
```

#### **Infraestructura**

```
✅ Servidor con HTTPS obligatoriamente
✅ Certificados SSL válidos
✅ URL pública para webhook (notificaciones)
✅ Base de datos para almacenar transacciones
✅ Variables de entorno para credenciales
```

### Credenciales Requeridas

```
1. payinMerchantID
   - Tu ID único como comerciante en Monnet
   - Se obtiene en el Back Office

2. KeyMonnet (API Key)
   - Clave para firmar requests
   - Confidencial, guardar en variables de entorno

3. Signature Key
   - Clave adicional para verificación
   - Se envía en headers HTTP
```

---

## 🚀 Guía de Inicio

### Paso 1: Acceder al Back Office

```
CERT (Pruebas):  https://cert.payin.monnetpayments.com/
PROD (Producción): https://payin.monnetpayments.com/
```

### Paso 2: Obtener Credenciales

1. Inicia sesión en el Back Office
2. Ve a **Profile / Mi Perfil**
3. Encuentra:
   - **API Key** (payinMerchantID, KeyMonnet)
   - **Signature Key**
4. Configura tu **URL de Webhook** en **Admin > Merchant Data**

### Paso 3: Configurar URLs

```python
# Variables a usar en tu código
CERT_BASE_URL = "https://cert.payin.api.monnetpayments.com/"
PROD_BASE_URL = "https://payin.api.monnetpayments.com/"

# Endpoints principales
CREATE_TRANSACTION_ENDPOINT = "{base_url}api-payin/v3/online-payments"
GET_STATUS_ENDPOINT = "{base_url}ms-experience-payin/merchant/{MID}/operations"

# Tu servidor debe tener:
WEBHOOK_URL = "https://tuciruelo.com/webhooks/monnet"
OK_URL = "https://tuciruelo.com/payment-success"
ERROR_URL = "https://tuciruelo.com/payment-error"
```

### Paso 4: Configurar Variables de Entorno

```bash
# .env (nunca commituear)
MONNET_MERCHANT_ID=12345
MONNET_KEY=your_secret_key_here
MONNET_SIGNATURE_KEY=your_signature_key
MONNET_WEBHOOK_URL=https://tudominio.com/webhook
MONNET_ENVIRONMENT=cert  # o "prod"
```

---

## 💳 Métodos de Pago Soportados

### Tabla de Referencia Completa

| Tag | Descripción | Países Disponibles |
|-----|-------------|-------------------|
| **TCTD** | Tarjeta Crédito/Débito | 🇨🇱 🇦🇷 🇲🇽 🇵🇪 🇨🇴 🇪🇨 |
| **TC** | Solo Tarjeta Crédito | 🇨🇱 🇲🇽 🇵🇪 🇦🇷 🇨🇴 🇪🇨 |
| **TD** | Solo Tarjeta Débito | 🇨🇱 🇲🇽 🇵🇪 🇦🇷 🇨🇴 🇪🇨 |
| **Cash** | Pago en Efectivo | 🇵🇪 🇪🇨 🇦🇷 🇨🇴 🇬🇹 |
| **BankTransfer** | Transferencia Bancaria | 🇵🇪 🇪🇨 🇲🇽 🇨🇱 🇦🇷 🇨🇴 🇬🇹 🇧🇷 |
| **BankTransfer_Businesses** | Transferencia para Empresas | 🇵🇪 |
| **Wallet** | Billetera Digital | 🇵🇪 🇪🇨 🇨🇴 🇬🇹 🇦🇷 |
| **QR** | Código QR | 🇵🇪 🇨🇱 |
| **VA** | Cuenta Virtual | 🇲🇽 🇦🇷 🇵🇪 |

### Consideraciones Especiales

```
⚠️ Códigos QR (Perú y Chile):
   - De un solo uso
   - No se puede reutilizar después de ser usado
   - Ideal para puntos de venta

⚠️ Transferencias Bancarias:
   - Requieren CBU en Argentina
   - Requieren CUIT para empresas en Argentina
   - Demora de acreditación depende del banco

⚠️ Yape (Perú):
   - Wallets digitales con suscripción
   - Soporta pagos puntuales y recurrentes
   - Requiere app Yape instalada en dispositivo
```

---

## 🏗️ Arquitectura General de la API

### Flujo de Comunicación

```
┌─────────────────────────────────────────────────────────────────┐
│                     ARQUITECTURA DE MONNET                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  Tu Aplicación (Backend)                                          │
│  ├─ 1. Crea Transacción (POST)                                   │
│  └─ 2. Recibe respuesta + link de pago                           │
│                                                                   │
│  Cliente (Navegador/App)                                         │
│  ├─ 3. Es redirigido a Monnet Payment Gateway                    │
│  ├─ 4. Realiza pago                                              │
│  └─ 5. Es devuelto a tu sitio (OK o ERROR)                       │
│                                                                   │
│  Tu Servidor (Webhook Listener)                                  │
│  └─ 6. Recibe notificación de Monnet (async)                     │
│                                                                   │
│  Instituciones Financieras                                       │
│  └─ Procesan los pagos en background                             │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Componentes Clave

| Componente | Responsabilidad | Ejemplo |
|-----------|-----------------|---------|
| **Merchant ID** | Tu identificador único | `payinMerchantID: 12345` |
| **API Key** | Autenticación | `KeyMonnet: abc123xyz789` |
| **Signature** | Verificación de integridad | SHA-512 hash |
| **Transaction ID** | Referencia única del pago | `payinMerchantOperationNumber` |
| **Webhook** | Notificación de resultado | POST a tu servidor |

### Headers HTTP Requeridos

```http
POST /api-payin/v3/online-payments HTTP/1.1
Host: cert.payin.api.monnetpayments.com
Content-Type: application/json
Authorization: {tu_signature_sha256}
X-Merchant-ID: {payinMerchantID}

{
  "body": { ... }
}
```

---

## 🔄 Flujo de Transacciones

### Flujo Completo: Paso a Paso

#### **Paso 1: Cliente Selecciona Pago con Monnet**

```
Tu sitio web/app
    ↓
Cliente hace clic en "Pagar con Monnet"
    ↓
Tu backend recibe el request de purchase
```

#### **Paso 2: Tu Backend Crea Transacción**

```java
// Ejemplo: Tu backend hace POST a Monnet
POST https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments

{
  "payinMerchantID": "12345",
  "payinAmount": "50.00",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "ORDER-001-2024",
  "payinMethod": "TCTD",
  "payinTransactionOKURL": "https://tudominio.com/success",
  "payinTransactionErrorURL": "https://tudominio.com/error",
  "payinVerification": "{hash_sha512}"
}
```

**Firma (Verification):**
```
SHA512(
  payinMerchantID +
  payinMerchantOperationNumber +
  payinAmount +
  payinCurrency +
  KeyMonnet
)
```

#### **Paso 3: Monnet Responde**

```json
{
  "payinStateID": "1",
  "payinState": "Creado",
  "payinErrorCode": "0000",
  "payinErrorMessage": "Success",
  "payinLink": "https://cert.payin.monnetpayments.com/pay?id=abc123def456"
}
```

#### **Paso 4: Redirigir Cliente a Monnet**

```javascript
// Tu frontend recibe el payinLink y redirige
window.location.href = "https://cert.payin.monnetpayments.com/pay?id=abc123def456"
```

#### **Paso 5: Cliente Realiza el Pago**

```
Cliente en Monnet Gateway
    ↓
Selecciona más detalles (si es necesario)
    ↓
Ingresa información de pago
    ↓
Confirma transacción
    ↓
Banco/Procesador procesa pago
```

#### **Paso 6: Cliente es Redirigido**

```
Después del pago:
    ↓
Si éxito → https://tudominio.com/success
Si error → https://tudominio.com/error
```

**Parámetros en URL:**
```
success?payinMerchantOperationNumber=ORDER-001-2024&payinStateID=5
error?payinMerchantOperationNumber=ORDER-001-2024&payinStateID=6
```

#### **Paso 7: Webhook de Confirmación (IMPORTANTE)**

```
Monnet envía POST ASINCRÓNICO a tu webhook:

POST https://tudominio.com/webhook

{
  "payinStateID": "5",
  "payinState": "Autorizado",
  "payinMerchantID": "12345",
  "payinAmount": "50.00",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "ORDER-001-2024",
  "payinMethod": "TCTD",
  "payinVerification": "{hash_sha512}"
}
```

**Tu servidor debe:**
1. ✅ Validar la firma (verificación)
2. ✅ Responder con HTTP 200
3. ✅ Actualizar estado en BD
4. ✅ Realizar acciones (enviar email, activar servicio, etc.)

---

## ⚙️ Códigos de Estado

### Estados Posibles de una Transacción

```
Ciclo de vida de un pago:

1 (Creado)
    ↓
2 (Pendiente de pago)
    ├─→ 3 (Expirado)  ❌
    ├─→ 5 (Autorizado) ✅
    │    ↓
    │   9 (Liquidado)
    │    ├─→ 11 (Devuelto) ↩️
    │    └─→ 10 (Reembolsado) ↩️
    │
    └─→ 6 (Denegado) ❌
```

### Tabla de Estados

| ID | Estado | Descripción | Acción Recomendada |
|----|--------|-------------|-------------------|
| **1** | Creado | Pago acaba de ser creado | Esperar confirmación del cliente |
| **2** | Pendiente de pago | Cliente está en proceso de pago | Esperar acción del cliente |
| **3** | Expirado | Link de pago expiró | Crear nuevo link si es necesario |
| **5** | Autorizado ✅ | Pago completado exitosamente | **Procesar pedido, liberar servicio** |
| **6** | Denegado ❌ | Pago fue rechazado | Informar al cliente, ofrecerr reintento |
| **9** | Liquidado ✅ | Pago fue liquidado en banco | Dinero disponible en cuenta |
| **10** | Reembolsado ↩️ | Pago fue reembolsado (antes de liquidarse) | Notificar cliente, registrar devolución |
| **11** | Devuelto ↩️ | Dinero fue devuelto (después de liquidarse) | Notificar cliente, registrar devolución |

### Flujo de Estados

```
Estado CREADO (1)
↓
Estado PENDIENTE DE PAGO (2)
↓
Tres posibilidades:
├─ Estado EXPIRADO (3) → No hay acción
├─ Estado AUTORIZADO (5) ✅ → Procesar
└─ Estado DENEGADO (6) → Error

Si AUTORIZADO (5):
↓
Estado LIQUIDADO (9)
↓
Dos posibilidades:
├─ Estado REEMBOLSADO (10) → Devolución antes de liquidar
└─ Estado DEVUELTO (11) → Devolución después de liquidar
```

---

## ❌ Códigos de Error

### Errores en Creación de Transacción

```
0000 ✅ Éxito - Link creado exitosamente
0001 ❌ payinMerchantID no válido (vacío)
0002 ❌ payinAmount no válido (vacío)
0003 ❌ payinCurrency no válida (vacía)
0004 ❌ payinMerchantOperationNumber no válido (vacío)
0005 ❌ payinVerification no válida (vacía)
0006 ❌ payinTransactionErrorURL no válida (vacía)
0007 ❌ payinTransactionOKURL no válida (vacía)
0008 ❌ payinProcessorCode no válido
0009 ❌ payinMerchantID incorrecta
0010 ❌ payinVerification incorrecta (firma mal)
0011 ❌ Comerciante no habilitado
0012 ❌ payinTransactionErrorURL no válida
0013 ❌ payinTransactionOKURL no válida
0015 ❌ Formato de payinAmount incorrecto
0017 ❌ payinCurrency no válida
0018 ❌ Procesador no válido
0019 ❌ Moneda no existe para este comerciante
0022 ❌ Tipo de documento de cliente no existe
0023 ❌ Documento de cliente no existe
0025 ❌ Tipo de documento inválido
0026 ❌ Número de documento inválido
0030 ❌ No cumple reglas de pre-autorización (Argentina)
0031 ❌ Procesador - código no registrado
0032 ❌ Procesador - clave no registrada
0040 ❌ CBU requerido (Argentina)
0041 ❌ CUIT requerido (Argentina)
0042 ❌ Error enviando a gateway Yuno
0043 ❌ CBU y CUIT no coinciden
0044 ❌ Suscripción inactiva
0046 ❌ CBU excede longitud máxima
0047 ❌ CUIT excede longitud máxima
0048 ❌ collectorCode inválido o no soportado
0049 ❌ collectorCode no habilitado para tu cuenta
0050 ❌ collectorCode: procesador temporalmente no disponible
0051 ❌ collectorCode: colector no disponible o monto fuera de límites
0099 ❌ Error interno de Payin
```

### Errores en Confirmación de Pago

```
9000 ✅ Éxito
9001 ❌ Referirse al emisor de tarjeta
9002 ❌ Referirse a condiciones especiales del emisor
9003 ❌ Comerciante inválido
9005 ❌ No honrar (rechazado)
9007 ❌ Recoger tarjeta - condición especial
9012 ❌ Transacción inválida
9013 ❌ Monto inválido
9014 ❌ Número de tarjeta no existe
9015 ❌ Emisor o BIN no existe
9017 ❌ Cancelación del cliente
9019 ❌ Reintentar transacción
9020 ❌ Respuesta inválida
9025 ❌ No se puede localizar registro
9030 ❌ Error de formato
9031 ❌ Banco no soportado por switch
9034 ❌ Sospecha de fraude
9039 ❌ Sin cuenta de crédito
9040 ❌ Función solicitada no soportada
9041 ❌ Tarjeta perdida
9043 ❌ Tarjeta robada - recoger
9051 ❌ Fondos insuficientes
9052 ❌ Cuenta corriente no existe
9054 ❌ Tarjeta expirada
9055 ❌ PIN incorrecto
9056 ❌ Sin registro de tarjeta
9057 ❌ Transacción no permitida para titular
9058 ❌ Transacción no permitida para terminal
9061 ❌ Adelanto no permitido
9062 ❌ Tarjeta restringida
9065 ❌ Excedidas transacciones diarias
9066 ❌ Departamento de seguridad - llamar
9068 ❌ Respuesta recibida muy tarde
9075 ❌ Intentos de PIN excedidos
9078 ❌ Tarjeta no activada
9080 ❌ Fecha de expiración de tarjeta inválida
9089 ❌ Información de cliente inválida
9090 ❌ Emisor o switch inoperante
9092 ❌ Institución financiera no encontrada
9093 ❌ Transacción no puede ser completada
9094 ❌ Transmisión duplicada
9096 ❌ Malfunction del sistema
9097 ❌ Timeout de operación
9099 ❌ Error genérico
```

---

## 📋 Documentos de Identificación por País

### Argentina

```
Documento | Acrónimo | Longitud | Ejemplo
────────────────────────────────────────
DNI      | DNI      | 8 dígitos| 12345678
CUIT     | CUIT     | 11 dígitos| 20123456789
CUIL     | CUIL     | 11 dígitos| 27987654321
```

### Chile

```
Documento | Acrónimo | Longitud | Ejemplo
────────────────────────────────────────
RUT      | RUT      | 7-9 dígitos | 12345678-9
Pasaporte| PP       | 9-10 dígitos| 1234567890
```

### Perú

```
Documento | Acrónimo | Longitud | Ejemplo
────────────────────────────────────────
DNI      | DNI      | 8 dígitos | 12345678
Carné Ext| CE       | 8-12 alfanumér| PE123456789
RUC      | RUC      | 9-10 dígitos | 1234567890
Pasaporte| PAS      | 7-12 dígitos | AB123456
```

### Ecuador

```
Documento | Acrónimo | Longitud | Ejemplo
────────────────────────────────────────
Cédula   | CI       | 10 dígitos | 1234567890
Pasaporte| PP       | 13 dígitos (Cash) | 1234567890ABC
RUC      | RUC      | 13 dígitos | 1234567890001
Pasaporte| PAS      | 9-10 dígitos | 12345678 (BankTransfer)
```

### México

```
Documento | Acrónimo | Longitud | Ejemplo
────────────────────────────────────────
CURP     | CURP     | 13-18 dígitos | ABCD123456HMCMRN09
RFC      | RFC      | 10, 12 o 13 dígitos | ABC123456XYZ
```

### Colombia

```
Documento | Acrónimo | Longitud | Ejemplo
────────────────────────────────────────
Cédula   | CC       | 6-10 dígitos | 1234567
NIT      | NIT      | 9 dígitos | 123456789
```

### Guatemala

```
Documento | Acrónimo | Longitud | Ejemplo
────────────────────────────────────────
DPI      | DPI      | 13 dígitos | 1234567890123
```

### Brasil

```
Documento | Acrónimo | Longitud | Ejemplo
────────────────────────────────────────
CPF      | CPF      | 11 dígitos | 12345678901
CNPJ     | CNPJ     | 14 dígitos | 12345678901234
```

---

## 🔐 Verificación y Seguridad

### ¿Por Qué es Importante?

La verificación asegura que:
- ✅ Solo tu servidor puede crear transacciones
- ✅ Nadie puede falsificar una transacción
- ✅ La integridad de los datos está garantizada
- ✅ Monnet sabe que eres realmente tú

### Tipo 1: Verificación Estándar (Standard API)

Se usa para **crear transacciones** y **verificar webhooks**.

#### Fórmula de Firma

```
SHA512(
  payinMerchantID +
  payinMerchantOperationNumber +
  payinAmount +
  payinCurrency +
  KeyMonnet
)
```

#### Implementación en Node.js

```javascript
const crypto = require('crypto');

function generateSignature(merchantId, operationNumber, amount, currency, keyMonnet) {
  const message = `${merchantId}${operationNumber}${amount}${currency}${keyMonnet}`;
  return crypto.createHash('sha512').update(message).digest('hex');
}

// Uso
const signature = generateSignature('12345', 'ORDER-001', '50.00', 'PEN', 'my_secret_key');
console.log(signature); // Output: hash_sha512
```

#### Implementación en PHP

```php
<?php
function generateSignature($merchantId, $operationNumber, $amount, $currency, $keyMonnet) {
    $message = $merchantId . $operationNumber . $amount . $currency . $keyMonnet;
    return hash('sha512', $message);
}

// Uso
$signature = generateSignature('12345', 'ORDER-001', '50.00', 'PEN', 'my_secret_key');
echo $signature; // Output: hash_sha512
?>
```

#### Implementación en Python

```python
import hashlib

def generate_signature(merchant_id, operation_number, amount, currency, key_monnet):
    message = f"{merchant_id}{operation_number}{amount}{currency}{key_monnet}"
    return hashlib.sha512(message.encode()).hexdigest()

# Uso
signature = generate_signature('12345', 'ORDER-001', '50.00', 'PEN', 'my_secret_key')
print(signature)  # Output: hash_sha512
```

#### Implementación en Java

```java
import javax.crypto.Mac;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import org.apache.commons.codec.binary.Hex;

public class SignatureGenerator {
    
    public static String generateSignature(
            String merchantId, 
            String operationNumber, 
            String amount, 
            String currency, 
            String keyMonnet) {
        
        try {
            String message = merchantId + operationNumber + amount + currency + keyMonnet;
            MessageDigest md = MessageDigest.getInstance("SHA-512");
            md.reset();
            md.update(message.getBytes());
            return new String(Hex.encodeHex(md.digest()));
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            return null;
        }
    }
    
    // Uso
    public static void main(String[] args) {
        String signature = generateSignature("12345", "ORDER-001", "50.00", "PEN", "my_secret_key");
        System.out.println(signature); // Output: hash_sha512
    }
}
```

### Tipo 2: Verificación Avanzada (Virtual Account API)

Se usa para **cuentas virtuales** y requiere **HMAC-SHA512**.

#### Fórmula

```
HMAC-SHA512(
  secret_key,
  timestamp + api_key + reference_id
)
```

#### Implementación en Node.js

```javascript
const crypto = require('crypto');

function generateHmacSignature(secret, timestamp, apiKey, referenceId) {
  const message = `${timestamp}${apiKey}${referenceId}`;
  return crypto
    .createHmac('sha512', secret)
    .update(message)
    .digest('hex');
}

// Uso
const timestamp = Math.floor(Date.now() / 1000); // Unix time en segundos
const signature = generateHmacSignature('my_secret', timestamp, 'mk_live_abc123', 'ref-123');
```

#### Implementación en Python

```python
import hmac
import hashlib
import time

def generate_hmac_signature(secret, timestamp, api_key, reference_id):
    message = f"{timestamp}{api_key}{reference_id}"
    return hmac.new(
        secret.encode(),
        message.encode(),
        hashlib.sha512
    ).hexdigest()

# Uso
timestamp = int(time.time())
signature = generate_hmac_signature('my_secret', timestamp, 'mk_live_abc123', 'ref-123')
```

#### Implementación en Java

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

public class HmacUtil {
    
    private static final String HMAC_SHA512 = "HmacSHA512";
    
    public static String generateHmacSignature(
            String secret, 
            long timestamp, 
            String apiKey, 
            String referenceId) {
        
        try {
            String message = timestamp + apiKey + referenceId;
            Mac mac = Mac.getInstance(HMAC_SHA512);
            SecretKeySpec secretKeySpec = new SecretKeySpec(
                secret.getBytes(StandardCharsets.UTF_8), 
                HMAC_SHA512
            );
            mac.init(secretKeySpec);
            
            byte[] rawHmac = mac.doFinal(message.getBytes(StandardCharsets.UTF_8));
            
            StringBuilder hex = new StringBuilder(2 * rawHmac.length);
            for (byte b : rawHmac) {
                hex.append(String.format("%02x", b));
            }
            
            return hex.toString();
        } catch (Exception e) {
            throw new RuntimeException("Error generating HMAC-SHA512", e);
        }
    }
}
```

### Validación de Webhook

Cuando Monnet te envía una notificación, debes verificar que es auténtica:

```javascript
// Tu endpoint webhook
app.post('/webhook', (req, res) => {
  const {
    payinMerchantID,
    payinMerchantOperationNumber,
    payinAmount,
    payinCurrency,
    payinVerification
  } = req.body;
  
  // Recrear la firma
  const expectedSignature = generateSignature(
    payinMerchantID,
    payinMerchantOperationNumber,
    payinAmount,
    payinCurrency,
    process.env.MONNET_KEY
  );
  
  // Comparar
  if (payinVerification === expectedSignature) {
    console.log('✅ Webhook verificado correctamente');
    // Procesar pago
    res.status(200).send('OK');
  } else {
    console.log('❌ Webhook NO verificado');
    res.status(401).send('Unauthorized');
  }
});
```

---

## 🚀 Funcionalidades Avanzadas

### 1. Yape One Shot (Pagos Puntuales con Yape)

#### ¿Qué es?

Yape One Shot permite que clientes en Perú realicen **pagos de una sola vez** usando la aplicación Yape, sin necesidad de guardar consentimiento para pagos futuros.

#### Características

```
✅ One-time: Un pago, un consentimiento
✅ Seguro: Aprobación vía push notification en app Yape
✅ Mobile-first: Optimizado para dispositivos móviles
✅ Deeplink: Redirige a Yape automáticamente en mobile
✅ Real-time: Validación en tiempo real
```

#### Flujo

```
1. Cliente selecciona Yape One Shot
    ↓
2. Merchant envía request de creación
    ↓
3. Monnet valida y crea transacción con Yape
    ↓
4. Se genera link con deeplink para Yape
    ↓
5. Cliente es redirigido a Yape app / manual instructions (web)
    ↓
6. Cliente aprueba en Yape app
    ↓
7. Monnet notifica resultado al merchant
```

#### Dispositivos Soportados

```
MOBILE:
├─ iOS (iOS Safari, Yape App)
└─ Android (Chrome, Yape App)

WEB:
├─ Desktop Chrome/Firefox/Safari
├─ Manual instructions para usuario
└─ Sin redirect automático
```

### 2. Yape On File (Subscripciones)

#### ¿Qué es?

Permite que clientes **autoricen pagos recurrentes** mediante consentimiento único almacenado (One Click Payment - OCP).

#### Tipos de Suscripción

```
ON_DEMAND:
- Merchant inicia cargos cuando quiera
- Cliente debe haber autorizado previamente
- Ideal para: Top-ups, pagos bajo demanda

RECURRENT:
- Pagos automáticos periódicos
- Ej: Membresías, suscripciones mensuales
- Ideal para: Servicios de suscripción
```

#### Flujo

```
1. Cliente autoriza suscripción
    ↓
2. Merchant envía create subscription request
    ↓
3. Monnet solicita autorización a Yape
    ↓
4. Cliente aprueba en app Yape
    ↓
5. Monnet almacena consentimiento tokenizado
    ↓
6. Merchant puede chargar cuando sea necesario
    ↓
7. Webhook notifica cada cargo
```

#### Endpoints

```
POST /create-subscription
├─ Crear nueva suscripción
├─ Requiere autorización previa
└─ Retorna token/reference

POST /cancel-subscription
├─ Cancelar suscripción existente
└─ Cliente deja de ser cargado

GET /webhook
└─ Recibe notificaciones de cambios
```

### 3. Cuentas Virtuales

#### ¿Qué es?

Una **cuenta bancaria virtual única** asignada a cada usuario para recibir transferencias bancarias locales.

#### Ventajas

```
✅ Identificación automática: Cada usuario = cuenta única
✅ Transferencias locales: Usa el sistema bancario del país
✅ Reconciliación automática: Webhook notifica cada depósito
✅ Depósitos rápidos: Acreditación casi inmediata
✅ Sin fricción: Cliente paga desde su banco normal
```

#### Flujo

```
1. Usuario solicita cuenta virtual
    ↓
2. Merchant crea cuenta virtual vía API
    ↓
3. Monnet genera IBAN/cuenta local única
    ↓
4. Merchant entrega detalles al usuario
    ↓
5. Usuario transfiere dinero desde su banco
    ↓
6. Monnet detecta depósito y lo atribuye al usuario
    ↓
7. Webhook notifica al merchant
    ↓
8. Merchant acredita saldo del usuario
```

#### Países Soportados

```
✅ México
✅ Argentina
✅ Perú
```

#### Endpoint disponibles

```
POST /virtual-accounts/create
├─ Crear nueva cuenta virtual
└─ Retorna detalles de cuenta

GET /virtual-accounts/{id}
├─ Obtener detalles de cuenta
└─ Estados, límites, etc.

PUT /virtual-accounts/{id}/information
├─ Actualizar datos del usuario
└─ Nombre, email, etc.

PUT /virtual-accounts/{id}/status
├─ Cambiar estado (activo/inactivo)
└─ Habilitar/deshabilitar cuenta

POST /webhook/deposits
└─ Notificación de depósitos
```

---

## 💻 Especificaciones Técnicas

### Limites y Restricciones

```
┌─────────────────────────────────────────┐
│       LÍMITES TÉCNICOS DE MONNET        │
├─────────────────────────────────────────┤
│ Máx transacciones/seg     │ 10          │
│ Máx tamaño payload        │ No definido │
│ Timeout de request        │ Standard    │
│ Tiempo de expiración link │ ~24 horas   │
│ Decimales en monto        │ 2 decimales │
│ Reintentos de webhook     │ Limitados   │
│ Rate limiting             │ No definido │
└─────────────────────────────────────────┘
```

### Browsers Soportados

#### Monnet Voucher (Payment Gateway)

| Browser | Versión Mínima | Estado |
|---------|----------------|--------|
| 🟦 Safari | 17 o superior | ✅ Soportado |
| 🟩 Chrome | 130 o superior | ✅ Soportado |
| 🟦 Edge | 130 o superior | ✅ Soportado |
| 🔥 Firefox | 133 o superior | ✅ Soportado |

#### Recomendación

```
Para usuarios finales:
✅ Mantener SO actualizado
✅ Mantener browser actualizado
✅ Preferentemente latest stable release
```

### Protocolos y Estándares

```
Protocolos:
✅ HTTPS 1.1 o superior (requerido)
✅ TLS 1.2 o superior
✅ Certificados SSL válidos

Formatos:
✅ JSON (request/response)
✅ UTF-8 (encoding)
✅ ISO-8601 (timestamps)
✅ ISO-4217 (códigos de moneda)

Métodos HTTP:
✅ POST (crear, actualizar)
✅ GET (consultar)
```

### Headers Requeridos

```http
Content-Type: application/json
Accept: application/json
Authorization: {signature_sha256}
X-Merchant-ID: {payinMerchantID}
User-Agent: {TuAplicacion/1.0}
```

---

## ✅ Checklist de Implementación

### Fase 1: Preparación

- [ ] Crear cuenta en Monnet (contactar account manager)
- [ ] Recibir credenciales (MerchantID, KeyMonnet, Signature Key)
- [ ] Acceder a Back Office (CERT)
- [ ] Configurar URL de webhook en Back Office
- [ ] Configurar URLs de success/error
- [ ] Guardar credenciales en variables de entorno (NUNCA en código)
- [ ] Configurar HTTPS en tu servidor (requerido)

### Fase 2: Backend - Desarrollo

- [ ] Crear endpoint para crear transacciones
- [ ] Implementar generación de firma (SHA-512)
- [ ] Crear endpoint para webhook (POST)
- [ ] Implementar validación de webhook (verificación de firma)
- [ ] Crear estructura BD para almacenar transacciones
- [ ] Conectar creación de transacción con tu lógica de negocio
- [ ] Conectar webhook con actualización de pedidos/suscripciones

### Fase 3: Frontend - Desarrollo

- [ ] Crear botón "Pagar con Monnet"
- [ ] Crear selector de método de pago (opcional)
- [ ] Capturar monto y datos del cliente
- [ ] Implementar redirección a Monnet (POST)
- [ ] Crear página de success (con confirmación visual)
- [ ] Crear página de error (con opción de reintento)
- [ ] Validar respuesta HTTP 200 del webhook

### Fase 4: Testing en CERT

- [ ] Probar flujo completo de pago
- [ ] Probar todos los métodos de pago disponibles
- [ ] Probar casos de error
- [ ] Verificar que webhooks llegan correctamente
- [ ] Verificar validación de firmas
- [ ] Probar timeouts y reintentos
- [ ] Validar en múltiples partners (si aplica)
- [ ] Verificar datos en transacciones

### Fase 5: Seguridad

- [ ] Validar HTTPS en todas partes
- [ ] Verificar credenciales stored en variables de entorno
- [ ] Nunca loguear datos sensibles (tarjetas, etc)
- [ ] Implementar CORS correctamente
- [ ] Validar todas las entradas (input validation)
- [ ] Sanitizar datos antes de guardar en BD
- [ ] Usar prepared statements (para prevenir SQL injection)

### Fase 6: Monitoreo y Documentación

- [ ] Configurar logging de transacciones
- [ ] Crear alertas para errores
- [ ] Documentar proceso de integración
- [ ] Crear runbook para troubleshooting
- [ ] Implementar healthchecks para webhook
- [ ] Crear dashboards de monitoreo

### Fase 7: Certificación

- [ ] Contactar a Monnet para proceso de certificación
- [ ] Preparar cuentas de prueba
- [ ] Realizar pagos de prueba con dinero real (bajo recomendación de Monnet)
- [ ] Completar test checklist de Monnet:
  - [ ] Check-out flow
  - [ ] Payment flow
  - [ ] Transaction creation
  - [ ] Notification handling
- [ ] Obtener aprobación de certificación

### Fase 8: Go to Production

- [ ] Solicitar credenciales de PROD
- [ ] Cambiar URLs a `https://payin.api.monnetpayments.com/`
- [ ] Cambiar URL de webhook a PROD
- [ ] Realizar test final en PROD (con monto pequeño)
- [ ] Monitorear primeros pagos
- [ ] Tener plan de rollback listo

---

## 📱 Casos de Uso Comunes

### Caso 1: Tienda E-commerce

```javascript
// Cliente en tu sitio e-commerce
1. Agrega productos al carrito
2. Va a checkout
3. Selecciona "Pagar con Monnet"
4. Selecciona método: Tarjeta, Transferencia, etc.
5. Backend crea transacción en Monnet
6. Redirige a Monnet Payment Gateway
7. Cliente ingresa datos de tarjeta/confirma transfer
8. Pago procesado
9. Webhook notifica éxito
10. Pedido es fulfillado
11. Email de confirmación al cliente

Frontend → Backend → Monnet → Frontend → Backend (webhook) → DB → Fulfillment
```

### Caso 2: Sistema de Suscripciones

```javascript
// Modelo: Yape On File (Subscripción recurrente)

1. Cliente se registra y selecciona plan
2. Backend crea subscription en Yape On File
3. Frontend redirige a autorización Yape
4. Cliente aprueba en app Yape
5. Monnet almacena consentimiento
6. Backend puede chargar automáticamente
7. Cada cargo genera webhook
8. Dashboard muestra suscripcción como activa
9. Cliente puede cancelar desde panel
10. Cancelación genera webhook

Fase 1: Autorización (customer consent)
Fase 2: Cargos recurrentes automáticos
Fase 3: Cancelación o pausas
```

### Caso 3: Plataforma de Wallets (Cuentas Virtuales)

```javascript
// Modelo: Virtual Accounts para deposits

1. Usuario nuevo en tu plataforma
2. Usuario selecciona "Cargar saldo"
3. Backend crea Cuenta Virtual (Peru/Argentina/Mexico)
4. Frontend muestra IBAN/número de cuenta
5. Usuario transfiere desde su banco local
6. Banco detecta depósito en cuenta virtual
7. Monnet recibe notificación del banco
8. Monnet envía webhook a tu backend
9. Backend acredita saldo del usuario
10. Usuario puede usar saldo en plataforma

Completamente automático y sin fricción ✅
```

---

## 🔍 Troubleshooting Común

### Error 0010: Verificación Incorrecta

```
❌ Problema: "Error in payinVerification (it's wrong)"

Causas posibles:
1. Firma SHA-512 mal calculada
2. Orden de parámetros incorrecto
3. KeyMonnet incorrecto o con espacios
4. Parámetros con espacios en blanco
5. Encoding incorrecto (UTF-8 vs otros)

✅ Solución:
- Asegurate de usar EXACTAMENTE: MerchantID + OperationNumber + Amount + Currency + KeyMonnet
- No incluir espacios ni caracteres especiales
- Usar SHA-512, no SHA-256
- Hacer .trim() a todos los valores
- Verificar encoding en UTF-8
```

### Error 0009: MerchantID Inválido

```
❌ Problema: "Error payinMerchantID not valid (it's wrong)"

Causas posibles:
1. MerchantID no coincide con credentials
2. Typo en MerchantID
3. Usando MerchantID de CERT en PROD (o vice versa)
4. MerchantID no activo

✅ Solución:
- Verificar MerchantID en Back Office
- Asegurarse de estar en el environment correcto
- Verificar que la cuenta esté activa en Monnet
```

### Error 0011: Comerciante No Habilitado

```
❌ Problema: "Error in merchant not enabled"

Causas posibles:
1. Cuenta no está activada
2. Cuenta está suspendida
3. Método de pago no está habilitado para la cuenta

✅ Solución:
- Contactar a soporte de Monnet
- Verificar estado de la cuenta en Back Office
- Verificar métodos de pago habilitados
```

### Webhook No Llega

```
❌ Problema: No recibo notificaciones de Monnet

Causas posibles:
1. URL de webhook incorrecto en Back Office
2. Servidor no responde con HTTP 200
3. Firewall bloqueando requests
4. Certificado SSL inválido
5. Server time desincronizado

✅ Solución:
- Verificar URL en Back Office Admin > Merchant Data
- Asegurar respuesta HTTP 200 en endpoint webhook
- Verificar firewall/security groups
- Validar certificados SSL (openssl s_client)
- Verificar logs del servidor
- Usar webhook.site para testing
```

### Transacción Creada pero No Llega Cliente al Pago

```
❌ Problema: Cliente ve error o página blanca en Monnet

Causas posibles:
1. Navegador no soportado (versión muy antigua)
2. Certifciado SSL inválido
3. JavaScript deshabilitado
4. Blockeo por adblock/filtros

✅ Solución:
- Solicitar al cliente abrir con browser moderno (Chrome 130+, Safari 17+, etc)
- Verificar certificado SSL en gateway
- Solicitar deshabilitar adblock
- Probar en incógnito/private mode
```

---

## 📞 Contacto y Recursos

### Soporte Oficial

```
📧 Email: support@monnetpayments.com
🌐 Web: https://www.monnetpayments.com/
📖 Docs: https://payinmonnetpayments.readme.io/

Back Office CERT: https://cert.payin.monnetpayments.com/
Back Office PROD: https://payin.monnetpayments.com/
```

### URLs Importantes

```
Documentación General:
https://payinmonnetpayments.readme.io/

API CERT:
https://cert.payin.api.monnetpayments.com/

API PROD:
https://payin.api.monnetpayments.com/

API Experience (Status, etc):
CERT: https://cert.monnetpayments.com/ms-experience-payin/
PROD: https://apiin.monnetpayments.com/ms-experience-payin/
```

---

## 🎓 Próximos Pasos Recomendados

1. **Obtén Credenciales**
   - Contacta tu account manager
   - Accede a Back Office CERT
   - Descarga API Key y Signature Key

2. **Estudia la Documentación**
   - Lee cada sección cuidadosamente
   - Entiende el flujo de pagos
   - Familiarízate con códigos de estado y error

3. **Implementa Localmente**
   - Clona/crea proyecto base
   - Implementa endpoint de creación
   - Implementa verificación de firmas
   - Implementa webhook listener

4. **Testea en CERT**
   - Prueba flujo completo
   - Prueba todos los métodos de pago
   - Simula errores
   - Verifica logs

5. **Certifícate con Monnet**
   - Contáctales para certificación
   - Completa test checklist
   - Obtén credenciales PROD

6. **Deploy a Producción**
   - Cambia URLs a PROD
   - Monitorea primeras transacciones
   - Mantén alertas activas

---

## 📊 Diagrama de Secuencia

```
┌─────────┐          ┌──────────┐          ┌──────────┐          ┌────────┐
│ Cliente │          │Merchant  │          │  Monnet  │          │ Banco  │
│(Browser)│          │  (Backend)          │  API     │          │ /Card  │
└────┬────┘          └────┬─────┘          └────┬─────┘          └────┬───┘
     │                    │                     │                     │
     │ 1. Selecciona pago  │                    │                     │
     ├───────────────────────>                  │                     │
     │                    │                     │                     │
     │                    │ 2. Create Tx        │                     │
     │                    ├────────────────────>│                     │
     │                    │                     │                     │
     │                    │ 3. Tx Created OK    │                     │
     │                    │<────────────────────┤                     │
     │                    │                     │                     │
     │ 4. Redirect + Link  │                     │                     │
     │<─────────────────────                     │                     │
     │                     │                     │                     │
     │ 5. Open Monnet Gateway                   │                     │
     ╎─────────────────────────────────────────>│                     │
     │                    │                     │  6. Load form        │
     │                    │                     │                     │
     │ 7. Enter Card Data  │                     │                     │
     │<─────────────────────────────────────────│                     │
     │                    │                     │                     │
     │ 8. Submit Form      │                     │                     │
     ├─────────────────────────────────────────>│                     │
     │                    │                     │  9. Process Tx      │
     │                    │                     ├────────────────────>│
     │                    │                     │                     │
     │                    │                     │  10. Tx Response    │
     │                    │                     │<────────────────────┤
     │                    │                     │                     │
     │ 11. Redirect OK     │                     │                     │
     │<──────────────────────────────────────────────────────────────│
     │                    │                     │                     │
     │                    │ 12. Webhook POST (async)                 │
     │                    │<────────────────────┤                     │
     │                    │                     │                     │
     │                    │ 13. HTTP 200 OK     │                     │
     │                    ├────────────────────>│                     │
     │                    │                     │                     │
     │ 14. Success Page    │                     │                     │
     │<──────────────────────                    │                     │
     │                    │                     │                     │
```

---

## 🎯 Resumen Ejecutivo

### Para Ejecutivos/PMs

**¿Qué es?**  
Plataforma de pagos que permite recibir dinero en múltiples países de LatAm con diversos métodos.

**¿Por qué nos interesa?**  
- ✅ Aumentar alcance geográfico
- ✅ Múltiples métodos de pago = más conversión
- ✅ Fácil integración
- ✅ Rápido time-to-market

**¿Cuánto toma implementar?**  
- 2-4 semanas para MVP (integración básica)
- Adicionales para funcionalidades avanzadas (suscripciones, cuentas virtuales)

**¿Cuál es el costo?**  
- Depende de acuerdo comercial
- Típicamente: comisión por transacción exitosa
- Sin costos de integración

**¿Qué riesgos mitiga?**  
- ✅ No requiere infraestructura propia de pagos
- ✅ Cumple regulaciones locales
- ✅ PCI DSS compliance delegado a Monnet
- ✅ Soporte 24/7 disponible

---

## 📝 Notas Importantes

```
⚠️ OBLIGATORIOS:
- HTTPS en todo momento
- Guardar credenciales en variables de entorno
- Validar firmas de webhook
- Responder HTTP 200 a webhooks
- Certificar antes de ir a producción

⚠️ PROHIBIDOS:
- Almacenar números completos de tarjeta
- Enviar datos de tarjeta por email
- Loguear información sensitiva
- Compartir credenciales por chat
- Usar credenciales CERT en PROD

⚠️ BUENAS PRÁCTICAS:
- Usar retry logic en webhooks
- Implementar logging detallado
- Monitorear continuamente
- Tener plan de rollback
- Documentar todo
```

---

**Fin del documento**  
*Última actualización: 20 de Febrero de 2026*
