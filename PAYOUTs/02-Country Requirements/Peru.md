# Peru

The necessary information for Peru is presented below in order to carry out a correct payout.

## Mandatory parameters

| Field                                 | Description                                                                                                                                                                                                                                |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| country                               | <p><strong>String (Required- Lenght: 3 characters)</strong> </p><p> ISO 3166-1-alpha-3. In the case of Peru: PER</p>                                                                                                                       |
| amount                                | <p><strong>Integer (Required max-lenght = 10 digits)</strong></p><p>The amount of the transaction. <strong>The 2 last digits will be consider as decimal. Ex:</strong><br><strong>100000 equals "one thousand"</strong></p>                |
| currency                              | <p><strong>String (Required-  Lenght: 3 digits)</strong> </p><p>The currency of the payroll. ISO-4217. In the case of PER: PEN o USD<br>You can see the explication on <a data-mention href="../quick-start/currencies">currencies</a></p> |
| orderId                               | <p> <strong>String (Required- Max Lenght: 20 characters)</strong> </p><p>Merchant transaction code. This ID must be unique per transaction.</p>                                                                                            |
| description                           | <p> <strong>String (Optional - Max Lenght: 50 characters)</strong></p><p>For customer use. This is the name that will appear in the payment reference for transfers to <strong>YAPE</strong> (037) and it´s mandatory.</p>                 |
| beneficiary.name                      | <p> <strong>String (Required)</strong> </p><p>Name of the Beneficiary</p>                                                                                                                                                                  |
| beneficiary.lastName                  | <p> <strong>String (Required)</strong> </p><p>Last name of the Beneficiary</p>                                                                                                                                                             |
| beneficiary.email                     | <p> <strong>String (Optional- Max Lenght: 50 characters)</strong> </p><p>Beneficiary's email. It is <strong>mandatory</strong> if account type is "Card". Do not send a dummy value.</p>                                                   |
| beneficiary.document.type             | <p><strong>Integer (Required)</strong> </p><p>Contains the beneficiary's document type. You must enter the parameter as indicated in "Document Validations"</p>                                                                            |
| beneficiary.document.number           | <p> <strong>String (Required)</strong> </p><p>Contains the beneficiary's document number. You can see how many digits must it have in "Document Validations"</p>                                                                           |
| destination.bankAccount.bankCode      | <p><strong>String (Optional - Lenght: 3 characters)</strong></p><p>Code of the receiving bank or financial institution as indicated in "Bank List". It is <strong>mandatory</strong> if account type is "Cellphone Number (4)". </p>       |
| destination.bankAccount.accountType   | <p><strong>Integer (Required)</strong><br>Type of Account of the beneficiary to pay. You must enter the numeric code.</p>                                                                                                                  |
| destination.bankAccount.accountNumber | <p><strong>Integer (Optional)</strong><br>Beneficiary's account number. In the case of account type is "Cellphone Number"(4),  the merchant have to send the phone number. Ex: 948474375</p>                                               |
| destination.bankAccount.cci           | <p><strong>String (Required - Lenght: 20 digits)</strong> </p><p>"Codigo de Cuenta Interbancaria (CCI)". Interbanking number account.</p>                                                                                                  |
| subMerchantInfo.code                  | <p><strong>String (Required only for PSP)</strong><br>Submerchant Identifier</p>                                                                                                                                                           |
| subMerchantInfo.name                  | <p><strong>String (Required only for PSP)</strong><br>Submerchant Name. This is the name that will appear in the payment reference for transfers to <strong>YAPE</strong> (037) and it´s mandatory.</p>                                    |
| subMerchantInfo.url                   | <p><strong>String (Required only for PSP)</strong><br>Submerchant URL</p>                                                                                                                                                                  |
| TimeoutURL                            | <p><strong>String</strong> <strong>(Required only for AccountType "Card")</strong></p><p>The URL of the merchant's application used to redirect the user in the event of a timeout during the payment process."</p>                        |
| merchantLogo                          | <p><strong>String (Optional)</strong></p><p>URL of the merchant’s logo to be displayed on the web payment form (AccountType "Card"). Including a logo is highly recommended for branding purposes. Suggested dimensions: 187x40px.</p>     |
| expirationTime                        | <p><strong>String (Optional)</strong></p><p>The duration of the tokenization session (AccountType "Card") in minutes (Min 5 min - Max 30 min). After this time elapses, the session will expire and the process must be restarted.</p>     |

