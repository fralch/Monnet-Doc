# Guía Completa de Integración: Monnet Payments Payin API (Fintech)

## 📌 1. ¿Qué es esta documentación y para qué sirve?

Esta documentación corresponde a la **API "Payin" de Monnet Payments**, una pasarela e intermediario de pagos online enfocada en el mercado de Latinoamérica. 

**¿Para qué sirve?**
Sirve para que tu nuevo proyecto Fintech pueda recaudar dinero (Payin) ofreciendo a los usuarios múltiples métodos de pago locales en varios países (Argentina, Chile, Ecuador, Perú, México y Colombia), todo a través de una **única API**.

Los canales físicos y digitales soportados son:
- Tarjetas locales (débito y crédito).
- Transferencias bancarias.
- Pagos móviles (ej. Yape en Perú).
- Depósitos en efectivo a través de redes de recaudación.

Como **Full Stack Web Developer**, la interacción con esta plataforma ocurre principalmente de forma asíncrona ("Server a Server" o API a API) mediante peticiones HTTP.

---

## 📋 2. Elementos Necesarios Previos a la Implementación

Antes de tirar código en tu proyecto Fintech, necesitas asegurarte de tener lo siguiente:

1. **Acceso al Back Office de Prueba (CERT):** 
   - Una cuenta proporcionada por el equipo de Monnet para acceder a `https://cert.payin.monnetpayments.com/`.
2. **Credenciales de Integración:** 
   - Obtenidas desde la sección de Perfil en el Back Office:
     - **Merchant ID** (`payinMerchantID`): Identificador único de tu Fintech.
     - **Signature Key** (`KeyMonnet` o llave secreta): Clave privada fundamental para generar los hashes de firma y validar transacciones.
3. **Punto de Enlace Público (Webhook URL):** 
   - Necesitas un servidor expuesto a Internet (HTTPS) capaz de recibir peticiones HTTP POST, por ejemplo: `https://api.tufintech.com/webhooks/monnet`.
4. **Stack Tecnológico Configurado:** 
   - Frontend (React, Vue, Angular) y Backend (Node.js/Express, Python/Django, PHP/Laravel, etc.) capaz de realizar peticiones HTTPS y rutear respuestas, generar cifrados criptográficos (SHA256 y SHA512).

---

## 🚀 3. Flujo Arquitectónico en tu App Full Stack

El flujo típico de compra o depósito ("Payin") dentro de tu aplicación sería:

1. **UX/UI (Frontend):** El usuario en tu plataforma de Fintech decide depositar fondos en su cuenta de la app o comprar un producto y selecciona "Depositar".
2. **Creación (Backend -> Monnet):** Tu Backend genera un identificador de transacción y hace una solicitud *POST* a la API de Monnet. 
3. **Redirección (Frontend -> Monnet):** Tu backend devuelve una URL de pago o tu Frontend redirige al usuario a la vista de Checkout de Monnet. El usuario paga en esa pantalla externa.
4. **Notificación (Monnet -> Backend):** El usuario paga y Monnet de forma imperceptible dispara una alerta (Webhook) *POST* a tu servidor para notificar el pago. Tu servidor verifica la firma SHA512, valida, y procesa el estatus, respondiendo con un **HTTP 200**.
5. **Comprobación (Backend -> Monnet):** (Opcional o en cron jobs / fallbacks) Tu Backend puede consultar el estado puntual mediante el endpoint de estatus (requiere firma SHA256).

---

## 🛠️ 4. Pasos Necesarios para la Implementación Técnica

A continuación, los pasos para estructurar las piezas en tu nuevo proyecto.

### Paso 1: Configuración de Variables de Entorno (Backend)

Nunca expongas tus llaves en el código. En tu archivo `.env`:

```env
MONNET_ENVIRONMENT=https://cert.payin.api.monnetpayments.com
MONNET_MERCHANT_ID=tu_merchant_id_de_pruebas
MONNET_SIGNATURE_KEY=tu_signature_key_de_pruebas
MONNET_WEBHOOK_URL=https://api.tu-fintech.com/webhooks/monnet
```

### Paso 2: Creación de Transacción (Checkout)

