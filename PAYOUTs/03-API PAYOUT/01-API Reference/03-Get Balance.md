# Get Balance

This GET method is used to retrieve the merchant’s current balance. It includes detailed information by country and currency.

## Get merchant balance

> Returns the current financial balance of the merchant, including both\
> available balance and total balance, organized by country and currency.

```json
{"openapi":"3.0.3","info":{"title":"Get Balance API","version":"1.0.0"},"servers":[{"url":"https://api.monnet.com/api/v1/{merchantId}","variables":{"merchantId":{"description":"The unique merchant identifier."}}}],"paths":{"/balance":{"get":{"summary":"Get merchant balance","description":"Returns the current financial balance of the merchant, including both\navailable balance and total balance, organized by country and currency.\n","operationId":"getBalance","responses":{"200":{"description":"Balance successfully retrieved","content":{"application/json":{"schema":{"type":"array","items":{"type":"array","items":{"$ref":"#/components/schemas/BalanceItem"}}}}}},"401":{"description":"Unauthorized"},"500":{"description":"Internal Server Error"}}}}},"components":{"schemas":{"BalanceItem":{"type":"object","description":"Balance information by country and currency.","properties":{"country":{"type":"string","description":"ISO 3166-1 alpha-3 country code."},"currency":{"type":"string","description":"ISO 4217 currency code."},"availableBalance":{"type":"number","description":"Amount available for use."},"balance":{"type":"number","description":"Total account balance."}}}}}}
```

> [!WARNING]
> Please note that the last digits of the amount will be considered as decimals (except Chile).
>
> **For example:**
>
> If you receive the amount: 10000 for Mexico, this means: 100.00 Mexican pesos.\
> If you receive the amount: 10000 for Chile, this means: 10000 Chilean pesos.
>
> For more information about currency handling, visit the [documentation](../../01-Quick%20Start/05-Currencies.md).
