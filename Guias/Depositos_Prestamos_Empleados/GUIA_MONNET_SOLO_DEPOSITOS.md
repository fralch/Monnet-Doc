# Guia Completa Monnet: Datos y Requisitos para Depositos a Cuentas (Solo Documentacion Monnet)

> Fuente base: `Guias/GUIA_COMPLETA_MONNET.md` + seccion `02-API-Payin/06_virtual_accounts/*`
> Alcance: Solo requisitos tecnicos que pide Monnet para depositos en cuentas virtuales (VA).
> Fecha: 2026-02-23

## 1. Alcance real segun la documentacion de Monnet

En la documentacion disponible de este repositorio, Monnet describe dos bloques:

1. `Payin` (cobros online por checkout, redireccion).
2. `Virtual Accounts (VA)` para recibir depositos/transferencias en una cuenta virtual creada por el comercio.

Para el caso de "depositos a cuentas", la seccion aplicable y detallada es **Virtual Accounts**.

## 2. Flujo oficial de deposito con Virtual Accounts (VA)

1. Tu sistema crea una Cuenta Virtual en Monnet para una persona/empresa.
2. Monnet devuelve `account.number` (numero de cuenta virtual) e `id`.
3. El depositante transfiere dinero a esa cuenta virtual.
4. Monnet envia webhook de deposito a tu endpoint con estado final (`5` autorizado o `6` denegado).

## 3. Endpoint para crear la cuenta virtual

- Metodo: `POST`
- Endpoint: `{{base_url}}/merchant-payin-accounts/v1/accounts`
- `base_url`: se solicita al equipo de Integraciones de Monnet por ambiente.

## 4. Headers obligatorios (creacion VA)

Debes enviar siempre:

- `X-Api-Key`: identificador publico del comercio.
- `X-Timestamp`: Unix timestamp (segundos), ventana valida <= 2 minutos.
- `X-Signature`: `HMAC_SHA512(secret-key, timestamp + api-key)`.
- `X-Account-deposit-mode`:
  - `OWNER`: solo el titular de la VA puede depositar.
  - `ANY`: cualquier persona puede depositar.

## 5. Firma requerida por Monnet (VA)

Regla:

```text
X-Signature = HMAC_SHA512(secret-key, X-Timestamp + X-Api-Key)
```

- `secret-key` la entrega Monnet.
- No enviar `secret-key` en request.
- Generar firma en backend, nunca frontend.

## 6. Body obligatorio para crear VA

### 6.1 owner

- `owner.referenceId` (requerido): id interno de tu usuario. Min 1, max 36.
- `owner.type` (requerido): `PERSON` o `COMPANY`.
- `owner.document.type` (requerido):
  - ARG: `CUIT`
  - MEX: `RFC`
  - PER: `DNI` o `RUC`
- `owner.document.number` (requerido): solo digitos, sin guiones ni espacios.
- `owner.firstName` (condicional): requerido para `PERSON`. Max 50.
- `owner.lastName` (condicional): requerido para `PERSON`. Max 50.
- `owner.companyName` (condicional): requerido para `COMPANY`. Max 80.
- `owner.email` (requerido): email valido. Max 50.
- `owner.phone.countryCode` (requerido): codigo pais numerico sin `+`. Max 4.
- `owner.phone.number` (requerido): telefono nacional numerico. Max 14.

### 6.2 account

- `account.category` (requerido): `VIRTUAL`.
- `account.type` (requerido):
  - ARG: `CVU`
  - MEX: `CLABE`
  - PER: `CCI`
- `account.country` (requerido): ISO 3166-1 alpha-3 (`ARG`, `MEX`, `PER`).
- `account.currency` (requerido): ISO 4217 (`ARS`, `MXN`, `PEN`, etc.).
- `account.name` (requerido): alias/etiqueta de cuenta.
  - Longitud general: 3 a 20.
  - Regla especial MEX: exactamente dos palabras alfabeticas, sin numeros.

### 6.3 metadata (opcional)

- Hasta 5 pares clave-valor.
- `key`: max 20, alfanumerico + `-` `_`.
- `value`: max 100, permite letras, numeros, espacios y `- _ . : , / @`.

## 7. Ejemplo completo de request (creacion VA)

```http
POST {{base_url}}/merchant-payin-accounts/v1/accounts
X-Api-Key: {{api_key}}
X-Timestamp: {{unix_ts}}
X-Signature: {{hmac_sha512}}
X-Account-deposit-mode: OWNER
Content-Type: application/json
```

```json
{
  "owner": {
    "referenceId": "EMP-1024",
    "type": "PERSON",
    "document": {
      "type": "DNI",
      "number": "12345678"
    },
    "firstName": "Juan",
    "lastName": "Perez",
    "companyName": null,
    "email": "juan.perez@empresa.com",
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
    "external_ref": "EMP-1024",
    "segment": "PAYROLL"
  }
}
```

## 8. Respuesta exitosa de creacion (campos a guardar)

Monnet devuelve, entre otros:

- `id`: identificador interno de la VA.
- `traceId`: id de trazabilidad.
- `timestamp`: fecha/hora de proceso.
- `status`: estado de cuenta (`ACTIVE`, etc.).
- `account.number`: numero de cuenta virtual a donde se deposita.
- `resultUrl` (opcional).

Campos criticos para persistir:

- `id`
- `account.number`
- `owner.referenceId`
- `status`
- `traceId`

