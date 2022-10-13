<script src="https://unpkg.com/mermaid@8.9.3/dist/mermaid.min.js"></script>
<script>

    window.addEventListener("DOMContentLoaded", (event) => {
        mermaid.initialize({
            startOnLoad:true,
            theme: "default",
        });
        window.mermaid.init(undefined, document.querySelectorAll('.language-mermaid'));    });
</script>


# CibleR API implementation documentation


### Environnements

Recette:
[https://qa.cibler.io](https://qa.cibler.io)

Production:
[https://prod.cibler.io](https://prod.cibler.io)

### Configuration variables
###### customerId : XX
###### api Key : XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
Those varaibles will be given to you by CibleR.

### Order tracking API request
Call this API server side (Your Site --> CibleR) for each paid order.
```http
 POST :  [ENV]/api/campaignBehaviors/order/{customerId}
 ```

#### Body exemple
```JSON
    {
      "orderContext":{
        "key": "d4c6c0b6-03fb-468e-b635-fe1ce4a44080",
        "email":"test@gmail.com",
        "phone": "3360000000",
        "country":"FR / FRA",
        "environnement":"Prod",
        "siteId": 13,
        "customerGuid":"bbc47c1f-020e-4885-a01d-53f5921a7210",
        "cookieId":"CIB1.395406195.1569857337",
       "userBalance": 75
      },
      "orderInformation":{
        "orderId":0,
        "totalSpent":75.50,
       "point": 75,
        "taxes": 2.00,
        "giftCode":"MANO66FZC4F4RTZ",
        "shippingCost":10.00,
        "discountAmount":2.50,        
        "orderLines": [
            {
                "productId": 5,
                "name": "Perceuse",
                "quantity": 2,
                "category": 11,
                "orderType": "Standard",
                "unitPrice": 34.00,
                "sellerId": 0
            }
        ]
      }
    }
 ```

#### Properties definitions

 |  Property                             | Type    | Description                                           | Mandatory |
 |---------------------------------------|:--------|:------------------------------------------------------|:----------|:---|
 | orderContext.key                      | String  | Api key given by CibleR                               | YES       |
 | orderContext.email                    | String  | Email of the user                                     | YES       |
 | orderContext.phone                    | String  | phone number of the user                               | NO       |
 | orderContext.country                  | String  | Country code                                          | YES       |
 | orderContext.environnement            | String  | Environnement of the website                          | YES       |
 | orderContext.siteId                   | Integer | Unique identifer of the website                       | YES       |
 | orderContext.customerGuid             | String  | Internal unique identifier of the user                | YES       |
 | orderContext.userBalance             | Long  | current number of loyalty point after purchase          | NO       |
 | cookieId                              | String  | Value of the cookie named cibler_id                   | YES       |
 | orderInformation.orderId              | Long    | Internal unique identifier of the order               | YES       |
| orderInformation.points               | Long    | addition loyalty point for this purchase               | NO       |
 | orderInformation.totalSpent           | Double  | Total amount of the order taxes and shipping included | YES       |
 | orderInformation.taxes                | Double  | Taxes                                                 | YES       |
 | orderInformation.shippingCost         | Double  | Cost of shipping                                      | YES       |
 | orderInformation.giftCode             | String  | Code used by the user (can be null)                   | YES       |
 | orderInformation.discountAmount       | Double  | Amount of discount (can be null)                      | YES       |
 | orderInformation.orderLines           | Array   | List of products                                      | NO        |
 | orderInformation.orderLines.productId | String  | Id or SKU of the product                              | YES       |
 | orderInformation.orderLines.name      | String  | Name of the product                                   | NO        |
 | orderInformation.orderLines.quantity  | Integer | Quantity of products                                  | YES       |
 | orderInformation.orderLines.category  | String  | Category of the product                               | NO        |
 | orderInformation.orderLines.orderType | String  | Standard or Marketplace                               | YES       |
 | orderInformation.orderLines.unitPrice | Double  | Unit price of the product taxes included              | YES       |
 | orderInformation.orderLines.sellerId  | Long    | Id of the seller                                      | NO        |


## Automatic coupon generation

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
The api should accept the following request body and create the voucher in your system.
Response is not required, return status should be 200-OK.

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



### Contact
If you have any trouble contact us at support@cibler.com

