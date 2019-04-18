# Manage-Windows-Subscription

How to manage product subscriptions in Windows Platform?

Prerequisite : 

- Create Azure account
- Create subscription that you need for require intervals
- Need client_id, client_secret and tenantId 

To manage windows purchase subscription and to get other require details we need to follow below mention steps :

#1 Create Azure access_token using below script and access token will expire after 1 hour after generation.

- Here tenantId = XXXXXXXXXXXXXXXXXX

Sample Request :

"curl -X POST --header \"Content-Type: application/x-www-form-urlencoded\" https://login.microsoftonline.com/tenantId /oauth2/token --data \"grant_type=client_credentials&client_id=XXXXXXXXXXXXXXXXXX&client_secret=XXXXXXXXXXXXXXXXXX&resource=https://onestore.microsoft.com\"";

Sample Response :

{
    "token_type": "Bearer",
    "expires_in": "3600",
    "ext_expires_in": "3600",
    "expires_on": "1547203266",
    "not_before": "1547199366",
    "resource": "https://onestore.microsoft.com",
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}

#2 To Get Purchase subscription data by providing b2b key

- Access token you will get from Step #1
- b2b key will pass Windows team while assigning subscription that we need to keep at safe place for future needs.

Sample Request :

"curl -X POST --header \"Authorization: Bearer $access_token\" --header \"Content-Type: application/x-www-form-urlencoded\" --header \"Host: purchase.mp.microsoft.com\" https://purchase.mp.microsoft.com/v8.0/b2b/recurrences/query --data \"b2bKey=$b2b\"";

Sample Response :

Success :

{
    "items": [
        {
            "autoRenew": false,
            "beneficiary": "pub:nileshpatil0810@yahoo.com",
            "expirationTime": "2019-01-10T23:59:59.00+00:00",
            "expirationTimeWithGrace": "2019-01-10T23:59:59.00+00:00",
            "id": "mdr:0:f7786d7c36f2478fa6e5fd9fb4de1026:68e9add8-9bee-4df5-8133-2388bc8c9391",
            "isTrial": true,
            "lastModified": "2019-01-04T12:38:07.48+00:00",
            "market": "IN",
            "productId": "ABCDEFGHIJ",
            "skuId": "0101",
            "startTime": "2019-01-04T00:00:00.00+00:00",
            "recurrenceState": "Active"
        }
    ]
}

Failure : 

{
    "code": "Unauthorized",
    "data": [],
    "details": [],
    "innererror": {
        "code": "AuthenticationTokenInvalid",
        "data": [],
        "details": [],
        "message": "Authentication token supplied is invalid"
    },
    "message": "The client is not authorized to perform the requested operation.",
    "source": "PurchaseFD"
}

- From success response we will get multiple items in case when user has purchase multiple subscription, So to identify particular product we need to store productId and compare with response and can obtain expiry_date, autoRenew and require details from matched occurrence.

#3 How to renew subscription?

- access_token you need to generate from step #1
- b2b that we have stored in our DB
- $data_string = json_encode(array("serviceTicket" => $access_token,'Key' => $b2b));

Sample Request : 

"curl -X POST --header \"Content-Type: application/json\" --header \"Accept: application/json\" --header \"Host: purchase.mp.microsoft.com\" https://purchase.mp.microsoft.com/v6.0/b2b/keys/renew -d '$data_string'";

Sample Reponse :

Success : 

{
    "key": "eyJ0eXAiOiJKV1QiLCJXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}

Failure :

{
    "code": "Unauthorized",
    "data": [],
    "details": [],
    "innererror": {
        "code": "AuthenticationTokenInvalid",
        "data": [],
        "details": [],
        "message": "Authentication token supplied is invalid"
    },
    "message": "The client is not authorized to perform the requested operation.",
    "source": "PurchaseFD"
}

- As from success response we will get key that we need to store/update in our DB for future autorenew.


Note :
- In Azure microsoft we can't create subscription product with duration 6 months.
- b2b key also expiring in 90 days, So to autorenew purpose we keep need to renew b2b key by providing previous b2b key.

Ref link :
- https://docs.microsoft.com/en-us/windows/uwp/monetize/query-for-products
- https://docs.microsoft.com/en-us/windows/uwp/monetize/renew-a-windows-store-id-key
