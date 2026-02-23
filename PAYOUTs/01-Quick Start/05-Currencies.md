# Currencies

The following table provides an overview of the countries Monnet works with and the corresponding currencies. It also includes information on the decimal precision accepted for each currency. Please note that the last digits of the amount will be considered as decimals (except Chile).

Please refer to the table below for more details.

| Country | Currency | Decimal places |
| :--- | :--- | :--- |
| Argentina | ARS (Argentine Peso) | 2 |
| Chile | CLP (Chilean Peso) | 0 |
| Colombia | COP (Colombian Peso) | 2 |
| Ecuador | USD (American Dollar) | 2 |
| Guatemala | GTQ (Quetzal) | 2 |
| Honduras | HNL (Lempira) | 2 |
| Mexico | MXN (Mexican Peso) | 2 |
| Peru | PEN (Peruvian Sol) | 2 |
| Brazil | BRL (Brazilian Real) | 2 |

> [!NOTE]
> **Considerations**
>
> * In the case of **Colombia** it is mandatory that **the last two digits (decimals) are 00.**
>
>   * **For example:**
>
>   Correct amount: 100000 COP\
>   Incorrect amount: 100055 COP

> [!WARNING]
> **IMPORTANT NOTE**
>
> The amount field is treated as an INT field; however, it is important to note that the last 2 digits of the amount will be taken **as decimals**.
>
> Please ensure that you provide the appropriate number of decimals when making transactions in the respective currencies.
>
> **For example:**
>
> If you send the amount: 10000 for Mexico, Monnet will take it as: 100.00 Mexican pesos.\
> If you send the amount: 5000 for Colombia, Monnet will take it as: 50.00 Colombian pesos.
