# Create Payout

This method consists of creating a payout and sending it to Monnet to be paid.

It is the use of an API with a POST method in which the information of the beneficiary used to make the corresponding dispersion will be sent to us.

Request example with all the fields:

## Create a payout

> Creates a payout and sends it to Monnet for processing.\
> If a Time Out response is received, resend the request after 30 seconds using the same `orderId`\
> to verify if it was processed and avoid duplicate payments.

```json
{"openapi":"3.0.3","info":{"title":"Create Payout API","version":"1.0.0"},"servers":[{"url":"https://api.monnet.com/api/v1/{merchantId}","variables":{"merchantId":{"description":"The unique merchant identifier."}}}],"paths":{"/payouts":{"post":{"summary":"Create a payout","description":"Creates a payout and sends it to Monnet for processing.\nIf a Time Out response is received, resend the request after 30 seconds using the same `orderId`\nto verify if it was processed and avoid duplicate payments.\n","operationId":"createPayout","requestBody":{"required":true,"content":{"application/json":{"schema":{"$ref":"#/components/schemas/CreatePayoutRequest"}}}},"responses":{"201":{"description":"OK","content":{"application/json":{"schema":{"$ref":"#/components/schemas/CreatePayoutResponse"}}}},"400":{"description":"Missing or invalid data","content":{"application/json":{}}},"401":{"description":"Data validation error","content":{"application/json":{}}}}}}},"components":{"schemas":{"CreatePayoutRequest":{"type":"object","required":["country","amount","currency","orderId","beneficiary","destination"],"properties":{"country":{"type":"string","description":"ISO 3166-1 alpha-3 code of the country (e.g., MEX, PER)"},"amount":{"type":"number","description":"The transaction amount"},"currency":{"type":"string","description":"Currency code in ISO 4217 format (e.g., MXN, PEN)"},"orderId":{"type":"string","description":"Merchant payout ID (Purchase Order)"},"description":{"type":"string","description":"Optional description for merchant use"},"beneficiary":{"$ref":"#/components/schemas/Beneficiary"},"destination":{"$ref":"#/components/schemas/Destination"},"subMerchantInfo":{"$ref":"#/components/schemas/SubMerchantInfo"}}},"Beneficiary":{"type":"object","required":["name","lastName","email","document"],"properties":{"customerId":{"type":"string"},"userName":{"type":"string"},"name":{"type":"string"},"lastName":{"type":"string"},"email":{"type":"string"},"phoneNumber":{"type":"string"},"document":{"type":"object","required":["type","number"],"properties":{"type":{"type":"integer"},"number":{"type":"string"}}},"address":{"$ref":"#/components/schemas/Address"}}},"Address":{"type":"object","properties":{"street":{"type":"string"},"houseNumber":{"type":"string"},"additionalInfo":{"type":"string"},"city":{"type":"string"},"province":{"type":"string"},"country":{"type":"string"},"zipCode":{"type":"string"}}},"Destination":{"type":"object","required":["bankAccount"],"description":"Destination account information where the payout will be sent.\n","properties":{"bankAccount":{"type":"object","required":["bankCode","accountType","accountNumber"],"properties":{"bankCode":{"type":"string","description":"Destination bank code. You must enter the numeric code as indicated on the Bank Code Block."},"accountType":{"type":"string","description":"Destination account type. You must enter the numeric code as indicated on the Account Type Block."},"accountNumber":{"type":"string","description":"Destination account number."},"clabe":{"type":"string","description":"Destination account CLABE (Only for MEX)."},"cbu":{"type":"string","description":"Destination account CBU (Only for ARG)."},"location":{"type":"object","properties":{"street":{"type":"string","description":"Contains the beneficiary account's street."},"houseNumber":{"type":"string","description":"Contains the beneficiary account's house number."},"additionalInfo":{"type":"string","description":"Contains the beneficiary account's additional info."},"city":{"type":"string","description":"Contains the beneficiary account's city."},"province":{"type":"string","description":"Contains the beneficiary account's province."},"country":{"type":"string","description":"Contains the beneficiary account's country."},"zipCode":{"type":"string","description":"Contains the beneficiary account's zip code."}}}}}}},"SubMerchantInfo":{"type":"object","properties":{"code":{"type":"string"},"name":{"type":"string"},"url":{"type":"string"}}},"CreatePayoutResponse":{"type":"object","properties":{"payout":{"type":"object","properties":{"Id":{"type":"integer"},"country":{"type":"string"},"amount":{"type":"number"},"currency":{"type":"string"},"orderId":{"type":"string"}}},"output":{"type":"object","properties":{"stage":{"type":"string"},"status":{"type":"string"},"statusChangeDateTime":{"type":"string","format":"date-time","description":"This timestamp field in the API response is standardized to UTC (Coordinated Universal Time)"}}}}}}}}
```

```json
{
	"country": "PER",
	"amount": 100000,
	"currency": "PEN",
	"orderId": "R123456",
	"description": "Optional Description",
	"beneficiary": {
		"customerId": "12345",
		"userName": "test",
		"name": "Test",
		"lastName": "Test",
		"email": "test@test.com",
		"phoneNumber": "5491128480000",
		"document": {
			"type": 1,
			"number": "12345678"
		},
		"address": {
			"street": "9 de Julio",
			"houseNumber": "123",
			"additionalInfo": "Extra information",
			"city": "Lima",
			"province": "Lima",
			"zipCode": "1408"
		}
	},
	"destination": {
		"bankAccount": {
			"bankCode": "001",
			"accountType": "1",
			"accountNumber": "00000000000",
			"alias": "",
			"cbu": "",
			"cci": "",
			"clabe":"",
			"location": {
				"street": "9 de Julio",
				"houseNumber": "123",
				"additionalInfo": "Extra information",
				"city": "",
				"province": "Lima",
				"country": "PER",
				"zipCode": "1408"
			}
		}
	},
	"subMerchantInfo": {
		"code": "1111",
		"name": "Prueba",
		"url": "http://prueba.com"
	}
}
```

> [!NOTE]
> When creating a Payout, there are optional fields that you can provide. Although not mandatory, including this information can improve the user's payment experience. Consider these fields if you wish to provide additional details.

> [!CAUTION]
> **Time Outs**
>
> In the case of receiving a **Time Out** as a response from this API we recommend resending the payout again after 30 seconds using the same **orderId**.
>
> This way we will check if the payout was processed and avoid double payments.
