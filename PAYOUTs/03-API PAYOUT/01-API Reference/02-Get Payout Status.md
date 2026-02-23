# Get Payout Status

This method is used to verify the status of the payroll and the transactions that belong to it

[Get Payout Status.yaml](https://1642515047-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FMvYrq8H0R42hedo5qReG%2Fuploads%2FPt23ye6mSbqG4qAvEPR5%2FGet%20Payout%20Status.yaml?alt=media\&token=f1b0adc0-7cb5-4611-a029-ea9b4074ff34)

Another option is to get a **Rejected Status** from a payout, in that case you will get the next body as response:

```json
{
"payout":{
	"Id": 1111111111,
	"country": "PER",
	"amount": 100000,
	"currency": "PEN",
	"orderId": "R123456",
	"description": "Free Text Free Text",
	"beneficiary":{
		"customerId": "12345",
		"userName": "testUser",
		"name": "test",
		"lastName": "test",
		"email": "test@test.com",
		"phoneNumber": "123456789",
		"document":{
			"type":	1,
			"number": "12345678"
		},
		"address":{
			"street": "9 de Julio",
			"houseNumber": "123",
			"additionalInfo": "Extra Information",
			"city": "LIMA",
			"province": "LIMA",
			"country": "PER",
			"zipCode": "1408"
		}
	},
	"destination": {
		"bankAccount": {
			"bankCode": "001",
			"accountType": "1",
			"location": {
				"street": "9 de Julio",
				"houseNumber": "123",
				"additionalInfo": "Extra Information",
				"city": "LIMA",
				"province": "LIMA",
				"country": "PER",
				"zipCode": "1408"
			}
		}
	},
	"subMerchantInfo":{
		"code": "1111",
		"name": "Prueba",
		"url": "http://prueba.com"
	}
},
"output":{
	"stage": "REJECTED",
	"status": "REJECTED_BANK",
	"statusChangeDateTime": "20220110223001"
},
"errors":[
	{
            "code" : "4001",
            "message" : "Bank Rejection - Beneficiary is not the Account Owner"
        }
]
}
```

> [!NOTE]
> In the Get Payout Status API endpoint, the **{id}** indicated is the one returned by monnet in the Create Payout response.