Tu backend debe crear un endpoint (`POST /api/deposit/create`) invocado por tu aplicación Frontend. Este endpoint se comunica con Monnet para gestionar la orden de pago.

* **Endpoint Monnet (CERT):** `https://cert.payin.api.monnetpayments.com/api-payin/v3/online-payments` (o el equivalente según el método de integración para generar un token / checkout URL).
* Este endpoint de Monnet debe retornar las indicaciones o la URL de redirección a tu Backend, para que el Frontend envíe al usuario.

### Paso 3: Recepción de Notificaciones de Pago (Webhooks) - ¡CRÍTICO!

Es el núcleo donde recibirás el estado de la transacción (Aprobado, Rechazado, etc.). Monnet enviará un payload mediante **POST** a tu URL configurada.

Crea un endpoint en tu backend (`POST /api/webhooks/monnet`).

**Reglas de seguridad obligatorias:**
Monnet te enviará el campo `payinVerification`. Debes validar que esta notificación es *realmente* de Monnet generando tu propio hash y comparándolo.

**Cálculo de la firma en el Webhook (Criptografía):**
El hash es un **SHA512** de la siguiente concatenación exacta: 
`payinMerchantID` + `payinMerchantOperationNumber` + `payinAmount` + `payinCurrency` + `KeyMonnet`

```javascript
// Ejemplo conceptual en Node.js de validación
const crypto = require('crypto');

app.post('/api/webhooks/monnet', (req, res) => {
    const data = req.body;
    
    // Concatenacion estricta (Ojo con los decimales en payinAmount)
    const rawString = `${data.payinMerchantID}${data.payinMerchantOperationNumber}${data.payinAmount}${data.payinCurrency}${process.env.MONNET_SIGNATURE_KEY}`;
    
    // Generar HASH
    const calculatedHash = crypto.createHash('sha512').update(rawString).digest('hex');
    
    // Validar autenticidad
    if (calculatedHash.toLowerCase() !== data.payinVerification.toLowerCase()) {
        console.error("Firma inválida. Posible fraude o ataque.");
        return res.status(401).send("Unauthorized");
    }
    
    // Validar status de la operacion (data.payinStateID == 5 es 'Autorizado')
    if(data.payinStateID === "5") {
        // Actualizar saldo del usuario en la Base de Datos
        // e.g. updateBalance(data.payinMerchantOperationNumber)
    }

    // Monnet exige una respuesta HTTP 200 rápida.
    res.status(200).send("OK");
});
```

### Paso 4: Consulta de Estado de Transacción (Fallback)

Para operaciones donde el usuario dice que "ya pagó" pero el Webhook falló por temas de red, implementa una tarea periódica o un endpoint en el Admin Dashboard de tu Fintech.

* **Endpoint (CERT):** `POST https://cert.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations`
* **Autenticación (Header `authorization`):** Requiere un hash **SHA256** derivado de `MerchantID` + `KeyMonnet`. *(Ojo, ¡es distinto al signature del webhook que es SHA512!)*

```javascript
// Ejemplo conceptual en Node.js para Header Auth
const authString = `${process.env.MONNET_MERCHANT_ID}${process.env.MONNET_SIGNATURE_KEY}`;
const authHeader = crypto.createHash('sha256').update(authString).digest('hex');

// Petición Axios o Fetch a Monnet
const response = await axios.post(`https://cert.monnetpayments.com/ms-experience-payin/merchant/${process.env.MONNET_MERCHANT_ID}/operations`, {
    "payinMerchantOperationNumber": "ID_INTERNO_TRANSACCION" 
}, {
    headers: {
        'authorization': authHeader
    }
});
// Leer estado y actualizar DB.
```

### Paso 5: Certificación y Pase a Producción (PROD)
Monnet exige un chequeo final. Deberás agendar una cita de **Certificación**. 
Ellos probarán tu ambiente realizando transacciones de flujo completo. Al superarlo:
1. Te emitirán credenciales PROD.
2. Cambiarás la URL de `https://cert.payin.*` a `https://payin.*`.
3. Podrás empezar a recaudar dinero real en tu proyecto Fintech.

---
Elaborado con todo el rigor necesario para garantizar la seguridad financiera de los fondos en tu Fintech.
