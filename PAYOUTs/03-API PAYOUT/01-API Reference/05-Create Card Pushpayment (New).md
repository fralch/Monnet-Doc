# Create Card Pushpayment (New)

> [!NOTE]
> **This API is only enabled for Peru. This method is only enabled for VISA cards issued by Peruvian banks.**

This endpoint is available to all merchants who wish to perform **payouts to cards**. The service provides a **user interface (UI)** where the beneficiary can securely enter their card details. To enable this flow, the merchant must specify `"accountType": "Card"` in the request. Once the card data is entered, the card is tokenized and the payout order is executed to the corresponding card.

Below is a detailed description of the integration flow that must be followed to correctly implement this service:

### **Components & Integration Flow**

![Integration Flow](https://1642515047-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FMvYrq8H0R42hedo5qReG%2Fuploads%2FmDicrsUgY6gduJBWkSvv%2Fimage.png?alt=media&token=0dd287ef-81c9-45b6-8f31-78fbc57a2731)

1. The user clicks "Withdraw funds" on the merchant's site.
2. The site embeds an **iframe** pointing to its backend.
3. The **backend** requests the iframe from Monnet via **REST API**, using `api-key` and `api-secret` signed with **HMAC-SHA256**.
4. Monnet generates the payout and responds with the **iframe + JWT** for subsequent frontend requests.
5.  The iframe is rendered by the merchant in the frontend and connects via **WebSocket** to Monnet.
6. The iframe displays a **card tokenization form** and sends the token via REST API (using the previously issued JWT).
7. Monnet sends an Iframe event message indicating process status: `ok` or `fail`.

> [!WARNING]
> The API keys used in this endpoint are the same as those used for the other endpoints.

**Method:** `POST`

```
URL: https://api.payout.monnet.io/api-payout-plus-real-time/{merchantId}
```

## POST /api-payout-plus-real-time/{merchantId}

> Request Monnet iframe for CARD payout

```json
{
  "openapi": "3.0.3",
  "info": {
    "title": "Monnet Payout Plus Real Time API",
    "version": "1.0.0"
  },
  "servers": [
    {
      "url": "https://api.payout.monnet.io"
    }
  ],
  "security": [
    {
      "monnetAuth": []
    }
  ],
  "components": {
    "securitySchemes": {
      "monnetAuth": {
        "type": "apiKey",
        "in": "header",
        "name": "monnet-api-key"
      }
    }
  },
  "paths": {
    "/api-payout-plus-real-time/{merchantId}": {
      "post": {
        "summary": "Request Monnet iframe for CARD payout",
        "operationId": "requestIframe",
        "tags": [
          "Payouts"
        ],
        "parameters": [
          {
            "name": "merchantId",
            "in": "path",
            "required": true,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "timestamp",
            "in": "path",
            "required": true,
            "schema": {
              "type": "integer"
            }
          },
          {
            "name": "embed-jwt",
            "in": "query",
            "required": false,
            "schema": {
              "type": "boolean",
              "default": true"
            }
          },
          {
            "name": "proxy-requests",
            "in": "query",
            "required": false,
            "schema": {
              "type": "boolean",
              "default": false"
            }
          }
        ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "required": [
                  "country",
                  "amount",
                  "currency",
                  "orderId",
                  "beneficiary",
                  "destination",
                  "timeoutUrl",
                  "email",
                  "document",
                  "accountType"
                ],
                "properties": {
                  "country": {
                    "type": "string"
                  },
                  "amount": {
                    "type": "integer"
                  },
                  "currency": {
                    "type": "string"
                  },
                  "orderId": {
                    "type": "string"
                  },
                  "beneficiary": {
                    "type": "object",
                    "required": [
                      "name",
                      "lastName",
                      "email",
                      "document"
                    ],
                    "properties": {
                      "name": {
                        "type": "string"
                      },
                      "lastName": {
                        "type": "string"
                      },
                      "email": {
                        "type": "string",
                        "format": "email"
                      },
                      "document": {
                        "type": "object",
                        "required": [
                          "type",
                          "number"
                        ],
                        "properties": {
                          "type": {
                            "type": "integer"
                          },
                          "number": {
                            "type": "string"
                          }
                        }
                      }
                    }
                  },
                  "destination": {
                    "type": "object",
                    "required": [
                      "bankAccount"
                    ],
                    "properties": {
                      "bankAccount": {
                        "type": "object",
                        "required": [
                          "accountType"
                        ],
                        "properties": {
                          "accountType": {
                            "type": "integer",
                            "description": "Use type 5 for CARD payouts"
                          }
                        }
                      }
                    }
                  },
                  "timeoutUrl": {
                    "type": "string"
                  },
                  "merchantLogo": {
                    "type": "string"
                  },
                  "expirationTime": {
                    "type": "string",
                    "description": "Time in minutes before the iframe expires"
                  }
                }
              }
            }
          }
        },
        "responses": {
          "200": {
            "description": "Iframe created successfully",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "iframe": {
                      "type": "string"
                    }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

> [!WARNING]
> The parameters embed-jwt and proxy-requests must not both be set to False simultaneously.

**Request Example:**

```json
{
  "country": "PER",
  "amount": 50001,
  "currency": "PEN",
  "orderId": "BJB_12345678",
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
      "accountType": 5
    }
  },
  "timeoutUrl": "https://www.test.com",
  "merchantLogo": "https://www.test.com",
  "expirationTime": "03"
}
```

> [!WARNING]
> This method is only available for **accountType = 5**.

**Response Example:**

```json
{
  "iframe": "https://payout-plus.monnet.io/..."
}
```

### **Iframe Events**

The iframe communicates events using the JavaScript `postMessage` system:

```js
{ topic: "event_name", data: {} }
```

> Note: The `data` variable is currently empty, but it may be used in the future for additional information.

#### Available Events:

| `topic`                       | Meaning                                                                           |
| ----------------------------- | --------------------------------------------------------------------------------- |
| `mps-payout-finished-timeout` | Payout timed out (It is triggered after 15 minutes without receiving information) |
| `mps-payout-finished-ok`      | Payout completed successfully                                                     |
| `mps-payout-finished-fail`    | Payout failed                                                                     |

The iframe events are not exclusive to the information sent via the webhook. The transaction response (status, error messages, reasons) will be received through the **webhook**.
