# CibleR API implementation documentation


### Environnements

Recette:
[https://qa.cibler.io](https://qa.cibler.io)

Production:
[https://prod.cibler.io](https://prod.cibler.io)

### Configuration variables
###### customerId : YOUR CUSTOMER ID
###### api Key : YOUR API KEY


### Order tracking API request
Call this API server side for each paid order on


#### Api call

```http
 POST :  [ENV]/api/campaignBehaviors/order/{customerId}
 ```

#### Body example for a 150CHF deposit with 30CHF bonus code
```JSON
    {
      "orderContext":{
        "key": "xxxx-xxxx-xxxx-xxxx-xxxx",
        "email":"test@gmail.com",
        "country":"CH / FRA",
        "environnement":"Prod",
        "siteId": 13,
        "customerGuid":"bbc47c1f-020e-4885-a01d-53f5921a7210",
        "cookieId":"CIB1.395406195.1569857337"
      },
      "orderInformation":{
        "orderId":234567865,
        "totalSpent":150.0,
        "taxes": 20.00,
        "giftCode":"MANO66FZC4F4RTZ",
        "shippingCost":0.00,
        "discountAmount":30.00
      }
    }
 ```

 #### Body example for a 20CHF bonus code activation
 ```JSON
     {
       "orderContext":{
         "key": "xxxx-xxxx-xxxx-xxxx-xxxx",
         "email":"test@gmail.com",
         "country":"CH / FRA",
         "environnement":"Prod",
         "siteId": 13,
         "customerGuid":"bbc47c1f-020e-4885-a01d-53f5921a7210",
         "cookieId":"CIB1.395406195.1569857337"
       },
       "orderInformation":{
         "orderId":234567866,
         "totalSpent":0.0,
         "taxes": 0.00,
         "giftCode":"MANO66FZC4F4RTZ",
         "shippingCost":0.00,
         "discountAmount":20.00
       }
     }
  ```

 #### Body Properties definitions

 | Property  | Type  | Description | Mandatory  |
 |---|---|---|---|
 | orderContext.key  |  String | Api key given by CibleR |  YES |
 | orderContext.email  |  String | Email of the user  |  YES |
 |  orderContext.country |  String |  Country code |  YES |
  |  orderContext.environnement |  String |  Environnement of the website |  YES |
   |  orderContext.siteId |  Integer |  Unique identifer of the website |  YES |
 | orderContext.customerGuid | String| Internal unique identifier of the user |  YES |
 | cookieId| String | Value of the cookie named cibler_id| YES |
 | orderInformation.orderId | Long | Internal unique identifier of the order | YES |
 | orderInformation.totalSpent | Double | Total amount of the order taxes and shipping included | YES |
 |  orderInformation.taxes| Double | Taxes | YES |
 | orderInformation.shippingCost | Double | Cost of shipping (0 ) | YES |
 | orderInformation.giftCode | String | Code used by the user (can be null) | YES |
 | orderInformation.discountAmount | Double | Amount of bonus of used code (can be null)| YES |




### Contact
If you have any trouble contact us at support@cibler.com

</div>
