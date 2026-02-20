# Payment Notification

Once Monnet receives a notification from the bank that a transaction has been completed (paid by the shopper), Monnet will notify the Merchant POST URL.

The merchant must provide Monnet with a public URL (HTTPS) page that accepts parameters using HTTP POST standards. This page can be coded in any language but should respond using a standard format.

### 📦 Request Payload

Monnet will POST the following information to the URL:

| Field | Description |
| :--- | :--- |
| payinStateID | **Integer** <br> Payin State ID. [Status](https://payinmonnetpayments.readme.io/reference/status-codes) |
| payinState | **String** <br> Payin Status description. [Status](https://payinmonnetpayments.readme.io/reference/status-codes) |
| payinStatusErrorCode | **Integer** <br> Field reserved for future use. Will not contain data in this notification and should not be evaluated. |
| payinStatusErrorMessage | **String** <br> Field reserved for future use. Will not contain data in this notification and should not be evaluated |
| payinMerchantID | **Integer** <br> Merchant Identifier for the Payin API. |
| payinAmount | **Decimal** <br> The amount of the transaction. Use 2 decimals. |
| payinCurrency | **String (ISO-4217)** <br> The currency of the transaction <br> **Example** <br> USD, EUR, MXN, etc. |
| payinMerchantOperationNumber | **String** <br> Reference number of the sale. <br> This value is used to notify Merchants about a payment. |
| payinMethod | **String** <br> Payment Method |
| payinVerification | **String (Required)** <br> HASH SHA512 of: <br> payinMerchantID+ <br> payinMerchantOperationNumber+ <br> payinAmount+ <br> payinCurrency+ <br> KeyMonnet |
| additionalInformation | **Array (Optional)** <br> Contains supplementary data from the processor, such as additional context or transaction metadata. |
| errorDetails | **Object (Optional)** <br> Contains the homologated error code and description, based on the error returned by the processor. |
| codeErrorTrx | **String** <br> Shows a homologated error code if payment confirmation fails. <br> [Error Codes](https://payinmonnetpayments.readme.io/reference/error-codes) |
| messageErrorTrx | **String** <br> Shows the homologated error description if payment confirmation fails <br> [Error Codes](https://payinmonnetpayments.readme.io/reference/error-codes) |

### ✅ Successful Response – 200

Monnet expects an **HTTP 200** response to confirm successful notification delivery.

### 📤 Request Standard Payload

```json
{
  "payinStateID": "5",
  "payinState": "Autorizado",
  "payinStatusErrorMessage": "",
  "payinStatusErrorCode": "00",
  "payinMerchantID": "674",
  "payinAmount": "30.0",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "2-2657443175",
  "payinMethod": "Cash",
  "payinVerification": ""
}
```

### 📤 Request Complete Payload

```json
{
  "payinStateID": "6",
  "payinState": "Denegado",
  "payinStatusErrorMessage": "",
  "payinStatusErrorCode": "00",
  "payinMerchantID": "722",
  "payinAmount": "200000.0",
  "payinCurrency": "CLP",
  "payinMerchantOperationNumber": "2874601",
  "payinMethod": "TCTD",
  "payinVerification": "",
  "additionalInformation": [],
  "errorDetails": 
   {
    "codeErrorTrx":"9012",
    "messageErrorTrx":"Invalid transaction"
   }
}
```