{% hint style="success" %}
New Feature:

**Email Notification to the Beneficiary**

It is now possible to automatically send emails to the beneficiary with the outcome of the Payout (success or rejected).

**Requirements:**

* Request the activation of **"Email notifications to the user"** through your account manager.
* Include the `beneficiary.email` parameter in each transaction. If this field is missing, the email will not be sent.
  {% endhint %}

{% hint style="info" %} <mark style="color:red;">**IMPORTANT**</mark>

* **For Account Type "Cellphone Number" the only currency available is "PEN"**
* **For Account Type "Cellphone Number" it's mandatory the parameter "bankCode"**
* For Account Type **"Cellphone Number",** the maximum amount allowed is 500 PEN. However, if the wallet is **Yape**, the maximum amount allowed is 3,000 PEN. For other Account Types, the maximum amount allowed is 30,000 PEN.
* We offer payments to **real-time entities and non-real-time entities**, applying the appropriate treatment in each case.
* **Real-time payments**: The **CCI** (*destination.bankAccount.cci*) is a mandatory field.
* **Non-real-time payments**: These are processed as **interbanking transfers**, subject to the applicable processing times indicated in the SLA.
  {% endhint %}

{% hint style="info" %} <mark style="color:red;">**IMPORTANT**</mark>

**Card Pushpayment&#x20;**<mark style="color:green;">**(Available from Q3 2025)**</mark>

This is a new payout method which enables real-time fund transfers to VISA credit, debit, or prepaid cards. This solution operates under the **OCT (Original Credit Transfer)** standard, allowing for secure and direct payments to cards.\
To use this service, a specific configuration must be requested and coordinated in advance with **your account manager.**

