<script src="https://unpkg.com/mermaid@8.9.3/dist/mermaid.min.js"></script>
<script>

    window.addEventListener("DOMContentLoaded", (event) => {
        mermaid.initialize({
            startOnLoad:true,
            theme: "default",
        });
        window.mermaid.init(undefined, document.querySelectorAll('.language-mermaid'));    });
</script>


# Automatic coupon generation
In this documentation we will describe two method to allow CibleR to generate coupon in your system.
The first one is done on the fly in the cart validation or orderconfirmation step, it creates coupon only when they're about to be used by a client.
The second one sends data back to your backend  as soon as a user receives a coupon.

## On the fly validation & generation

### Sequence diagram One Coupon OK

 ```mermaid
 sequenceDiagram
 finalUser->>ecommerce-backend: enter CibleR coupon in the order
 ecommerce-backend->>ecommerce-backend: if coupon is not a ecommerce-backend coupon
 ecommerce-backend->>backend : POST giftCode/Validate
 backend->>ecommerce-backend : canUse = true, combinable = true
 ecommerce-backend->>ecommerce-backend : create a ecommerce-backend coupon
 ecommerce-backend->>ecommerce-backend : add the coupon to the order
 finalUser->>ecommerce-backend : make the order with the coupon
 ecommerce-backend->>backend : POST campaignBehaviors/order with CibleR coupon
 backend->>backend : Update state giftCode / prize
 ```

### Sequence diagram Two Coupons OK

 ```mermaid
 sequenceDiagram
 finalUser->>ecommerce-backend: enter coupon A CibleR in the order
 ecommerce-backend->>ecommerce-backend: if the coupon is not a ecommerce-backend coupon
 ecommerce-backend->>backend : POST giftCode/Validate
 backend->>ecommerce-backend : canUse = true, combinable = true
 ecommerce-backend->>ecommerce-backend : create a ecommerce-backend coupon
 ecommerce-backend->>ecommerce-backend : add the coupon to the order
 finalUser->>ecommerce-backend: enter coupon B CibleR in the order
 ecommerce-backend->>ecommerce-backend: if coupon is not a ecommerce-backend coupon
 ecommerce-backend->>backend : POST giftCode/Validate
 backend->>ecommerce-backend : canUse = true, combinable = true
 ecommerce-backend->>ecommerce-backend : create a ecommerce-backend coupon
 ecommerce-backend->>ecommerce-backend : add the coupon to the order
 finalUser->>ecommerce-backend : make the order with 2 coupons
 ecommerce-backend->>backend : POST campaignBehaviors/order with CibleR coupons
 backend->>backend : update state giftCode / prize
 ```

### Sequence diagram Two Coupons KO

 ```mermaid
 sequenceDiagram
 finalUser->>ecommerce-backend: enter coupon A CibleR in the order
 ecommerce-backend->>ecommerce-backend: if the coupon is not a ecommerce-backend coupon
 ecommerce-backend->>backend : POST giftCode/Validate
 backend->>ecommerce-backend : canUse = true, combinable = true
 ecommerce-backend->>ecommerce-backend : create a ecommerce-backend coupon
 ecommerce-backend->>ecommerce-backend : add the coupon to the order
 finalUser->>ecommerce-backend: enter coupon B CibleR in the order
 ecommerce-backend->>ecommerce-backend: if coupon is not a ecommerce-backend coupon
 ecommerce-backend->>backend : POST giftCode/Validate
 backend->>ecommerce-backend : canUse = true, combinable = false
 ecommerce-backend->>ecommerce-backend : Do not create ecommerce-backend coupon
 finalUser->>ecommerce-backend : make the order with only 1 coupon (A)
 ecommerce-backend->>backend : POST campaignBehaviors/order with coupon A CibleR
 backend->>backend : Update state giftCode / prize
 ```

