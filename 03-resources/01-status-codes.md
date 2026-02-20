Status Codes

# Status Codes

During the process, the transaction changes its status depending on specific events

![](https://files.readme.io/5db73de-Diagrama_en_blanco_1.jpeg "Diagrama en blanco (1).jpeg")

| payinStateID | payinState                                                                                    |
| :----------- | :-------------------------------------------------------------------------------------------- |
| 1            | Creado (The payment has been created)                                                         |
| 2            | Pendiente de pago (The payment is pending)                                                    |
| 3            | Expirado (The payment has expired)                                                            |
| 5            | Autorizado (The payment has been completed successfully)                                      |
| 6            | Denegado (The payment has been denied)                                                        |
| 9            | Liquidado (The payment has been settled, this state is available after authorization )        |
| 10           | Reembolsado (The payment has been refunded / Only to non completed successfully transactions) |
| 11           | Devuelto (The payment has been refunded / Only to settled transactions)                       |