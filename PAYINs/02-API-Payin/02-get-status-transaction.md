Get Status Transaction

# Get Status Transaction

This method is used to verify the status of the transaction. You can get the status using POST method to the API

## Request

> 👍 Environments
>
> Certification: <https://cert.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations>\
> Production: <https://apiin.monnetpayments.com/ms-experience-payin/merchant/{MID}/operations>

There is also neccesary to add a header called authorization that will have a SHA256 value from the concatenation of: MerchantID + KeyMonnet

| Name                         | Description                                                                                        |
| :--------------------------- | :------------------------------------------------------------------------------------------------- |
| payinStartDate               | Initial date of searching (Date) (can't use current day, but current day -1 or any other past day) |
| payinEndDate                 | Last date of searching (Date)                                                                      |
| payinMerchantOperationNumber | Transaction identifier (String)                                                                    |

## "Get Status" body request examples

### Example 1: Using all Parameters

```json V3
data: {
  "payinStartDate": "2024-01-01",
  "payinEndDate": "2024-02-14",  
  "payinMerchantOperationNumber": "1234567890"  
}
```

### Example 2: By date range

If search is to be made by date range both dates are mandatory

```json V3
data: {  
  "payinStartDate": "2024-01-01",
  "payinEndDate": "2024-02-14"
}
```

### Example 3: By Operation ID

If search is to be made by Operation ID it must be done as follows:

```json V3
data: {  
  "payinMerchantOperationNumber": "1234567890"  
}
```

### Example 3: All transactions

If all transactions need to be retrieve leave empty value for field **payinMerchantOperationNumber**

```json V3
data: {  
  "payinMerchantOperationNumber": "" //empty value 
}
```

## "Get Status" response example

```json V3
{
    "payinMerchantID": "999",
    "operations": [
        {
            "payinStateID": "5",
            "payinState": "AUTORIZADO",
            "payinAmount": "31980.0",
            "payinCurrency": "CLP",
            "payinMerchantOperationNumber": "1234567890",
            "payinID": "0987654321"
        }        
      ]
}
```