### Sequence diagram Two Coupons KO

 ```mermaid
 sequenceDiagram
 finalUser->>ecommerce-backend: enter coupon A CibleR in the order
 ecommerce-backend->>ecommerce-backend: if the coupon is not a ecommerce-backend coupon
 ecommerce-backend->>backend : POST giftCode/Validate
 backend->>ecommerce-backend : canUse = true, combinable = false
 ecommerce-backend->>ecommerce-backend : create a ecommerce-backend coupon
 ecommerce-backend->>ecommerce-backend : add the coupon to the order
 finalUser->>ecommerce-backend: enter coupon B CibleR in the order
 ecommerce-backend->>ecommerce-backend: if coupon is not a ecommerce-backend coupon
 ecommerce-backend->>backend : POST giftCode/Validate
 backend->>ecommerce-backend : canUse = true, combinable = true
 ecommerce-backend->>ecommerce-backend : Do not create ecommerce-backend coupon
 finalUser->>ecommerce-backend : make the order with only 1 coupon (A)
 ecommerce-backend->>backend : POST campaignBehaviors/order with coupon A CibleR
 backend->>backend : Update state giftCode / prize
 ```

#### API request
Request must be done server side when a coupon is entered by the end user.
If coupon is not one of your coupon, you must call the API to verifiy if the coupon is a CibleR coupon and get the values of the coupon.

 ```http
  POST :  [ENV]/api/giftCodes/validate/{customerId}
  ```

##### Exemple de body
 ```JSON
 {
   "email" : "christine@orange.fr",
   "cibler_id" : "AZ34R5TYU7UY654RE32RT6YT",
   "code" : "BJGJT9DJAAAA",
   "cart": {
      "total" : 132600,
      "items" : [{
        "product_id":"CANOE123",
        "category":675,
        "quantity":1,
        "name":"ACME Foldable Canoe",
        "unit_price":126900,
        "price_currency_code":"EUR"
      }, {
        "product_id":"TOOTHBR",
        "category":13,
        "quantity":3,
        "name":"Large yellow toothbrush",
        "unit_price":1900,
        "price_currency_code":"EUR"
      }]
    }
 }
  ```

##### Response example
When coupon is valid.
 ```JSON
 {
     "canUse": true,
     "message": null,
     "value": 5.0,
     "threshold": 30.00,
     "combinable": false,
     "discountType": "VALUE",
     "description": "5€",
     "expirationDate": "2021-06-23",
     "products": null,
     "categories": null
 }
 ```
When coupon is not valid
 ```JSON
 {
     "canUse": false,   
     "message": "Ce code n'existe pas",
     "value": null,
     "threshold": null
 }
 ```

#### Response Properties definitions

|  Property      | Type           | Description                                                                                         | Mandatory |
|:----------------|:---------------|:----------------------------------------------------------------------------------------------------|:----------|:---|
| canUse         | Boolean        | True if coupon can be used by the user                                                              | YES       |
| message        | String         | Error message                                                                                       | YES       |
| value          | Double         | Value of the coupon                                                                                 | YES       |
| threshold      | Double         | Threshold for this coupon                                                                           | YES       |
| combinable     | Boolean        | True is the coupon can be used with other coupons                                                   | YES       |
| discountType   | Enum           | Defines the type of the coupon VALUE, PERCENT, FREESHIPPING                                         | YES       |
| expirationDate | Datetime       | Date when the coupon will expire                                                                    | YES       |
| products       | List of String | When this list if filled, this means that the coupon is only eligible on the products of the list   | YES       |
| categories     | List of String | When this list if filled, this means that the coupon is only eligible on the categories of the list | YES       |

# Coupon creation API

### Request example
Alternatively you can provide an secured API for CibleR to call to create a voucher in your system. 
The API can be secured by API-KEY or OAUTH scheme, contact us for any other security scheme.
Response is not required, return status should be 200-OK.

The api should accept the following request body and create the a voucher based on these constraint in your system.
##### Body example
```JSON
{
  "code": "YHBGFREZERFRE",
  "value": 10,
  "discountWholeCart" : 1,
  "name" : "Code 10% CibleR",
  "startDate”:“2023-12-01T02:00:00.000Z",
  "discountType": "PERCENT or VALUE",
  "threshold": 100,
  "description": "10 %",
  "combinable": true,
  "expirationDate": "2019-07-03",
  "campaignId": 63006938,
  "customerId": 4,
  "numberOfUsage": 43,
  "user": {
    "email": "john@gmail.com",
  },
  "products": [
    "PS4V2",
    "XBOXONEX"
  ],
  "categories": [
    "TELEPHONE",
    "532543"
  ]
}
```


##### Properties specifications
- user can be null
- products list can be empty. If set it means the created coupon must only be used on the specified produtc
- categories can be empty. If set, it means the created coupons must only be used on the specified categories
