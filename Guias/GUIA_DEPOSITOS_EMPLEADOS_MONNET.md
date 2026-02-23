# 📚 Guía Completa: Depositar Préstamos a Empleados usando Monnet

> **Caso de Uso:** Depositar préstamos a cuentas bancarias de empleados (solo para depositar, no para cobrar)
> **Audiencia:** Desarrolladores y equipos técnicos
> **Versión:** 1.0 - Febrero 2026
> **Idioma:** Español

---

## 📋 Tabla de Contenidos

1. [Introducción](#1-introducción)
2. [Requisitos Previos](#2-requisitos-previos)
3. [Configuración Inicial](#3-configuración-inicial)
4. [Flujo de Depósito a Empleados](#4-flujo-de-depósito-a-empleados)
5. [Implementación Técnica](#5-implementación-técnica)
6. [Ejemplos de Código](#6-ejemplos-de-código)
7. [Manejo de Errores](#7-manejo-de-errores)
8. [Pruebas y Validación](#8-pruebas-y-validación)
9. [Pasos para Producción](#9-pasos-para-producción)
10. [Checklist de Seguridad](#10-checklist-de-seguridad)

---

## 1. Introducción

Esta guía te ayudará a implementar Monnet para **depositar préstamos directamente a las cuentas bancarias de tus empleados**. El enfoque es **solo para depósitos**, no para cobros.

### ¿Por qué Monnet para depósitos a empleados?
- **Seguridad:** Transacciones bancarias seguras y trazables
- **Automatización:** Depósitos masivos con API
- **Trazabilidad:** Registro claro de cada transacción
- **Multi-país:** Soporte para empleados en diferentes países de LATAM

---

## 2. Requisitos Previos

### 2.1 Credenciales de Monnet
Necesitarás:
- `payinMerchantID`: Tu ID de comerciante
- `KeyMonnet`: Clave secreta para firmas SHA512
- `X-Api-Key` y `secret-key`: Para cuentas virtuales (si aplica)

### 2.2 Información de Empleados
Para cada empleado necesitas:
- Nombre completo
- Tipo de documento (DNI, RUC, etc.)
- Número de documento
- Número de cuenta bancaria (CCI, CVU, CLABE según país)
- Banco destino
- Correo electrónico
- Teléfono

### 2.3 Infraestructura Técnica
- Servidor backend con HTTPS
- Base de datos para registrar transacciones
- Endpoint público para webhooks
- Capacidad para generar hashes SHA512

---

## 3. Configuración Inicial

### 3.1 Contactar a Monnet
1. Contacta a tu representante de Monnet
2. Solicita acceso al ambiente CERT (sandbox)
3. Obtén tus credenciales de prueba

### 3.2 Configurar Variables de Entorno

```env
# .env
MONNET_MERCHANT_ID=674
MONNET_KEY=tu_key_monnet_secreta
MONNET_ENV=cert

# URLs
MONNET_API_CERT=https://cert.monnetpayments.com/api-payin/v3/online-payments
MONNET_API_PROD=https://payin.api.monnetpayments.com/api-payin/v3/online-payments

# Para Virtual Accounts (si usas cuentas virtuales)
MONNET_VA_API_KEY=tu_api_key_va
MONNET_VA_SECRET=tu_secret_va

# Tu aplicación
APP_URL=https://tuapp.com
WEBHOOK_URL=https://tuapp.com/api/webhooks/monnet
```

### 3.3 Estructura de Proyecto Recomendada

```
tu-proyecto/
├── backend/
│   ├── services/
│   │   └── monnet.service.js     # Lógica de Monnet
│   ├── controllers/
│   │   └── deposit.controller.js # Controlador de depósitos
│   ├── models/
│   │   └── deposit.model.js      # Modelo de depósito
│   └── config/
│       └── monnet.config.js      # Configuración
├── frontend/
│   └── pages/
│       └── admin/
│           └── deposits.js      # Interfaz de depósitos
└── .env
```

---

## 4. Flujo de Depósito a Empleados

### 4.1 Diagrama del Flujo

```
1. [Admin] Selecciona empleados y monto de préstamo
2. [Backend] Crea transacción con Monnet API
3. [Monnet] Procesa el depósito bancario
4. [Monnet] Envía notificación via webhook
5. [Backend] Actualiza estado del depósito
6. [Admin] Recibe confirmación
```

### 4.2 Pasos Detallados

1. **Selección de Empleados:**
   - Admin selecciona empleados desde la interfaz
   - Sistema valida que los empleados tengan información bancaria completa

2. **Creación de Transacción:**
   - Backend genera el hash SHA512
   - Envía solicitud a Monnet con datos del empleado y monto
   - Monnet devuelve confirmación o error

3. **Procesamiento Bancario:**
   - Monnet procesa la transferencia al banco del empleado
   - Esto puede tomar de minutos a horas según el banco

4. **Notificación:**
   - Monnet envía webhook con el estado del depósito
   - Backend verifica la firma y actualiza la base de datos

5. **Confirmación:**
   - Admin recibe notificación del estado
   - Empleado recibe correo/notificación del depósito

---

## 5. Implementación Técnica

### 5.1 Modelo de Datos de Depósito

```javascript
// models/deposit.model.js
const Deposit = sequelize.define('Deposit', {
  id: { type: Sequelize.INTEGER, primaryKey: true, autoIncrement: true },
  employeeId: { type: Sequelize.INTEGER, allowNull: false },
  amount: { type: Sequelize.DECIMAL(10, 2), allowNull: false },
  currency: { type: Sequelize.STRING(3), allowNull: false },
  status: { type: Sequelize.STRING(20), defaultValue: 'PENDING' },
  monnetTrxId: { type: Sequelize.STRING(50) },
  bankAccount: { type: Sequelize.STRING(50), allowNull: false },
  bankName: { type: Sequelize.STRING(50) },
  reference: { type: Sequelize.STRING(100) },
  createdAt: { type: Sequelize.DATE, defaultValue: Sequelize.NOW },
  updatedAt: { type: Sequelize.DATE, defaultValue: Sequelize.NOW }
});
```

### 5.2 Servicio de Monnet

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

  generateVerification(operationNumber, amount, currency) {
    const raw = `${this.merchantId}${operationNumber}${amount}${currency}${this.keyMonnet}`;
    return crypto.createHash('sha512').update(raw.trim()).digest('hex');
  }

  verifyWebhookSignature(data) {
    const { payinMerchantID, payinMerchantOperationNumber, payinAmount, payinCurrency, payinVerification } = data;
    const expected = crypto
      .createHash('sha512')
      .update(`${payinMerchantID}${payinMerchantOperationNumber}${payinAmount}${payinCurrency}${this.keyMonnet}`)
      .digest('hex');
    return expected === payinVerification;
  }

  async createDeposit(depositData) {
    const verification = this.generateVerification(
      depositData.operationNumber,
      depositData.amount,
      depositData.currency
    );

    const payload = {
      payinMerchantID: this.merchantId,
      payinAmount: depositData.amount,
      payinCurrency: depositData.currency,
      payinMerchantOperationNumber: depositData.operationNumber,
      payinMethod: 'BankTransfer', // Método para depósitos bancarios
      payinVerification: verification,
      payinCustomerName: depositData.employeeName,
      payinCustomerLastName: depositData.employeeLastName,
      payinCustomerEmail: depositData.employeeEmail,
      payinCustomerPhone: depositData.employeePhone,
      payinCustomerTypeDocument: depositData.documentType,
      payinCustomerDocument: depositData.documentNumber,
      payinLanguage: 'ES',
      payinExpirationTime: '1440', // 24 horas para depósitos
      payinDateTime: new Date().toISOString().split('T')[0],
      payinTransactionOKURL: `${process.env.APP_URL}/deposits/success`,
      payinTransactionErrorURL: `${process.env.APP_URL}/deposits/error`,
      payinCustomerAddress: depositData.employeeAddress || 'N/A',
      payinCustomerCity: depositData.employeeCity || 'Lima',
      payinCustomerRegion: depositData.employeeRegion || 'Lima',
      payinCustomerCountry: depositData.employeeCountry || 'Peru',
      payinCustomerZipCode: depositData.employeeZipCode || '15000',
      payinProductID: 'DEP-001',
      payinProductDescription: 'Depósito de préstamo a empleado',
      payinProductAmount: depositData.amount,
      payinProductSku: 'DEP-001',
      payinProductQuantity: '1',
      URLMonnet: this.baseUrl,
      typePost: 'json'
    };

    try {
      const response = await axios.post(this.baseUrl, payload, {
        headers: { 'Content-Type': 'application/json' }
      });
      return response.data;
    } catch (error) {
      console.error('Error creating deposit:', error.response?.data);
      throw error;
    }
  }
}

module.exports = new MonnetService();
```

### 5.3 Controlador de Depósitos

```javascript
// controllers/deposit.controller.js
const monnetService = require('../services/monnet.service');
const Deposit = require('../models/deposit.model');
const Employee = require('../models/employee.model');

// Crear depósito
exports.createDeposit = async (req, res) => {
  try {
    const { employeeId, amount, currency = 'PEN' } = req.body;
    
    // Validar empleado
    const employee = await Employee.findByPk(employeeId);
    if (!employee || !employee.bankAccount) {
      return res.status(400).json({ error: 'Empleado no encontrado o sin cuenta bancaria' });
    }

    // Generar número de operación único
    const operationNumber = `DEP-${Date.now()}-${employeeId}`;

    // Crear registro en BD
    const deposit = await Deposit.create({
      employeeId,
      amount,
      currency,
      status: 'PENDING',
      bankAccount: employee.bankAccount,
      bankName: employee.bankName,
      reference: operationNumber
    });

    // Enviar a Monnet
    const result = await monnetService.createDeposit({
      operationNumber,
      amount: parseFloat(amount).toFixed(2),
      currency,
      employeeName: employee.firstName,
      employeeLastName: employee.lastName,
      employeeEmail: employee.email,
      employeePhone: employee.phone,
      documentType: employee.documentType,
      documentNumber: employee.documentNumber,
      employeeAddress: employee.address,
      employeeCity: employee.city,
      employeeRegion: employee.region,
      employeeCountry: employee.country,
      employeeZipCode: employee.zipCode
    });

    // Actualizar con ID de transacción de Monnet
    if (result.payinErrorCode === '0000') {
      await deposit.update({ 
        monnetTrxId: result.payinTrxOperation,
        status: 'PROCESSING'
      });
      return res.json({ success: true, depositId: deposit.id });
    } else {
      await deposit.update({ status: 'FAILED', error: result.payinErrorMessage });
      return res.status(400).json({ success: false, error: result.payinErrorMessage });
    }

  } catch (error) {
    console.error('Error creating deposit:', error);
    res.status(500).json({ success: false, error: 'Error interno del servidor' });
  }
};

// Webhook de Monnet
exports.handleWebhook = async (req, res) => {
  // Siempre responder 200 primero
  res.status(200).send('OK');

  try {
    const data = req.body;

    // Verificar firma
    if (!monnetService.verifyWebhookSignature(data)) {
      console.warn('⚠️ Webhook con firma inválida:', data.payinMerchantOperationNumber);
      return;
    }

    const { payinStateID, payinMerchantOperationNumber, payinAmount } = data;

    // Buscar depósito
    const deposit = await Deposit.findOne({ where: { reference: payinMerchantOperationNumber } });
    if (!deposit) return;

    // Actualizar según estado
    if (payinStateID === '5') {
      // Depósito AUTORIZADO
      await deposit.update({ 
        status: 'COMPLETED',
        completedAt: new Date()
      });
      
      // Notificar al empleado
      await sendDepositNotification(deposit);
      
    } else if (payinStateID === '6') {
      // Depósito DENEGADO
      await deposit.update({ 
        status: 'FAILED',
        error: data.errorDetails?.messageErrorTrx || 'Depósito denegado'
      });
      
      // Notificar al admin
      await sendAdminAlert(deposit);
    }

  } catch (error) {
    console.error('Error procesando webhook:', error);
  }
};

// Funciones auxiliares
async function sendDepositNotification(deposit) {
  const employee = await Employee.findByPk(deposit.employeeId);
  // Implementar envío de correo/SMS al empleado
  console.log(`Notificación enviada a ${employee.email}: Depósito de ${deposit.amount} ${deposit.currency} completado`);
}

async function sendAdminAlert(deposit) {
  // Implementar notificación al admin
  console.log(`ALERTA: Depósito ${deposit.id} falló para empleado ${deposit.employeeId}`);
}
```

---

## 6. Ejemplos de Código

### 6.1 Ejemplo de Solicitud a Monnet

```json
{
  "payinMerchantID": "674",
  "payinAmount": "1500.00",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "DEP-20260220-001",
  "payinMethod": "BankTransfer",
  "payinVerification": "a3f8b2...hash_sha512...d9c1",
  "payinCustomerName": "Juan",
  "payinCustomerLastName": "Perez",
  "payinCustomerEmail": "juan.perez@empresa.com",
  "payinCustomerPhone": "912345678",
  "payinCustomerTypeDocument": "DNI",
  "payinCustomerDocument": "12345678",
  "payinLanguage": "ES",
  "payinExpirationTime": "1440",
  "payinDateTime": "2026-02-20",
  "payinTransactionOKURL": "https://tuapp.com/deposits/success",
  "payinTransactionErrorURL": "https://tuapp.com/deposits/error",
  "payinCustomerAddress": "Av. Principal 123",
  "payinCustomerCity": "Lima",
  "payinCustomerRegion": "Lima",
  "payinCustomerCountry": "Peru",
  "payinCustomerZipCode": "15036",
  "payinProductID": "DEP-001",
  "payinProductDescription": "Depósito de préstamo a empleado",
  "payinProductAmount": "1500.00",
  "payinProductSku": "DEP-001",
  "payinProductQuantity": "1",
  "URLMonnet": "https://cert.monnetpayments.com/api-payin/v3/online-payments",
  "typePost": "json"
}
```

### 6.2 Ejemplo de Respuesta Exitosa

```json
{
  "url": "https://cert.monnetpayments.com/gateway/pay/xxx",
  "payinErrorCode": "0000",
  "payinErrorMessage": "Successfull process",
  "payinTrxOperation": "MONTRX207249992409275755"
}
```

### 6.3 Ejemplo de Webhook Recibido

```json
{
  "payinStateID": "5",
  "payinState": "Autorizado",
  "payinStatusErrorMessage": "",
  "payinStatusErrorCode": "00",
  "payinMerchantID": "674",
  "payinAmount": "1500.00",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "DEP-20260220-001",
  "payinMethod": "BankTransfer",
  "payinVerification": "hash_sha512_para_verificar",
  "additionalInformation": [],
  "errorDetails": null
}
```

---

## 7. Manejo de Errores

### 7.1 Errores Comunes y Soluciones

| Código | Descripción | Solución |
|---|---|---|
| `0001` | `payinMerchantID` vacío | Verificar credenciales |
| `0002` | `payinAmount` vacío | Validar monto |
| `0005` | `payinVerification` inválido | Verificar generación de hash |
| `0025` | Tipo de documento inválido | Usar `DNI`, `RUC`, etc. |
| `9051` | Fondos insuficientes | Verificar saldo de cuenta origen |
| `9017` | Operación cancelada | Reintentar o contactar soporte |

### 7.2 Validación de Datos

```javascript
function validateDepositData(data) {
  const errors = [];
  
  if (!data.employeeId) errors.push('employeeId requerido');
  if (!data.amount || isNaN(data.amount) || data.amount <= 0) {
    errors.push('amount debe ser un número positivo');
  }
  if (!data.currency || !['PEN', 'USD', 'MXN', 'ARS', 'CLP'].includes(data.currency)) {
    errors.push('currency inválida');
  }
  
  return errors.length > 0 ? errors : null;
}
```

---

## 8. Pruebas y Validación

### 8.1 Checklist de Pruebas en CERT

- [ ] ✅ Crear depósito exitoso
- [ ] ✅ Manejar error de empleado sin cuenta bancaria
- [ ] ✅ Validar generación correcta de hash SHA512
- [ ] ✅ Procesar webhook de depósito exitoso
- [ ] ✅ Procesar webhook de depósito fallido
- [ ] ✅ Verificar firma de webhook inválida
- [ ] ✅ Manejar depósitos duplicados (idempotencia)
- [ ] ✅ Validar montos y divisas
- [ ] ✅ Probar con diferentes tipos de documentos

### 8.2 Pruebas de Integración

```javascript
// Test de creación de depósito
async function testDepositCreation() {
  try {
    const result = await monnetService.createDeposit({
      operationNumber: 'TEST-DEP-001',
      amount: '100.00',
      currency: 'PEN',
      employeeName: 'Test',
      employeeLastName: 'User',
      employeeEmail: 'test@example.com',
      employeePhone: '912345678',
      documentType: 'DNI',
      documentNumber: '12345678'
    });
    
    console.log('Test deposit result:', result);
    return result.payinErrorCode === '0000';
  } catch (error) {
    console.error('Test failed:', error);
    return false;
  }
}
```

---

## 9. Pasos para Producción

### 9.1 Checklist Pre-Producción

- [ ] ✅ Completar pruebas en CERT
- [ ] ✅ Contactar a Monnet para certificación
- [ ] ✅ Obtener credenciales de producción
- [ ] ✅ Configurar webhook en Back Office PROD
- [ ] ✅ Actualizar variables de entorno a PROD
- [ ] ✅ Implementar monitoreo de transacciones
- [ ] ✅ Configurar alertas para errores
- [ ] ✅ Documentar procedimientos de soporte

### 9.2 Migración a Producción

1. **Actualizar .env:**
   ```env
   MONNET_ENV=prod
   MONNET_KEY=nueva_key_produccion
   MONNET_MERCHANT_ID=id_produccion
   ```

2. **Configurar Back Office:**
   - Ingresar a [https://payin.monnetpayments.com/](https://payin.monnetpayments.com/)
   - Configurar URL de webhook: `https://tuapp.com/api/webhooks/monnet`
   - Verificar credenciales

3. **Primeras Transacciones:**
   - Realizar depósitos de prueba con montos pequeños
   - Verificar que los empleados reciban los fondos
   - Monitorear logs y webhooks

---

## 10. Checklist de Seguridad

### 10.1 Seguridad de Credenciales

- [ ] 🔐 `KeyMonnet` en variables de entorno, nunca en código
- [ ] 🔐 No exponer credenciales en frontend
- [ ] 🔐 Usar HTTPS en todas las comunicaciones
- [ ] 🔐 Rotar credenciales periódicamente
- [ ] 🔐 Restringir acceso a variables de entorno

### 10.2 Seguridad de Transacciones

- [ ] 🔐 Verificar hash SHA512 en todos los webhooks
- [ ] 🔐 Implementar idempotencia para evitar duplicados
- [ ] 🔐 Validar montos antes de procesar
- [ ] 🔐 Registrar todas las transacciones en base de datos
- [ ] 🔐 Implementar rate limiting en endpoints públicos

### 10.3 Monitoreo y Alertas

- [ ] 📊 Monitorear tasa de éxito de transacciones
- [ ] 📊 Alertas para errores críticos
- [ ] 📊 Logs detallados de todas las operaciones
- [ ] 📊 Verificación periódica de saldos

---

## 🎯 Conclusión

Con esta guía, estás listo para implementar depósitos de préstamos a empleados usando Monnet. Recuerda:

1. **Siempre prueba en CERT antes de producción**
2. **Valida todos los datos antes de enviar a Monnet**
3. **Monitorea todas las transacciones**
4. **Mantén tus credenciales seguras**
5. **Documenta tus procesos**

Si tienes dudas específicas sobre tu implementación, consulta la documentación oficial de Monnet o contacta a su equipo de soporte.

---

*Guía creada específicamente para depositar préstamos a empleados usando Monnet - Febrero 2026*
