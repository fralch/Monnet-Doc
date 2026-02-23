# Peru

The purpose of this guide is to explain how to create a transaction in Peru step by step. Monnet has available 3 Payment methods in Peru: Cards (Tarjetas), Online (Banca por internet) and Cash (Pagar en efectivo).

Currently, the Monnet Payin API has 1 version:

* For V3 a URL will be returned on the response, the merchant has to redirect the shopper to this URL

> **📘 URL**
>
> API v3 (for JSON format with an url as response):
>
> Certification: <https://cert.monnetpayments.com/api-payin/v3/online-payments>
>
> Production: <https://payin.api.monnetpayments.com/api-payin/v3/online-payments>

# V3: Create a Transaction

To create an online payment using Monnet Payments API, you must call the method **online-payments** as shown in the following example:

> **🚧 IMPORTANT REMINDER:**
>
> **All fields must be included** on request no matter if they are required or optional. Also pay attention on  **payinCustomerName** (fist name), **payinCustomerLastName** , **payinCustomerEmail** and **payinCustomerPhone** on each case. This four fields must **NOT** be sent with empty values on any case.

## Creation - Online

#### Request

| Field | Description |
| :--- | :--- |
| payinMerchantID | **Integer (Required)** <br> Merchant Identifier for the Payin API. |
| payinAmount | **Decimal (Required)** <br> The amount of the transaction. Use 2 decimals. |
| payinCurrency | **String (Required, ISO-4217)** <br> The currency of the transaction <br> **Example** <br> PEN |
| payinMerchantOperationNumber | **String (Required, max-lenght = 50)** <br> Reference number of the sale. <br> This value is used to notify Merchants about a payment. |
| payinMethod | **String (Required)** <br> Payment Method Identifier for the Payin API. <br> EX:_ TCTD, TC, TD, BankTransfer_Businesses, BankTransfer, Cash, QR(_), etc. <br> [Full List](https://payinmonnetpayments.readme.io/reference/payin-method) <br> (\_) **QR code is one use. Once it was use can't be reused by final customer** |
| payinVerification | **String (Required)** <br> HASH SHA512 of: <br> payinMerchantID+ <br> payinMerchantOperationNumber+ <br> payinAmount+ <br> payinCurrency+ <br> KeyMonnet |
| payinTransactionOKURL | **String (Required)** (max 250 chars) <br> URL where the customer will be redirected from the online banking if payment was successful **(Must be Https)** (max 250 chars) |
| payinTransactionErrorURL | **String (Required)** (max 250 chars) <br> URL where the customer will be redirected from the online banking if payment was wrong.**(Must be Https)** (max 250 chars) |
| payinExpirationTime | **Integer (Required)** <br> Time in minutes to expire the token. <br> ** Minimum Suggested: ** <br> Online: 30 minutes <br> Cash: 2 hours |
| payinLanguage | **String (Required, ISO 639-1)** <br> Example: EN, ES, PT, DE, etc. |
| payinCustomerEmail | **String (Required)** <br> Contains the customer's email. Max. 50 chars |
| payinCustomerName | **String (Required)** <br> Contains the customer's first name Max. 30 digits |
| payinCustomerLastName | **String (Required)** <br> Contains the customer's last name. Max. 30 digits |
| payinCustomerTypeDocument | **String (Required)** <br> Contains the customer's type of document. [payinCustomerTypeDocument](https://payinmonnetpayments.readme.io/reference/payincustomertypedocument) |
| payinCustomerDocument | **String (Required)** <br> Contains the customer's document number |
| payinCustomerPhone | **String (Required)** <br> Contains the customer's mobile number (9 digits) |
| payinCustomerAddress | **String (Required)** <br> Customer information |
| payinCustomerCity | **String (Required)** <br> Customer information |
| payinCustomerRegion | **String (Required)** <br> Customer information (Default value is "**Lima**" unless different value is indicated by integration team) |
| payinCustomerCountry | **String (Required)** <br> Customer information |
| payinCustomerZipCode | **String (Required)** <br> Customer information |
| payinCustomerShippingName | **String (Required)** <br> Customer information |
| payinCustomerShippingPhone | **String (Required)** <br> Customer information |
| payinCustomerShippingAddress | **String (Required)** <br> Customer information |
| payinCustomerShippingCity | **String (Required)** <br> Customer information |
| payinCustomerShippingRegion | **String (Required)** <br> Customer information |
| payinCustomerShippingCountry | **String (Required)** <br> Customer information |
| payinCustomerShippingZipCode | **String (Required)** <br> Customer information |
| payinRegularCustomer | **String (Optional)** <br> Customer information |
| payinCustomerID | **String (Optional)** <br> Customer information (**PSPs** must complete submerchant name **mandatory**) |
| payinDiscountCoupon | **String (Requied)** <br> Customer information |
| payinFilterBy | **String (Optional)** <br> Customer information |
| payinProductID | **String (Required)** <br> Product information. **Can not be empty,** could send a default value i.e. "0" |
| payinProductDescription | **String (Required)** <br> Product information. **Can not be empty,** could send a default value i.e. "0" |
| payinProductAmount | **String (Required)** <br> Product information. **Can not be empty,** could send a default value i.e. "0" |
| payinDateTime | **String (Required)** <br> Date of the transaction. **Can not be empty,** could send a default value i.e. current date |
| payinProductSku | **String (Required)** <br> Product information. **Can not be empty,** could send a default value i.e. "0" |
| payinProductQuantity | **String (Required)** <br> Product information. **Can not be empty,** could send a default value i.e. "0" |
| URLMonnet (For API v3) | **String (Required)** <br> Value in cert environment: <https://cert.monnetpayments.com/api-payin/v1/online-payments> <br> Value in prod environment: <br> <https://payin.api.monnetpayments.com/api-payin/v1/online-payments> |
| typePost (For API v3) | **String (Required)** <br> Value: json |

#### Sample request

**Pay attention on fields name format,  must be Camel Case as shown below:**

```json
{
  "payinMerchantID": "00",
  "payinAmount": "00.00",
  "payinCurrency": "PEN",
  "payinMerchantOperationNumber": "0000",
  "payinMethod": "BankTransfer",
  "payinVerification": "string",
  "payinCustomerName": "string",
  "payinCustomerLastName": "string",
  "payinCustomerEmail": "test@test.com",
  "payinCustomerPhone": "0000",
  "payinCustomerTypeDocument": "DNI",
  "payinCustomerDocument": "00000000",
  "payinRegularCustomer": "string",
  "payinCustomerID": "string",
  "payinDiscountCoupon": "string",
  "payinLanguage": "ES",
  "payinExpirationTime": "000",
  "payinDateTime": "YYYY-MM-DD",
  "payinTransactionOKURL": "https://test.com",
  "payinTransactionErrorURL": "https://test.com",
  "payinFilterBy": "string",
  "payinCustomerAddress": "string",
  "payinCustomerCity": "string",
  "payinCustomerRegion": "string",
  "payinCustomerCountry": "Peru",
  "payinCustomerZipCode": "0000",
  "payinCustomerShippingName": "string",
  "payinCustomerShippingPhone": "0000",
  "payinCustomerShippingAddress": "string",
  "payinCustomerShippingCity": "string",
  "payinCustomerShippingRegion": "string",
  "payinCustomerShippingCountry": "Peru",
  "payinCustomerShippingZipCode": "0000",
  "payinProductID": "0000",
  "payinProductDescription": "string",
  "payinProductAmount": "0000",
  "payinProductSku": "string",
  "payinProductQuantity": "0000",
  "URLMonnet": "https://cert.monnetpayments.com/api-payin/v1/online-payments",
  "typePost": "json"
}
```

#### Response

> **🚧 V3**
>
> For V3 a URL will be returned on the response, the merchant has to redirect the shopper to this URL

```json
{
"url": "https://test.test",
"payinErrorCode": "0000",
"payinErrorMessage": "Successfull process",
"payinTrxOperation": "MONTRX207249992409275755"
}
```

## Payment

When the transaction is created, a redirection to the payment gateway will be done.

> **🚧 payinMerchantOperationNumber**
>
> The payinMerchantOperationNumber returned in the response is the unique identifier for the transaction inside of Monnet's system. It is not needed to complete the payment, so it shouldn't be shown to the shopper because it may confuse him.

#### Important

Just because the shopper is redirected to the payinTransactionOKURL does not mean that the payment has been billed correctly. We recommend you to work with the notification system in the following section.
