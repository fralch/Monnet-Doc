# Authorization Protocol: OAuth 2.0 bearer tokensen

In the standard authorization protocol based on OAuth 2.0 bearer tokens merchants obtain a JSON Web Token or JWT using the OAuth 2.0 Client Credentials provided by Monnet (user and password).

After that, the client must include the JWT, received in the response, in requests to Monnet APIs using the **Authorization: \<access\_token>** HTTP header.

#### Endpoints <a href="#endpoints" id="endpoints"></a>

In order to authenticate HTTP requests by the OAuth 2.0 Protocol, merchants must call the following endpoint according to the target environment:

* CERT: <https://cert.authentication.monnet.io/ms-authentication-service/authentication/api-integration/token>
* PROD: <https://authentication.monnet.io/ms-authentication-service/authentication/api-integration/token>

#### Header and Request <a href="#header-and-request" id="header-and-request"></a>

All calls to the token endpoint must be made over HTTPS. The client must include the headers and request structure listed below:

**Header:**

| Header       | Value            | Required | Description                                                           |
| ------------ | ---------------- | -------- | --------------------------------------------------------------------- |
| Content-Type | application/json | Yes      | This header indicates that the request body is sent using JSON format |

**Request:**

Specifies the structure and mandatory parameters that must be included in the request body to ensure the successful creation of a JSON Web Token.

Specifies the structure and mandatory parameters that must be included in the request body to ensure the successful creation of a JSON Web Token.

{ \
"user":"merchantuser\_1", \
"password":"123abc" \
}

Field details:

| Field    | Type   | Required | Description                                                                                                               |
| -------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| userName | String | Yes      | Technical and unique identifier of each merchant, it is assigned for authentication purposes by Monnet                    |
| password | String | Yes      | Secret credential associated with the client identifier. It is used to authenticate the request and is assigned by Monnet |
|          |        |          |                                                                                                                           |

#### Response <a href="#response" id="response"></a>

After calling the token endpoint, the merchant will receive one of the following responses:

1. Successful response:\
   If the user and password provided in the request are correct, the API returns a successful response with the following structure:

   {\
   "access\_token":"dsc6284ynb262ynj8ik9opl2km5nhnb", \
   "expires\_in":300, \
   "refresh\_expires\_in":1800, \
   "refresh\_token":"etd7395zoc373zok9jl9pq23ln6oioc", \
   "token\_type":"Bearer"\
   &#x20;}

\
Field details:

| Field                | Type                          | Required | Description                                                                                                                                                               |
| -------------------- | ----------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| access\_token        | String                        | Yes      | JWT access token issued by Keycloak. It must be included in the **Authorization: \<access\_token>** header in all subsequent API requests.                                |
| expires\_in          | Integer (Expresed in seconds) | Yes      | Token validity period expressed in seconds. Indicates how long the access token will remain valid before expiration.                                                      |
| refresh\_expires\_in | Integer (Expresed in seconds) | No       | <p>Specifies the lifetime of the refresh token in seconds. After this time the refresh token can no longer be used.<br>This field is part of a future implementation.</p> |
| refresh\_token       | String                        | No       | <p>Token that can be used after the access\_token is expires, without resubmitting user credentials.<br>This field is part of a future implementation.</p>                |
| token\_type          | String                        | No       | Indicates the token format. Typically returned as **"Bearer"**.                                                                                                           |

2. Error response:\
   If the user and password provided in the request are incorrect, the API returns an error response with the following structure:<br>

   HTTP Status Code: 401 - Unauthorized\
   Example:

   {

   "message":"Invalid credentials"\
   }

Field details:

| Field   | Type   | Required | Description                                                                                         |
| ------- | ------ | -------- | --------------------------------------------------------------------------------------------------- |
| message | String | Yes      | Textual description of the error indicating the reason why the authentication request was rejected. |

#### 🔐 Authorization Header <a href="#authorization-header" id="authorization-header"></a>

All requests to the endpoints that require this second mechanism for authentication must include a custom **Authorization** header to ensure secure communication and proper merchant validation.

| Header        | Value                | Required | Description                                                                              |
| ------------- | -------------------- | -------- | ---------------------------------------------------------------------------------------- |
| Authorization | JSON Web Token (JWT) | Yes      | JWT token used to authenticate the merchant and authorize access to protected endpoints. |

#### ![exclamation mark](https://monnetpayments.atlassian.net/gateway/api/emoji/799e3ebd-b521-4d72-aabd-379bc31b61f5/2757/path?scale=MDPI) Relevant Information <a href="#relevant-information" id="relevant-information"></a>

* The access token is valid for 5 minutes from the time of issuance.
* Once it expires, a new token must be generated.
* The same token can be reused for multiple requests until it expires.
