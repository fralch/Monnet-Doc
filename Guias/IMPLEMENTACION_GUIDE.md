# Monnet Payments Payin API - Guía Completa de Implementación para Full Stack Web Developers

## 🎯 Introducción

Monnet Payments Payin API es una plataforma integral de procesamiento de pagos que permite a las empresas aceptar pagos en toda Latinoamérica a través de múltiples canales. Funciona como intermediario entre los comerciantes y diversos procesadores de pago, ofreciendo una API unificada para manejar diversos métodos de pago incluyendo tarjetas de crédito/débito, transferencias bancarias, pagos en efectivo, billeteras móviles y cuentas virtuales.

## 🚀 Características Principales

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

## 📋 Requisitos Técnicos y Prerrequisitos

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

## 🔗 Pasos de Implementación

### Paso 1: Configuración de Cuenta y Credenciales
1. Contactar a Monnet Payments para crear cuenta de comerciante
2. Acceder al Back Office en `https://cert.payin.monnetpayments.com/`
3. Recuperar API Key y Signature Key desde la sección de perfil
4. Configurar URL webhook en Back Office (Admin > Merchant Data)

### Paso 2: Configuración del Entorno
```javascript
// Ejemplo de configuración
const config = {
  environment: 'CERT', // o 'PROD'
  baseUrl: 'https://cert.payin.api.monnetpayments.com/',
  apiKey: 'your_api_key',
  signatureKey: 'your_signature_key',
  merchantId: 'your_merchant_id',
  webhookUrl: 'https://yourdomain.com/webhook'
};
```

### Paso 3: Implementación del Flujo de Pago

#### 3.1 Flujo Básico de Transacción
1. **Crear Transacción**: El comerciante envía solicitud de pago a Monnet
2. **Pasarela de Pago**: Cliente redirigido a instrucciones de pago
3. **Completar Pago**: Cliente completa el pago
4. **Notificación**: Monnet envía notificación webhook
5. **Verificación de Estado**: Comerciante verifica estado del pago

#### 3.2 Integración Yape
1. **Creación de Suscripción**: Crear suscripción Yape (ON_DEMAND o RECURRENT)
2. **Autorización del Usuario**: Cliente autoriza via app Yape
3. **Ejecución del Pago**: Comerciante inicia pagos usando suscripción
4. **Notificación**: Webhook actualiza eventos de suscripción

#### 3.3 Integración de Cuentas Virtuales
1. **Creación de Cuenta**: Crear cuenta virtual para usuario
2. **Distribución de Cuenta**: Proporcionar detalles de cuenta al usuario
3. **Procesamiento de Depósito**: Usuario realiza transferencia bancaria
4. **Notificación Automatizada**: Webhook notifica sobre depósito

## 📝 Ejemplos de Código y Mejores Prácticas

### 5.1 Ejemplo de Creación de Transacción (Node.js)
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

### 5.2 Verificación de Webhook
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

### 5.3 Creación de Cuenta Virtual
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

### 5.4 Creación de Suscripción Yape
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

### 5.5 Verificación de Estado de Pago
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

## 🔗 Patrones de Integración

### 6.1 Flujo de Pago Síncrono
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

### 6.2 Flujo Asíncrono con Webhooks
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

### 6.3 Patrón de Integración de Cuentas Virtuales
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

### 6.4 Patrón de Suscripción Yape
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

## 🔧 Manejo de Errores y Gestión de Estados

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

## 🎯 Mejores Prácticas

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

## 📊 Flujos de Pago Detallados

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

## 🚀 Implementación en Nuevo Proyecto

### Estructura Recomendada del Proyecto
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

### Flujo de Desarrollo Recomendado
1. **Setup Inicial**: Configurar entorno CERT y obtener credenciales
2. **Implementación Básica**: Crear servicios API fundamentales
3. **Integración Frontend**: Implementar flujo de pago en frontend
4. **Webhooks**: Crear y probar endpoint de webhooks
5. **Testing**: Realizar pruebas exhaustivas en CERT
6. **Certificación**: Probar con Monnet para certificación
7. **Producción**: Migrar a entorno PROD con monitoreo

### Monitoreo y Mantenimiento
- **Logs**: Implementar logging estructurado para todas las operaciones
- **Métricas**: Monitorear tasas de éxito, tiempos de respuesta, errores
- **Alertas**: Configurar alertas para fallos críticos
- **Actualizaciones**: Mantenerse informado sobre cambios en API
- **Cumplimiento**: Revisar regularmente cumplimiento regulatorio

Esta guía completa proporciona todo lo necesario para que un desarrollador full stack web implemente exitosamente Monnet Payments Payin API en sus proyectos, cubriendo todos los aspectos desde configuración inicial hasta despliegue en producción y mantenimiento continuo.