To use **Card Pushpayment** service, merchants must consume the dedicated **`Create Card Pushpayment`** endpoint ([Review our documentation](https://docs.monnet.io/OoF1XuWTEpPjI0i2ERRO/api-payout/api-reference/create-card-pushpayment-new)), which is **exclusively intended for `accountType` = `5` (Card)**.\
⚠️ **Do not use the `Create Payout` endpoint** for this type of transaction.

Key implementation considerations:&#x20;

* The "Create Card Pushpayment" endpoint responds with an HTML template that must be embedded by the merchant **within an iFrame**. This template renders the payment form where the user inputs their card information.
* The parameter **Account Type** must be sent with the value **5** (Card).
* The parameter **email** is **mandatory** and must belong to the **cardholder**. Do not send dummy or placeholder data.
* The parameters **bankCode** , **bankAccount.cc**i and **bankAccount.accountNumber are optional.**
* The **Card on File** functionality is **enabled by default**. It allows storing the user's card for future withdrawals, eliminating the need to re-register the card for each transaction. This feature can be disabled if needed, in which case the card registration will be required for every transaction.
  {% endhint %}

## Request Example

**Payout example for Bank Transfer**

```json
{
	"country": "PER",
	"amount": 100000,
	"currency": "PEN",
	"orderId": "R123456",
	"description": "FreeTextFreeTextFreeTextFreeText",
	"beneficiary": {
		"name": "Sergio",
		"lastName": "test",
		"email": "test@test.com",
		"document": {
			"type": 1,
			"number": "12345678"
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

**Payout example for Cellphone Number (Wallets)**

```json
{
	"country": "PER",
	"amount": 100000,
	"currency": "PEN",
	"orderId": "R123456",
	"description": "FreeTextFreeTextFreeTextFreeText",
	"beneficiary": {
		"name": "Sergio",
		"lastName": "test",
		"email": "test@test.com",
		"document": {
			"type": 1,
			"number": "12345678"
		}
	},
	"destination": {
		"bankAccount": {
			"bankCode": "037",
			"accountType": "4",
			"accountNumber": "938267873"
		}
	}
}
```

**Card Pushpayment example**

```json
{
  "country": "PER",
  "amount": 50001,
  "currency": "PEN",
  "orderId": `XYZ12345`,
  "beneficiary": {
    "name": "Juan",
    "lastName": "Prueba",
    "email": "a@a.com",
    "document": {
      "type": 4,
      "number": "20629932888"
    }
  },
  "destination": {
    "bankAccount": {
      "accountType": 5,
    }
  },
  "timeoutUrl": "https://www.test.com",
  "merchantLogo": "https://www.test.com",
  "expirationTime": "03"
}
```

{% hint style="danger" %} <mark style="color:red;">**Time Outs**</mark>&#x20;

In the case of receiving a <mark style="color:red;">**Time Out**</mark> as a response from this API we recommend resending the payout again after 30 seconds using the same **orderId**.&#x20;

This way we will check if the payout was processed and avoid double payments.
{% endhint %}

## Document Validations

| Parameter | ID Type         | Lenght        |
| --------- | --------------- | ------------- |
| 1         | DNI             | 8 digits      |
| 2         | PAS             | 7 - 12 digits |
| 3         | CE              | 8 - 12 digits |
| 4         | RUC<sup>1</sup> | 11 digits     |

{% hint style="warning" %}
(1) RUC is not available for Scotiabank´s accounts (BankCode 004)
{% endhint %}

## Account Type

| Parameter | Type                                                                                                                                                        |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1         | Current Account                                                                                                                                             |
| 2         | Savings Account                                                                                                                                             |
| 4         | Cellphone Number                                                                                                                                            |
| 5         | Card (Exclusive for "[Create Card Pushpayment](https://docs.monnet.io/OoF1XuWTEpPjI0i2ERRO/api-payout/api-reference/create-card-pushpayment-new)" endpoint) |

> In the **"Account Type - Cellphone Number"** field, the corresponding cellphone number must be entered to process the **payout** correctly.
>
> The payout can be made not only to wallets (Yape, Plin, Bim) but also to financial entities (Bank List) associated with the cellphone number. For example, a cellphone number could have 3 entities associated: "Yape", "Plin" and "Banco Pichincha".&#x20;

## Account Number Information

| Bank                                        | Description                                                                                                                             |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Banco BBVA (001)                            | <p>Field “accountNumber” must contain 18 or 20 digits<br>EX: 001106660100012345 (18 digits)<br>EX: 00110666010001234512 (20 digits)</p> |
| Banco de Credito (002)                      | <p>Field “accountNumber” must contain:<br>- 13 digits for Current or Master Account<br>- 14 digits for Savings Account</p>              |
| <p></p><p>Interbank (003)</p><p></p>        | Field “accountNumber” must contain 13 digits                                                                                            |
| Scotiabank (004)                            | <p>Field “accountNumber” must contain 10 digits<br>0037651234 (10 digits: 3 agency + 7 account)</p>                                     |
| For any other bank or financial institution | A 20-digit CCI interbank account code must be entered. Example: 00219400254640654321 (20 digits)                                        |
| Wallets (Yape,Plin, Bim, CCE directory)     | Field “accountNumber” must contain 9 digits and start with "9".                                                                         |

### Bank List&#x20;

Below are the entities we support according to the processing time (real-time or batch).

<table><thead><tr><th width="249">ID</th><th>Bank Name</th><th data-type="checkbox">Real-Time Transfer</th><th data-type="checkbox">No Real-Time Transfer</th></tr></thead><tbody><tr><td>001</td><td>Banco BBVA</td><td>true</td><td>true</td></tr><tr><td>002</td><td>Banco de Credito</td><td>true</td><td>true</td></tr><tr><td>003</td><td>Interbank</td><td>true</td><td>true</td></tr><tr><td>004</td><td>Scotiabank</td><td>true</td><td>true</td></tr><tr><td>005</td><td>Banco de Comercio</td><td>true</td><td>true</td></tr><tr><td>006</td><td>Banco Interamericano de Finanzas - BanBif</td><td>true</td><td>true</td></tr><tr><td>007</td><td>Banco Pichincha</td><td>true</td><td>true</td></tr><tr><td>008</td><td>Citibank</td><td>false</td><td>true</td></tr><tr><td>011</td><td>Banco GNB</td><td>true</td><td>true</td></tr><tr><td>014</td><td>Banco Santander</td><td>false</td><td>true</td></tr><tr><td>016</td><td>Banco Cencosud</td><td>false</td><td>true</td></tr><tr><td>017</td><td>ICBC PERU BANK</td><td>false</td><td>true</td></tr><tr><td>018</td><td>Banco de la Nación</td><td>true</td><td>true</td></tr><tr><td>019</td><td>Caja Arequipa</td><td>true</td><td>true</td></tr><tr><td>020</td><td>Caja Cusco</td><td>true</td><td>true</td></tr><tr><td>021</td><td>Caja Huancayo</td><td>true</td><td>true</td></tr><tr><td>022</td><td>Caja Maynas</td><td>false</td><td>true</td></tr><tr><td>023</td><td>Caja Metropolitana</td><td>true</td><td>true</td></tr><tr><td>024</td><td>Caja Municipal Ica</td><td>true</td><td>true</td></tr><tr><td><del>026</del></td><td><del>Caja Sullana</del></td><td>false</td><td>false</td></tr><tr><td>027</td><td>Caja Tacna</td><td>false</td><td>true</td></tr><tr><td>028</td><td>Caja Trujillo</td><td>true</td><td>true</td></tr><tr><td>029</td><td><del>Credinka</del></td><td>false</td><td>false</td></tr><tr><td>030</td><td>Banco Falabella</td><td>true</td><td>false</td></tr><tr><td>031</td><td>Caja Los Andes</td><td>true</td><td>false</td></tr><tr><td>032</td><td>Mibanco</td><td>true</td><td>true</td></tr><tr><td>033</td><td><del>Bim (Wallet)</del></td><td>false</td><td>false</td></tr><tr><td>034</td><td>Caja Piura</td><td>true</td><td>true</td></tr><tr><td>035</td><td>Ripley</td><td>true</td><td>false</td></tr><tr><td>036</td><td>Financiera Efectiva</td><td>true</td><td>false</td></tr><tr><td>037</td><td>Yape (Wallet)</td><td>true</td><td>false</td></tr><tr><td>038</td><td>Compartamos Financiera</td><td>true</td><td>false</td></tr><tr><td>039</td><td>Banco Alfin</td><td>true</td><td>false</td></tr><tr><td>040</td><td>Plin (Wallet)</td><td>true</td><td>false</td></tr><tr><td>041</td><td>Cooperativa Abaco</td><td>true</td><td>false</td></tr><tr><td>042</td><td><del>Luqea (Wallet)</del></td><td>false</td><td>false</td></tr><tr><td>043</td><td>Tarjetas Peruanas Prepago - Ligo (Walllet)</td><td>true</td><td>false</td></tr><tr><td>044</td><td>Financiera Confianza</td><td>true</td><td>false</td></tr><tr><td>045</td><td>Prex (Wallet)</td><td>true</td><td>false</td></tr><tr><td>046</td><td>Tarjeta Oh</td><td>true</td><td>false</td></tr><tr><td>047</td><td>Crediscotia</td><td>true</td><td>true</td></tr><tr><td>048</td><td><del>Dale (Wallet)</del></td><td>false</td><td>false</td></tr><tr><td>049</td><td><del>Global66 (Wallet)</del></td><td>false</td><td>false</td></tr></tbody></table>
