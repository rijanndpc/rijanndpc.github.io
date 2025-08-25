# NamastePay Checkout

## Workflow
### 1. Merchant initiates a checkout
Checkout can be initiated by a **server side** POST request to the `/checkout/api/initiate` endpoint.

```sh
curl \
  "http://testpay.namastepay.com/checkout/api/initiate" \
  --request POST \
  --header "Authorization: Basic $(echo -n $PHONE:$PIN | base64)" \
  --header "Content-Type: application/json" \
  --data-raw '
  {
    "remarks": "TEST",
    "amount": 75000,
    "amount_breakdown": {
      "Amount": 100000,
      "Discount": -25000
    },
    "return_url": "http://example.com"
  }'
```

**Output**

```json
{
  "id": "NSJSDrljYeW5vagedvZ",
  "url": "http://testpay.namastepay.com/checkout?id=NSJSDrljYeW5vagedvZ",
  "expires_on": "2025-08-14 10:45:13.920809"
}
```

Headers must consist of following fields:

- `Content-Type` must be set to `application/json`
- `Authorization` must use `Basic` scheme with base64 encoded credentials
  in the form `phone_number:pin`

Payload consists of following fields:

- `amount` is a mandatory integer field
  - represents the total amount in paisa
  - minimum and maximum limits can be set in the server
- `remarks` is a mandatory string field
  - currently no restrictions placed on `remarks`
- `return_url` is a mandatory string field
  - must be a valid url
- `amount_breakdown` is an optional map field
  - each entry in `amount_breakdown` contains a key and a value
  - the key is a string representing the label
  - the value is an integer representing amount in paisa
  - if the `amount_breakdown` field is present then sum of all values must be equal to `amount`

Response payload consists of following fields:

- `id`: the unique id of the checkout
- `url`: the url to which user should be redirected
- `expires_on`: datetime at which the checkout will expire

### 2. User is redirected to `url` returned in the result
The url leads to our checkout page which shows the `remarks`, `amount` and `amount_breakdown` alongside a login prompt

### 3. User enters their phone number, pin and optionally a promo code
- The promo code, if given, is verified
- Entered phone number and pin are verified
- OTP is sent to the user
- Page updates to show a prompt for OTP

### 4. User enters OTP
- OTP is verified: the unique id of the checkout
url: the url to which user should be redirected
- Page updates to show checkout status

### 5. User is redirected to `return_url`

### 6. Merchant enquires the checkout status
Checkout status can be enquired by calling the `/checkout/api/enquire` endpoint.
```sh
curl \
  "http://testpay.namastepay.com/checkout/api/enquire" \
  --request POST \
  --header "Authorization: Basic $(echo -n $PHONE:$PIN | base64)" \
  --header "Content-Type: application/json" \
  --data-raw '
  {
    "id" : "NSJSDrljYeW5vagedvZ"
  }'
```

**Output**
```json
{
  "status": "expired",
  "total_amount": 75000,
  "order_id": null
}
```

## Notes
- User can choose to cancel the checkout before or after they log in
- Checkout expires after 15 minutes
