# Guía Completa de Implementación de Depósitos a Empleados con Monnet

Esta guía consolida la información de los siguientes documentos oficiales:
- [Payout Flow](01-Quick%20Start/01-Payout%20Flow.md)
- [Country Requirements: Peru](02-Country%20Requirements/Peru.md)
- [Create Payout API](03-API%20PAYOUT/01-API%20Reference/01-Create%20Payout.md)
- [Get Balance API](03-API%20PAYOUT/01-API%20Reference/03-Get%20Balance.md)
- [Monnet Error Codes](04-RECURSES/02-Monnet%20error%20codes.md)

## Tabla de Contenidos
1. [Introducción](#introducción)
2. [Requisitos Previos](#requisitos-previos)
3. [Flujo de Depósito](#flujo-de-depósito)
4. [Gestión de Saldo](#gestión-de-saldo)
5. [Configuración Inicial](#configuración-inicial)
6. [API de Creación de Payouts](#api-de-creación-de-payouts)
7. [Parámetros Obligatorios](#parámetros-obligatorios)
8. [Ejemplos de Solicitud](#ejemplos-de-solicitud)
9. [Manejo de Respuestas](#manejo-de-respuestas)
10. [Códigos de Error](#códigos-de-error)
11. [Validaciones Específicas](#validaciones-específicas)
12. [Bancos Soportados](#bancos-soportados)
13. [Notificaciones](#notificaciones)
14. [Buenas Prácticas y Seguridad](#buenas-prácticas-y-seguridad)
15. [Nuevas Funcionalidades](#nuevas-funcionalidades)
16. [Integración en Código](#ejemplo-de-integración-en-código)

## Introducción

Esta guía proporciona una implementación completa y detallada para realizar depósitos a cuentas bancarias de empleados utilizando la API de Monnet Payouts. Está diseñada específicamente para integrarse con sistemas de préstamos y adelantos de sueldo.

Para una visión general del flujo, consulta: [Payout Flow](01-Quick%20Start/01-Payout%20Flow.md).

## Requisitos Previos

> **Nota sobre entornos:** Monnet ofrece entornos de pruebas (Certificación) y producción.
> Fuente: [Environments](01-Quick%20Start/03-Environments.md)

1. **Credenciales de Monnet**:
   - Merchant ID
   - API Key
   - Estas credenciales se obtienen a través de tu account manager

2. **Entorno de Pruebas y Producción**:
   - **Certificación (Pruebas)**: `https://cert.api.payout.monnet.io/api/v1/{merchantId}`
   - **Producción (Live)**: `https://api.payout.monnet.io/api/v1/{merchantId}`

3. **Información del Empleado**:
   - Nombre completo
   - Tipo y número de documento
   - Información bancaria (CCI, número de cuenta, tipo de cuenta)
   - Correo electrónico (opcional pero recomendado)

## Flujo de Depósito

> **Referencia visual:** Diagrama completo del flujo de pagos disponible en [Payout Flow](01-Quick%20Start/01-Payout%20Flow.md).

El proceso de desembolso consta de etapas críticas que aseguran la correcta validación y ejecución del pago:

1. **Validación de Políticas (Backend Fintech)**: Verificar elegibilidad y capacidad de endeudamiento.
2. **Consulta de Saldo (API Monnet)**: Asegurar fondos suficientes antes de procesar.
3. **Solicitud de Desembolso**: FindTech envía la orden de pago a Monnet.
4. **Procesamiento Bancario**: Monnet y el banco receptor procesan la transacción.
5. **Confirmación**: Monnet notifica el resultado final (Éxito/Rechazo).
6. **Conciliación**: FindTech actualiza el estado del préstamo.

### Validación de Política de Crédito (Recomendado)

Antes de invocar la API de desembolso, se recomienda validar las siguientes reglas de negocio para reducir rechazos y riesgos:

- **Antigüedad Laboral**: Verificar tiempo mínimo en la empresa (ej. >3 meses).
- **Capacidad de Endeudamiento**: Asegurar que el descuento total no exceda el porcentaje legal (ej. 60% del sueldo bruto).
- **Cooling-off**: Validar tiempo mínimo desde el último adelanto.
- **Topes por Nivel**: Verificar que el monto esté dentro del rango permitido para el cargo del empleado.

## Gestión de Saldo

Antes de realizar cualquier dispersión, es fundamental verificar que la cuenta del comercio tenga fondos suficientes.
Fuente: [Get Balance API](03-API%20PAYOUT/01-API%20Reference/03-Get%20Balance.md).

### Consultar Saldo Disponible

**Endpoint**: `GET /balance`

```json
// Respuesta Exitosa
[
  {
    "country": "PER",
    "currency": "PEN",
    "availableBalance": 50000.00,
    "balance": 50000.00
  }
]
```

## Configuración Inicial

### Autenticación

Monnet soporta dos mecanismos de autenticación:

#### HMAC (Keyed-hash message authentication codes)
Para operaciones de alta integridad:
- **Crear Payout**: `POST /payouts`
- **Consultar Saldo**: `GET /balance`
- **Consultar Estado**: `GET /payouts/{id}`
- **Create Card Pushpayment**: `POST /api-payout-plus-real-time/{merchantId}`

Incluir headers:
- `monnet-api-key`: API Key del merchant
- `signature`: HMAC SHA-256
- `timestamp`: Timestamp de la solicitud

#### OAuth 2.0 Bearer Token
Para endpoints de validación:
- **Scanning Account Info**: Validar entidades asociadas a celular
- **Get Account Info**: Validar titularidad de cuenta

Obtener token en:
- **Certificación**: `https://cert.authentication.monnet.io/ms-authentication-service/authentication/api-integration/token`
- **Producción**: `https://authentication.monnet.io/ms-authentication-service/authentication/api-integration/token`

### Endpoints Principales

- **Consultar Saldo**: `GET /balance`
- **Crear Payout**: `POST /payouts`
- **Consultar Estado**: `GET /payouts/{id}`
- **Notificaciones**: Configurar URL de callback con Monnet

## API de Creación de Payouts

> **Documentación Oficial:** [Create Payout API](03-API%20PAYOUT/01-API%20Reference/01-Create%20Payout.md)

### Estructura Básica

```json
{
  "country": "PER",
  "amount": 100000,
  "currency": "PEN",
  "orderId": "PRESTAMO-00123",
  "description": "Préstamo DPP - Juan Perez",
  "beneficiary": {
    "name": "Juan",
    "lastName": "Perez",
    "email": "juan.perez@empresa.com",
    "phoneNumber": "999888777",
    "document": {
      "type": 1,
      "number": "12345678"
    },
    "address": {
      "street": "Av. Javier Prado",
      "houseNumber": "123",
      "city": "Lima",
      "country": "PER"
    }
  },
  "destination": {
    "bankAccount": {
      "bankCode": "002",
      "accountType": "1",
      "cci": "00243000584290204477"
    }
  }
}
```

## Parámetros Obligatorios

> **Validaciones de País:** Basado en los requisitos específicos para Perú definidos en [Country Requirements: Peru](02-Country%20Requirements/Peru.md).

### Campos Requeridos

| Campo | Tipo | Descripción | Ejemplo |
|-------|------|-------------|---------|
| country | String(3) | Código ISO 3166-1 alpha-3 | "PER" |
| amount | Integer | Monto en centavos (2 últimos dígitos decimales) | 100000 = S/1,000.00 |
| currency | String(3) | Código ISO 4217 | "PEN" o "USD" |
| orderId | String(20) | ID único de la transacción | "PRESTAMO-00123" |
| beneficiary.name | String | Nombre del empleado | "Juan" |
| beneficiary.lastName | String | Apellido del empleado | "Perez" |
| beneficiary.document.type | Integer | Tipo de documento (ver tabla) | 1 (DNI) |
| beneficiary.document.number | String | Número de documento | "12345678" |
| destination.bankAccount.bankCode | String(3) | Código del banco | "002" |
| destination.bankAccount.accountType | Integer | Tipo de cuenta (ver tabla) | 1 (Cuenta Corriente) |
| destination.bankAccount.cci | String(20) | Código de Cuenta Interbancaria | "00243000584290204477" |

### Tipos de Documento (Perú)

| ID | Tipo | Longitud |
|----|------|----------|
| 1 | DNI | 8 dígitos |
| 2 | PAS | 7-12 dígitos |
| 3 | CE | 8-12 dígitos |
| 4 | RUC | 11 dígitos |

### Tipos de Cuenta

| ID | Tipo |
|----|------|
| 1 | Cuenta Corriente |
| 2 | Cuenta de Ahorros |
| 4 | Número de Celular (billeteras) |
| 5 | Tarjeta (Card Pushpayment - Uso exclusivo de endpoint dedicado) |

## Ejemplos de Solicitud

### Ejemplo 1: Depósito a Cuenta Bancaria

```json
{
  "country": "PER",
  "amount": 500000,
  "currency": "PEN",
  "orderId": "DPP-2024-00456",
  "description": "Préstamo DPP - Maria Lopez",
  "beneficiary": {
    "name": "Maria",
    "lastName": "Lopez",
    "email": "maria.lopez@empresa.com",
    "document": {
      "type": 1,
      "number": "87654321"
    }
  },
  "destination": {
    "bankAccount": {
      "bankCode": "002",
      "accountType": "2",
      "cci": "00219400254640654321"
    }
  }
}
```

### Ejemplo 2: Depósito a Billetera Móvil (Yape)

```json
{
  "country": "PER",
  "amount": 300000,
  "currency": "PEN",
  "orderId": "AS-2024-00789",
  "description": "Adelanto de sueldo - Carlos Ramirez",
  "beneficiary": {
    "name": "Carlos",
    "lastName": "Ramirez",
    "email": "carlos.ramirez@empresa.com",
    "document": {
      "type": 1,
      "number": "11223344"
    }
  },
  "destination": {
    "bankAccount": {
      "bankCode": "037",
      "accountType": "4",
      "accountNumber": "987654321"
    }
  }
}
```

## Manejo de Respuestas

### Respuesta Exitosa

```json
{
  "payout": {
    "Id": 123456789,
    "country": "PER",
    "amount": 500000,
    "currency": "PEN",
    "orderId": "DPP-2024-00456"
  },
  "output": {
    "stage": "PROCESADO",
    "status": "EXITOSO",
    "statusChangeDateTime": "2024-03-15T14:30:00Z"
  }
}
```

### Respuesta de Error

```json
{
  "payout": {
    "Id": 123456789,
    "country": "PER",
    "amount": 500000,
    "currency": "PEN",
    "orderId": "DPP-2024-00456",
    "beneficiary": {
      "name": "Maria",
      "lastName": "Lopez",
      "email": "maria.lopez@empresa.com",
      "document": {
        "type": 1,
        "number": "87654321"
      }
    },
    "destination": {
      "bankAccount": {
        "bankCode": "002",
        "accountType": "2",
        "cci": "00219400254640654321"
      }
    }
  },
  "output": {
    "stage": "RECHAZADO",
    "status": "REJECTED_BANK",
    "statusChangeDateTime": "2024-03-15T14:35:00Z"
  },
  "errors": [
    {
      "code": "4001",
      "message": "Bank Rejection - Beneficiary is not the Account Owner"
    }
  ]
}
```

## Códigos de Error Comunes

> **Referencia Completa:** Lista exhaustiva de códigos en [Monnet Error Codes](04-RECURSES/02-Monnet%20error%20codes.md).

### Errores de Validación Básica (Layer 1)

| Código | Mensaje | Campo | Solución |
|--------|---------|-------|----------|
| 1001 | Missing or invalid Authentication | Authentication | Verificar credenciales |
| 1011 | Missing field [amount] | amount | Incluir campo amount |
| 1013 | Missing field [orderId] | orderId | Incluir orderId único |
| 1014 | Missing field [beneficiary] | beneficiary | Completar información del beneficiario |
| 1017 | Missing field [beneficiary.document] | document | Incluir tipo y número de documento |
| 1020 | Missing field [destination] | destination | Incluir información de destino |
| 1023 | Missing field [destination.bankAccount.bankCode] | bankCode | Incluir código de banco |
| 1025 | Missing field [destination.bankAccount.accountNumber] | accountNumber | Incluir número de cuenta |
| 1026 | Missing field [destination.bankAccount.cci] | cci | Incluir CCI para Perú |

### Errores de Validación de Campos (Layer 1)

| Código | Mensaje | Campo | Solución |
|--------|---------|-------|----------|
| 1103 | [amount] only accepts numeric values | amount | Enviar valor numérico |
| 1105 | [beneficiary.document.type] invalid | document.type | Usar valores válidos (1-4) |
| 1107 | [destination.bankAccount.bankCode] invalid | bankCode | Usar código de banco válido |
| 1108 | [destination.bankAccount.accountType] invalid | accountType | Usar tipo de cuenta válido |
| 1117 | [amount] must be a number | amount | Enviar valor numérico |
| 1118 | [beneficiary.document.type] must be a number | document.type | Enviar valor numérico |

### Errores Específicos de Perú (Layer 1)

| Código | Mensaje | Campo | Solución |
|--------|---------|-------|----------|
| 1201 | [beneficiary.document.number] must have 8 digits for DNI | document.number | Validar 8 dígitos para DNI |
| 1204 | [beneficiary.document.number] must have 11 digits for RUC | document.number | Validar 11 dígitos para RUC |
| 1209 | [destination.bankAccount.cci] must have 20 digits | cci | Validar 20 dígitos para CCI |
| 1210 | [destination.bankAccount.cci] only accepts numeric values | cci | Solo números en CCI |
| 1226 | [destination.bankAccount.accountNumber] must be numeric with 9 digits and start with 9 | accountNumber | Validar formato para billeteras |
| 1227 | [amount] must not exceed 500 PEN for Cellphone number | amount | Máximo S/500.00 para billeteras |
| 1228 | [currency] must be PEN for Cellphone number | currency | Solo PEN para billeteras |

### Errores de Negocio (Layer 2)

| Código | Mensaje | Campo | Solución |
|--------|---------|-------|----------|
| 2001 | Beneficiary is not the Account Owner | beneficiary.document.number | Verificar titularidad de cuenta |
| 2003 | [currency] not allowed for merchantId | currency | Contactar a soporte Monnet |
| 2005 | [orderId] Duplicated | orderId | Usar orderId único |
| 2006 | [beneficiary.document.number] does not correspond to the bank account | document.number | Verificar correspondencia |
| 2008 | CCI duplicated. This CCI account has already been used | bankAccount.cci | Verificar CCI no duplicada |

### Errores de Banco (Layer 4)

| Código | Mensaje | Solución |
|--------|---------|----------|
| 4000 | Bank Rejection | Contactar al banco |
| 4001 | Bank Rejection - Beneficiary is not the Account Owner | Verificar titularidad |
| 4002 | Bank Rejection - Cancelled Credit Account | Cuenta cancelada |
| 4004 | Bank Rejection - Account Number not enabled | Cuenta no habilitada |
| 4012 | Bank Rejection - Amount exceeds the allowance | Monto excede límite |
| 4099 | Unknown Error | Contactar a soporte Monnet |

## Validaciones Específicas

### Para Transferencias Bancarias

1. **CCI Obligatorio**: Para transferencias en tiempo real, el campo `destination.bankAccount.cci` es obligatorio (20 dígitos numéricos)
2. **Formato de Monto**: El amount debe enviarse en centavos (ej: S/1,000.00 = 100000)
3. **orderId Único**: Cada transacción debe tener un orderId único para evitar duplicados
4. **Correo Electrónico**: Recomendado para notificaciones automáticas al empleado

### Para Billeteras Móviles

1. **Solo Moneda PEN**: Para accountType "Cellphone Number" (4), solo se acepta currency "PEN"
2. **bankCode Obligatorio**: Para billeteras, el bankCode es obligatorio (ej: "037" para Yape)
3. **Límite de Monto**: Máximo S/500.00 para billeteras, excepto:
   - **Yape (037)**: Máximo S/3,000.00
   - **Otros Account Types**: Máximo S/30,000.00 PEN
4. **Número de Celular**: Debe tener 9 dígitos y empezar con 9

### Validaciones de Documentos

- **DNI**: Exactly 8 dígitos numéricos
- **PAS**: 7-12 dígitos numéricos
- **CE**: 8-12 dígitos numéricos
- **RUC**: Exactly 11 dígitos numéricos

## Bancos Soportados en Perú

### Entidades Financieras Soportadas

| Código | Entidad | Tipo |
|--------|---------|------|
| 001 | Banco BBVA | Banco |
| 002 | Banco de Crédito | Banco |
| 003 | Interbank | Banco |
| 004 | Scotiabank | Banco |
| 005 | Banco de Comercio | Banco |
| 006 | Banco Interamericano de Finanzas (BanBif) | Banco |
| 007 | Banco Pichincha | Banco |
| 008 | Citibank | Banco |
| 011 | Banco GNB | Banco |
| 014 | Banco Santander | Banco |
| 016 | Banco Cencosud | Banco |
| 017 | ICBC PERU BANK | Banco |
| 018 | Banco de la Nación | Banco |
| 019 | Caja Arequipa | Caja |
| 020 | Caja Cusco | Caja |
| 021 | Caja Huancayo | Caja |
| 022 | Caja Maynas | Caja |
| 023 | Caja Metropolitana | Caja |
| 024 | Caja Municipal Ica | Caja |
| 026 | Caja Sullana | Caja |
| 027 | Caja Tacna | Caja |
| 028 | Caja Trujillo | Caja |
| 029 | Credinka | Caja |
| 030 | Banco Falabella | Banco |
| 031 | Caja Los Andes | Caja |
| 032 | Mibanco | Banco |
| 033 | Bim (Wallet) | Billetera |
| 034 | Caja Piura | Caja |
| 035 | Ripley | Banco |
| 036 | Financiera Efectiva | Financiera |
| 037 | Yape (Wallet) | Billetera |
| 038 | Compartamos Financiera | Financiera |
| 039 | Banco Alfin | Banco |
| 040 | Plin (Wallet) | Billetera |
| 041 | Cooperativa Abaco | Cooperativa |
| 042 | Luqea (Wallet) | Billetera |
| 043 | Tarjetas Peruanas Prepago - Ligo (Wallet) | Billetera |
| 044 | Financiera Confianza | Financiera |
| 045 | Prex (Wallet) | Billetera |
| 046 | Tarjeta Oh | Tarjeta |
| 047 | Crediscotia | Banco |
| 048 | Dale (Wallet) | Billetera |
| 049 | Global66 (Wallet) | Billetera |

### Límites por Tipo de Billetera

| Código | Billetera | Límite Máximo |
|--------|-----------|---------------|
| 037 | Yape | S/3,000.00 |
| 040 | Plin | S/500.00 |
| 033 | Bim | S/500.00 |
| 042 | Luqea | S/500.00 |
| 043 | Ligo | S/500.00 |
| 045 | Prex | S/500.00 |
| 048 | Dale | S/500.00 |
| 049 | Global66 | S/500.00 |

## Notificaciones

> **Guía de Integración:** Detalles completos sobre configuración y seguridad de webhooks en [Notification API](03-API%20PAYOUT/01-API%20Reference/04-Notification.md).

### Configuración de Notificaciones

1. **URL de Callback**: Configurar con tu account manager la URL donde Monnet enviará notificaciones
2. **Formato de Notificación**: JSON con estructura similar a la respuesta de Get Payout Status
3. **Eventos Notificados**:
   - Payout procesado exitosamente
   - Payout rechazado
   - Cambios de estado

### Ejemplo de Notificación

```json
{
  "payout": {
    "Id": 123456789,
    "country": "PER",
    "amount": 500000,
    "currency": "PEN",
    "orderId": "DPP-2024-00456",
    "beneficiary": {
      "name": "Maria",
      "lastName": "Lopez",
      "email": "maria.lopez@empresa.com",
      "document": {
        "type": 1,
        "number": "87654321"
      }
    },
    "destination": {
      "bankAccount": {
        "bankCode": "002",
        "accountType": "2",
        "cci": "00219400254640654321"
      }
    }
  },
  "output": {
    "stage": "COMPLETED",
    "status": "SUCCESS",
    "statusChangeDateTime": "2024-03-15T14:40:00Z"
  }
}
```

## Buenas Prácticas y Seguridad

### Seguridad de la Transacción

1. **Validación de Cuenta**: Asegúrate de que la cuenta bancaria pertenezca al empleado para evitar fraudes y rechazos bancarios (Error 2001/4001).
2. **Idempotencia**: Usa siempre un `orderId` único por transacción. Esto previene duplicidad de pagos ante reintentos por timeout.
3. **Límites de Monto**: Respeta los topes definidos en tu Política de Crédito y los límites bancarios (ej. S/500 para algunas billeteras).

### Manejo de Timeouts y Reintentos

Si recibes un timeout (no respuesta en 30 segundos):
1. Esperar 30 segundos
2. Reenviar la misma solicitud con el mismo `orderId`
3. Monnet verificará si ya fue procesada para evitar duplicados

### Logging y Trazabilidad

1. Registrar todas las solicitudes enviadas a Monnet
2. Registrar todas las respuestas recibidas
3. Implementar un sistema de reconciliación diaria
4. Monitorear códigos de error para mejorar el proceso

### Pruebas Recomendadas

1. **Pruebas Unitarias**: Validar formato de datos antes de enviar
2. **Pruebas de Integración**: Probar con entorno de certificación
3. **Pruebas de Error**: Simular diferentes escenarios de error
4. **Pruebas de Carga**: Validar rendimiento con múltiples solicitudes

## Nuevas Funcionalidades

### Card Pushpayment (Disponible desde Q3 2025)

Nuevo método para transferencias en tiempo real a tarjetas VISA (crédito, débito o prepago) usando OCT (Original Credit Transfer).

**Diferencias con Create Payout:**
- Endpoint específico: `POST /api-payout-plus-real-time/{merchantId}`
- Account Type: `5` (exclusivo para Card)
- Responde con **iframe HTML** que debe embeberse en tu aplicación
- El usuario ingresa datos de tarjeta en el formulario
- Soporta Card on File (habilitado por defecto)

**Parámetros requeridos:**
- `email`: Obligatorio (debe pertenecer al titular de la tarjeta)
- `accountType`: 5
- `timeoutUrl`: URL de redirección en caso de timeout
- `merchantLogo`: (Opcional) URL del logo del merchant (187x40px recomendado)
- `expirationTime`: (Opcional) Duración de sesión en minutos (5-30)

**Ejemplo de solicitud:**

```json
{
  "country": "PER",
  "amount": 50001,
  "currency": "PEN",
  "orderId": "CARD-2024-00789",
  "beneficiary": {
    "name": "Juan",
    "lastName": "Perez",
    "email": "juan.perez@empresa.com",
    "document": {
      "type": 1,
      "number": "12345678"
    }
  },
  "destination": {
    "bankAccount": {
      "accountType": 5
    }
  },
  "timeoutUrl": "https://mitienda.com/timeout",
  "merchantLogo": "https://mitienda.com/logo.png",
  "expirationTime": "10"
}
```

### Scanning Account Info (Solo Perú)

Valida qué entidades están disponibles para recibir pagos en un número de celular.

**Endpoint:**
- Certificación: `https://cert.scanning.payout.monnet.io/ms-payouts-scanning-entities/api/v1/scanning/account-info`
- Producción: `https://scanning.payout.monnet.io/ms-payouts-scanning-entities/api/v1/scanning/account-info`

**Autenticación:** OAuth 2.0 Bearer Token

**Parámetros requeridos:**
- `request_id`: UUID único para trazabilidad
- `account_type`: 4 (Cellphone Number)
- `account_identifier`: Número de celular (9 dígitos, empieza con 9)
- `country`: PER

**Ejemplo:**
```json
{
  "request_id": "fa4f11ce-a997-46fb-b7c6-351975f0a6d4",
  "account_type": "4",
  "account_identifier": "987654321",
  "country": "PER"
}
```

**Respuesta exitosa:**
```json
{
  "traceId": "3caa82a1-c23e-48b8-ab8b-125432863db3",
  "data": {
    "requestId": "fa4f11ce-a997-46fb-b7c6-351975f0a6d4",
    "accountIdentifier": "987654321",
    "entities": [
      { "entityBrand": "Yape", "status": "AVAILABLE" },
      { "entityBrand": "Plin", "status": "AVAILABLE" },
      { "entityBrand": "Banco de Crédito", "status": "AVAILABLE" }
    ],
    "date": "2024-03-21T19:37:29Z"
  }
}
```

**Límite de tasa:** 60 requests por minuto (TPM)

### Get Account Info (Solo Argentina - Disponible para Perú próximamente)

Valida titularidad de cuenta bancaria sin ejecutar un payout.

**Endpoint:**
- Certificación: `https://cert.accountinfo.payout.monnet.io/ms-payouts-account-validations/api/v1/account-validations/account-info`
- Producción: `https://accountinfo.payout.monnnet.io/ms-payouts-account-validations/api/v1/account-validations/account-info`

**Autenticación:** OAuth 2.0 Bearer Token

**Casos de uso:**
- Validar que la cuenta pertenece al empleado
- Mejorar conversión de usuarios
- Cumplimiento regulatorio

## Ejemplo de Integración en Código

### Python Example

```python
import requests
import json

def create_payout(employee_data, amount, order_id):
    url = "https://api.payout.monnet.io/api/v1/{merchantId}/payouts"
    headers = {
        "Merchant-ID": "YOUR_MERCHANT_ID",
        "API-Key": "YOUR_API_KEY",
        "Content-Type": "application/json"
    }
    
    payload = {
        "country": "PER",
        "amount": int(amount * 100),  # Convert to centavos
        "currency": "PEN",
        "orderId": order_id,
        "description": f"Préstamo DPP - {employee_data['name']} {employee_data['lastName']}",
        "beneficiary": {
            "name": employee_data['name'],
            "lastName": employee_data['lastName'],
            "email": employee_data['email'],
            "document": {
                "type": employee_data['document_type'],
                "number": employee_data['document_number']
            }
        },
        "destination": {
            "bankAccount": {
                "bankCode": employee_data['bank_code'],
                "accountType": employee_data['account_type'],
                "cci": employee_data['cci']
            }
        }
    }
    
    try:
        response = requests.post(url, headers=headers, data=json.dumps(payload))
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"Error creating payout: {e}")
        return None

# Ejemplo de uso
employee = {
    "name": "Juan",
    "lastName": "Perez",
    "email": "juan.perez@empresa.com",
    "document_type": 1,  # DNI
    "document_number": "12345678",
    "bank_code": "002",  # Banco de Credito
    "account_type": 1,  # Cuenta Corriente
    "cci": "00243000584290204477"
}

result = create_payout(employee, 1000.00, "DPP-2024-00123")
print(result)
```

### JavaScript Example

```javascript
const axios = require('axios');

async function createPayout(employeeData, amount, orderId) {
    const url = `https://api.payout.monnet.io/api/v1/{merchantId}/payouts`;
    const headers = {
        'Merchant-ID': 'YOUR_MERCHANT_ID',
        'API-Key': 'YOUR_API_KEY',
        'Content-Type': 'application/json'
    };
    
    const payload = {
        country: "PER",
        amount: Math.round(amount * 100), // Convert to centavos
        currency: "PEN",
        orderId: orderId,
        description: `Préstamo DPP - ${employeeData.name} ${employeeData.lastName}`,
        beneficiary: {
            name: employeeData.name,
            lastName: employeeData.lastName,
            email: employeeData.email,
            document: {
                type: employeeData.documentType,
                number: employeeData.documentNumber
            }
        },
        destination: {
            bankAccount: {
                bankCode: employeeData.bankCode,
                accountType: employeeData.accountType,
                cci: employeeData.cci
            }
        }
    };
    
    try {
        const response = await axios.post(url, payload, { headers });
        return response.data;
    } catch (error) {
        console.error('Error creating payout:', error.response?.data || error.message);
        return null;
    }
}

// Ejemplo de uso
const employee = {
    name: "Maria",
    lastName: "Lopez",
    email: "maria.lopez@empresa.com",
    documentType: 1, // DNI
    documentNumber: "87654321",
    bankCode: "002", // Banco de Credito
    accountType: 2, // Cuenta de Ahorros
    cci: "00219400254640654321"
};

createPayout(employee, 1500.50, "DPP-2024-00456")
    .then(result => console.log(result))
    .catch(err => console.error(err));
```

## Soporte y Contacto

Para cualquier problema o consulta:
- **Soporte Técnico**: support@monnet.io
- **Account Manager**: Contactar a tu representante asignado
- **Documentación Completa**: https://docs.monnet.io

## Anexo: Códigos de Error Completos

> **Recurso Original:** [Monnet error codes](04-RECURSES/02-Monnet%20error%20codes.md)

### Errores de Validación Básica (Layer 1)
| Code | Message | Field |
|---|---|---|
| 1001 | Missing or invalid Authentication | Authentication |
| 1010 | Missing field [country] | country |
| 1011 | Missing field [amount] | amount |
| 1012 | Missing field [currency] | currency |
| 1013 | Missing field [orderId] | orderId |
| 1014 | Missing field [beneficiary] | beneficiary |
| 1015 | Missing field [beneficiary.name] | beneficiary(name) |
| 1016 | Missing field [beneficiary.lastName] | beneficiary(lastName) |
| 1017 | Missing field [beneficiary.document] | document |
| 1019 | Missing field [beneficiary.document.type] | document(type) |
| 1020 | Missing field [destination] | document(destination) |
| 1021 | Missing field [destination.bankAccount] | document(bankAccount) |
| 1022 | Missing field [beneficiary.document.number] | document(number) |
| 1023 | Missing field [destination.bankAccount.bankCode] | destination(bankCode) |
| 1024 | Missing field [destination.bankAccount.accountType] | destination(accountType) |
| 1025 | Missing field [destination.bankAccount.accountNumber] | destination(accountNumber) |
| 1026 | Missing field [destination.bankAccount.cci] | destination(cci) |
| 1027 | Missing field [destination.bankAccount.alias] | destination(alias) |
| 1028 | Missing field [destination.bankAccount.cbu] | destination(cbu) |
| 1030 | Missing field [destination.bankAccount.clabe] | clabe |
| 1031 | [Field] exceeds allowed characters | Any |
| 1032 | Missing field | ? |
| 1033 | Missing field [beneficiary.email] | beneficiary(email) |
| 1034 | Wrong format [beneficiary.email] | beneficiary(email) |
| 1035 | The following attribute is not valid: [field]. Validate attribute exist and comply with case sensitive. | Any |

### Errores de Validación de Campos (Layer 1)
| Code | Message | Field |
|---|---|---|
| 1101 | [country] not allowed for merchantId | country |
| 1102 | [country] invalid | country |
| 1103 | [amount] only accepts numeric values | amount |
| 1104 | [currency] invalid | currency |
| 1105 | [beneficiary.document.type] invalid | document(type) |
| 1106 | [beneficiary.document.number] only accepts numeric values | document (number) |
| 1107 | [destination.bankAccount.bankCode] invalid | destination(bankCode) |
| 1108 | [destination.bankAccount.accounType] invalid | destination(accountType) |
| 1109 | [destination.bankAccount.accountNumber] only accepts numeric values | destination(accountNumber) |
| 1110 | [amount] invalid length | amount |
| 1111 | [destination.bankAccount.accountNumber] must be a string | [destination.bankAccount.accountNumber |
| 1112 | [destination.bankAccount.cci] must be a string | destination.bankAccount.cci |
| 1113 | [destination.bankAccount.cbu] must be a string | destination.bankAccount.cbu |
| 1114 | [destination.bankAccount.clabe] must be a string | destination.bankAccount.clabe |
| 1115 | [destination.bankAccount.bankCode] must be a string | destination.bankAccount.bankCode |
| 1116 | [destination.bankAccount.accountType] must be a number | destination.bankAccount.bankCode |
| 1117 | [amount] must be a number | amount |
| 1118 | [beneficiary.document.type] must be a number | document(type) |
| 1121 | The field [Timeout_URL] is mandatory when [destination.bankAccount.accountType] = Card | Timeout_URL |
| 1122 | For [country] = PER and [destination.bankAccount.accountType] = Card, use the Create PushPayment endpoint. | [destination.bankAccount.accountType] |
| 1123 | Merchant does not have credentials for the specified currency. Please contact Monnet support. | currency |

### Errores Específicos de Perú (Layer 1)
| Code | Message | Field |
|---|---|---|
| 1201 | [beneficiary.document.number] must have 8 digits for [beneficiary.document.type] = DNI | document(number) |
| 1202 | [beneficiary.document.number] must have between 7-12 digits for [beneficiary.document.type] = PAS | document(number) |
| 1203 | [beneficiary.document.number] must have between 8-12 digits for [beneficiary.document.type] = CE | document(number) |
| 1204 | [beneficiary.document.number] must have 11 digits for [beneficiary.document.type] = RUC | document(number) |
| 1205 | [destination.bankAccount.accountNumber] must have between 18-20 digits for [destination.bankAccount.bankCode] = 001 (Banco Continental) | destination(accountNumber) |
| 1206 | [destination.bankAccount.accountNumber] must have 13 digits for [destination.bankAccount.accountType] = Current account and [destination.bankAccount.bankCode] = 002 (Banco de Credito) | destination(accountNumber) |
| 1207 | [destination.bankAccount.accountNumber] must have 13 digits for [destination.bankAccount.bankCode] = 003 (Interbank) | destination(accountNumber) |
| 1208 | [destination.bankAccount.accountNumber] must have 10 digits for [destination.bankAccount.bankCode] = 004 (Scotiabank) | destination(accountNumber) |
| 1209 | [destination.bankAccount.cci] must have 20 digits | destination(CCI) |
| 1210 | [destination.bankAccount.cci] only accepts numeric values | destination(CCI) |
| 1213 | [destination.bankAccount.accountNumber] must contain '0' digit in position 11 for [destination.bankAccount.accountType] = Current account, [destination.bankAccount.bankCode] = 002 (Banco de Credito) and [currency] = PEN | [destination.bankAccount.accountNumber] |
| 1214 | [destination.bankAccount.accountNumber] must contain '1' digit in position 11 for [destination.bankAccount.accountType] = Current account,[destination.bankAccount.bankCode] = 002 (Banco de Credito) and [currency] = USD | [destination.bankAccount.accountNumber] |
| 1215 | [destination.bankAccount.cci] field must begin with '011' for [destination.bankAccount.bankCode] = 001 (Banco Continental) and [destination.bankAccount.accountType] = Current account | [destination.bankAccount.cci] |
| 1216 | [destination.bankAccount.cci] field must begin with '002' for [destination.bankAccount.bankCode] = 002 (Banco Continental) | [destination.bankAccount.cci] |
| 1217 | [destination.bankAccount.cci] must contain '00' digits in position 7 and 8 for [destination.bankAccount.bankCode] = 002 (Banco de Credito) and [destination.bankAccount.accountType] = Current account | [destination.bankAccount.cci] |
| 1218 | [destination.bankAccount.cci] must contain '1' digit in position 7 for [destination.bankAccount.bankCode] = 002 (Banco de Credito) and [destination.bankAccount.accountType] = Savings account | [destination.bankAccount.cci] |
| 1219 | [destination.bankAccount.cci] must begin with '003' for [destination.bankAccount.bankCode] = 003 (Interbank) | [destination.bankAccount.cci] |
| 1220 | [destination.bankAccount.cci] must begin with '009' for [destination.bankAccount.bankCode] = 004 (Scotiabank) | [destination.bankAccount.cci] |
| 1221 | [destination.bankAccount.accountNumber] must contain '0' digit in position 12 for [destination.bankAccount.accountType] = Savings account, [destination.bankAccount.bankCode] = 002 (Banco de Credito) and [currency] = PEN | [destination.bankAccount.accountNumber] |
| 1222 | [destination.bankAccount.accountNumber] must contain '1' digit in position 12 for [destination.bankAccount.accountType] = Savings account, [destination.bankAccount.bankCode] = 002 (Banco de Credito) and [currency] = USD | [destination.bankAccount.accountNumber] |
| 1223 | [destination.bankAccount.accountNumber] must have 14 digits for [destination.bankAccount.accountType] = Savings account and [destination.bankAccount.bankCode] = 002 (Banco de Credito) | destination(accountNumber) |
| 1224 | [amount] exceeds the limit allowed for this destination bank | amount |
| 1225 | [destination.bankAccount.cci] first three digits not registered in the banks of Peru | [detail.cciNumber] |
| 1226 | [destination.bankAccount.accountNumber] must be numeric with 9 digits and start with the number “9” when [destination.bankAccount.accountType]= Cellphone Number | [destination.bankAccount.accountNumber] |
| 1227 | [amount] must not exceed 500 PEN for [destination.bankAccount.accountType]= Cellphone number | [amount] |
| 1228 | [currency] must be PEN for [destination.bankAccount.accountType]= Cellphone number | [currency] |
| 1229 | [destination.bankAccount.bankCode] is mandatory when [destination.bankAccount.accountType]= Cellphone Number | [destination.bankAccount.bankCode] |
| 1230 | [destination.bankAccount.cci] must contain '0' digit in position 16 for [destination.bankAccount.accountType] = Current account,[destination.bankAccount.bankCode] = 002 (Banco de Credito) and [currency] = PEN | [destination.bankAccount.cci] |
| 1231 | [destination.bankAccount.cci] must contain '1' digit in position 16 for [destination.bankAccount.accountType] = Current account,[destination.bankAccount.bankCode] = 002 (Banco de Credito) and [currency] = USD | [destination.bankAccount.cci] |
| 1232 | [destination.bankAccount.cci] must contain '0' digit in position 16 for [destination.bankAccount.accountType] = Savings account, [destination.bankAccount.bankCode] = 002 (Banco de Credito) and [currency] = PEN | [destination.bankAccount.cci] |
| 1233 | [destination.bankAccount.cci] must contain '1' digit in position 16 for [destination.bankAccount.accountType] = Savings account, [destination.bankAccount.bankCode] = 002 (Banco de Credito) and [currency] = USD | [destination.bankAccount.cci] |
| 1234 | [amount] must not exceed 40000 for [destination.bankAccount.bankCode] = 123 (Kasnet) | [amount] |
| 1235 | [destination.bankAccount.account.Type] must be Cash for [destination.bankAccount.bankCode] = 123 (Kasnet) | [destination.bankAccount.account.Type] |
| 1236 | [beneficiary.email] is required for accountType = Cash and bankCode = 123 | [beneficiary.email] |
| 1237 | [beneficiary.document.type] is not valid for bankCode = 123 | [beneficiary.document.type] |
| 1238 | [destination.bankAccount.expirationDate] must have this structure: YYYY-MM-DD | [destination.bankAccount.expirationDate] |
| 1239 | [destination.bankAccount.expirationDate] must be at least one calendar day up to 7 days. | [destination.bankAccount.expirationDate] |
| 1240 | [destination.bankAccount.bankCode] must be 123 (Kasnet) for [destination.bankAccount.account.Type] = Cash | [destination.bankAccount.expirationDate] |
| 1241 | [destination.bankAccount.accountType] must be “Cellphone number” when [destination.bankAccount.bankCode] = 033 (BIM), 037 (YAPE) and 040 (PLIN) | [destination.bankAccount.accountType] |
| 1242 | [destination.bankAccount.accountNumber] is mandatory when [destination.bankAccount.accountType]= Cellphone Number | [destination.bankAccount.accountNumber] |
| 1243 | [destination.bankAccount.accountNumber] must have 18 or 20 digits for [destination.bankAccount.bankCode] = 001 (Banco Continental) | [destination.bankAccount.accountNumber] |
| 1244 | [destination.bankAccount.accountNumber] must contain '00' digits in positions 9 and 10 for [destination.bankAccount.bankCode] = 001 (Banco Continental) and [destination.bankAccount.accountNumber] = 20 digits | [destination.bankAccount.accountNumber] |

### Errores de Negocio (Layer 2)
| Code | Message | Field |
|---|---|---|
| 2001 | Beneficiary is not the Account Owner | beneficiary.document.number |
| 2003 | [currency] not allowed for merchantId | currency |
| 2005 | [orderId] Duplicated | orderId |
| 2006 | [beneficiary.document.number] does not correspond to the bank account of the beneficiary | beneficiary.document.number |
| 2007 | [destination.bankAccount.alias] is inactive | destination(alias) |
| 2008 | CCI duplicated. This CCI account has already been used previously with another document.number | destination (bankAccount.cci) |
| 2009 | [destination.bankAccount.alias] could not be validated | destination(alias) |
| 2023 | [amount] has exceed the limit per payout | amount |
| 2024 | [amount] is below the allowed limit | amount |

### Errores de Fraude (Layer 3)
| Code | Message | Field |
|---|---|---|
| 3000 | Fraud Validation - Bank account in Black List | destionation(accountNumber) |
| 3001 | Fraud Validation - CCI account in Black List | destination(cci) |
| 3002 | Fraud Validation - Document Number in Black List | beneficiary(document.number) |
| 3003 | Fraud Validation - Email in Black List | beneficiary(email) |
| 3004 | Fraud Validation - Account number allows to another Customer ID | beneficiary(customerId) |
| 3005 | Fraud Validation - Amount must not exceed 25,000 | amount |
| 3006 | Fraud Validation -Beneficiary cannot receive more than 200,000 soles in a week | document.number |
| 3007 | Fraud validation - operation rejected due to anti-fraud rules | - |
| 3010 | Internal Validation - The transaction could not be processed in accordance with our internal policies | - |

### Errores de Banco (Layer 4)
| Code | Message |
|---|---|
| 4000 | Bank Rejection |
| 4001 | Bank Rejection - Beneficiary is not the Account Owner |
| 4002 | Bank Rejection - Cancelled Credit Account |
| 4003 | Bank Rejection - Blocked Credit Account |
| 4004 | Bank Rejection - Account Number not enabled |
| 4005 | Bank Rejection - Restricted credit account |
| 4006 | Bank Rejection - The document is wrong |
| 4007 | Bank Rejection - The interbank code is wrong |
| 4008 | Bank Rejection - Bank out of service |
| 4009 | Bank Rejection - Credit account is in another currency |
| 4010 | Bank Rejection - Credit account type does not allow transfer |
| 4011 | Bank Rejection - Incorrect account information |
| 4012 | Bank Rejection - Amount exceeds the allowance |
| 4013 | Bank Rejection - The AFP account cannot be linked |
| 4014 | Bank Rejection - Product credit account not enabled |
| 4015 | Bank Rejection - Rejected Requested by customer |
| 4016 | Bank Rejection - Reason to confirm |
| 4017 | Bank Rejection - Information Incomplete |
| 4018 | Bank Rejection - Payment type Incorrect |
| 4019 | Bank Rejection - Incorrect Character |
| 4020 | Bank Rejection - Incorrect Information structure |
| 4021 | Bank Rejection - Rejection by agreement with the beneficiary |
| 4022 | Bank Rejection - Duplicated Order |
| 4023 | Bank Rejection - Bank Code Incorrect |
| 4024 | Bank Rejection - Beneficiary cannot receive payments |
| 4025 | Bank Rejection - Account Number does not exist |
| 4026 | Bank Rejection - Expired payment order |
| 4027 | Bank Rejection - Directory does not exist as available |
| 4028 | Bank Rejection - The destination entity was unable to provide us the account holder details |
| 4030 | Bank Rejection - Card tokenization could not be executed due to communication problems with the provider |
| 4031 | Bank Rejection - The card could not be registered successfully |
| 4032 | Bank Rejection - Rejection referred to the card issuing bank |
| 4033 | Bank Rejection - Invalid amount according to issuing bank |
| 4034 | Bank Rejection - Rejected due to lost card |
| 4035 | Bank Rejection - Rejected due to stolen card |
| 4036 | Bank Rejection - Expired card or expiration date is missing |
| 4037 | Bank Rejection - Rejected due to suspicion of fraud |
| 4038 | Bank Rejection - Restricted card (card invalid in this region or country) |
| 4039 | Bank Rejection - Additional customer authentication required by issuing bank |
| 4040 | Bank Rejection - Card Authentication failed |
| 4041 | We couldn’t verify the account ownership because the beneficiary selected a different bank. |
| 4042 | The beneficiary’s account registration process has expired. A new request is required. |
| 4099 | Unknown Error |

---

**Nota**: Esta documentación está diseñada específicamente para la integración de depósitos a empleados en el sistema FindTech, enfocándose únicamente en la funcionalidad de Monnet para transferencias bancarias y pagos a billeteras móviles.