## 9. Como Monnet notifica un deposito (webhook)

Cuando entra un deposito a la VA, Monnet envia `POST` a tu URL registrada.

Tu endpoint debe:

- Ser publico por HTTPS.
- Aceptar JSON.
- Responder HTTP 200 inmediatamente.
- Procesar logica en segundo plano (asincrono).

### 9.1 Payload de webhook (deposito VA)

Campos principales:

- `operationId` (requerido): id de operacion en Monnet.
- `status.code` (requerido): para VA solo `5` (Autorizado) o `6` (Denegado).
- `status.description` (requerido): descripcion del estado.
- `merchantId` (requerido): comercio en Monnet.
- `amount` (requerido): monto con 2 decimales.
- `currency` (requerido): moneda ISO 4217.
- `payinMethod` (requerido): para VA siempre `VA`.
- `depositDetails.account.id` (requerido): coincide con `id` de la VA.
- `depositDetails.account.number` (requerido): coincide con `account.number`.
- `depositDetails.owner.referenceId` (requerido): tu id interno enviado en creacion.
- `depositDetails.depositor.*`: datos del depositante (si aplica).

### 9.2 Ejemplo webhook

```json
{
  "version": "1.0",
  "operationId": "7019283",
  "status": {
    "code": "5",
    "description": "Aprobado"
  },
  "merchantId": "722",
  "amount": "200.00",
  "currency": "ARS",
  "payinMethod": "VA",
  "depositDetails": {
    "account": {
      "id": "acc_8af98b8c8a4",
      "category": "VIRTUAL",
      "type": "CVU",
      "number": "200300310012",
      "name": "john.doe"
    },
    "owner": {
      "fullName": "Joe Doe",
      "documentType": "CUIT",
      "documentNumber": "20123456783",
      "referenceId": "COMPANY-USER-1234"
    },
    "depositor": {
      "fullName": "Miles Morales",
      "documentType": "CUIT",
      "documentNumber": "32216549878"
    }
  },
  "errors": [],
  "receivedAt": "2025-02-21T14:22:05Z"
}
```

## 10. Endpoint para consultar cuenta virtual

- Metodo: `GET`
- Endpoint: `{{base_url}}/merchant-payin-accounts/v1/accounts?`
- Headers: `X-Api-Key`, `X-Timestamp`, `X-Signature`.
- Query params opcionales:
  - `accountId`
  - `accountNumber`
  - `documentType` (+ `documentNumber` requerido si envias `documentType`)

## 11. Endpoint para actualizar informacion de VA

- Metodo: `PATCH`
- Endpoint: `{{base_url}}/merchant-payin-accounts/v1/accounts/{{accountid}}`
- Headers: `X-Api-Key`, `X-Timestamp`, `X-Signature`.
- Campos editables:
  - `owner.email` (opcional)
  - `owner.phone.countryCode` (opcional)
  - `owner.phone.number` (opcional)
  - `account.name` (opcional)

## 12. Endpoint para actualizar estado de VA

- Metodo: `PATCH`
- Endpoint: `{{base_url}}/merchant-payin-accounts/v1/accounts/{{accountid}}/status`
- Headers: `X-Api-Key`, `X-Timestamp`, `X-Signature`.
- Body:
  - `status`: `ACTIVE`, `INACTIVE`, `DELETED`
  - `reason`: motivo corto

Notas Monnet:

- En ARG no aplica `INACTIVE`.
- `DELETED` es irreversible y deshabilita recepcion de transferencias.

## 13. Validaciones y formatos clave que pide Monnet

- Timestamp valido dentro de ventana <= 2 minutos.
- Firma HMAC-SHA512 correcta.
- Documento por pais correcto (`CUIT`, `RFC`, `DNI`/`RUC`).
- `account.type` acorde al pais (`CVU`/`CLABE`/`CCI`).
- `document.number` solo digitos.
- `account.name` con reglas por pais (especialmente MEX).

## 14. Errores (estructura)

Monnet responde errores con:

- `error.type`: `validation_error`, `authentication_error`, `conflict_error`, `provider_error`, `internal_error`.
- `error.summary`
- `error.details[]` con `code`, `message`, `field`.
- `error.timestamp`
- `error.traceId`

## 15. Lista de datos minimos que debes enviar a Monnet para habilitar depositos via VA

Request de creacion VA (minimo practico):

- Headers: `X-Api-Key`, `X-Timestamp`, `X-Signature`, `X-Account-deposit-mode`.
- Body:
  - `owner.referenceId`
  - `owner.type`
  - `owner.document.type`
  - `owner.document.number`
  - `owner.firstName`/`owner.lastName` (si PERSON)
  - `owner.companyName` (si COMPANY)
  - `owner.email`
  - `owner.phone.countryCode`
  - `owner.phone.number`
  - `account.category` = `VIRTUAL`
  - `account.type` (CVU/CLABE/CCI)
  - `account.country`
  - `account.currency`
  - `account.name`

## 16. Aclaracion importante segun la documentacion disponible

Esta documentacion describe **depositos entrantes** a cuentas virtuales de Monnet (VA).
No muestra, en estos archivos, un endpoint de **transferencia saliente** para que el comercio "envie" dinero directamente a una cuenta bancaria externa.

Si necesitas ese flujo de salida, debes pedir a Monnet su API de `Payout/Disbursement` y su documentacion especifica.
