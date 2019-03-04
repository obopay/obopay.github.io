# Payments Integration for Merchants

Obopay offers payments service in compliance to RBI PPI (Prepaid Payment Instrument) rules and regulations. End users wishing to make payments via Obopay, must complete the KYC process before availing payment services. 

Merchants can integrate directly with Obopay payments system via an SDK on following channels:

- iOS
- Android
- Web 

For each channel, Payments SDK aims at providing the common flow prebuilt for the merchant, making integration very simple. 

This documentation is currently covers details of Android SDK in detail. Web & iOS details are WIP.

## End user payments flow via merchant

1) An user wishes to make a purchase on merchant app / website. Merchant invokes the request via payments SDK. 
2) On Web-channel, Obopay Payments SDK displays a QR Code that can be directly scanned by the users via Obopay app. On App-channels, the payment request is securely passed to Obopay app (without user intervention)
3) User verifies the payment details (amount, beneficiary) and keys in hir M-Pin on the app to approve of payment.
4) On successful authentication, Obopay executes the transfer from user to the merchant's account and invokes a pre-configured callback url of the merchant server with result. In case of any failure, same url is called back with failed status.


## Integration Steps on Android

The Android SDK seemlessly manages the payment flow so that you can focus more on providing service and less on payment intricacies.

To use the Obopay Payments SDK in your app, make it a dependency in Maven.

- In your project, open **your_project > Gradle Scripts > build.gradle (Project)** and make sure to add the following  

      buildscript {
        repositories {
          jcenter{}
        }
      }

- In your project, open `your_project > Gradle Scripts > build.gradle(Module:app)` and add the following implementation statement to `dependencies` section depending the latest version of the SDK always.

      implementation 'com.obopay.android:payments:[1,2)'

- Initialize the SDK: On the application OnCreate() or on your launcher activity, initialize the SDK as following:

      ObopayPayments.initialize(merchantId)
      MerchantId is allocated by Obopay at the time of business setup.

- When user wishes to make a payment:

  - Call your server to allocate a `PaymentContextId`. PaymentContextId is described in section below.
  
  - Create a purchase request like:

    ObopayPayments.requestPayment(callingActivity, PaymentContextId, amount, message);

        callingActivity   : Activity that launches the payment request
        PaymentContextId  : Unique context from step above
        amount            : Amount for which transaction has been initiated 
        message           : Optional message / reason for which 
                            transaction has been initiated 
                            (shown on My Obopay App)

  - Response:

    Response is received in the calling activities onActivityResult callback like:

        onActivityResult(int requestCode, int resultCode,
                Intent data)

        requestCode : set to ObopayPayments.PAYMENT_REQUEST
        resultCode  : is zero when request is completed. Otherwise it is 
                      set to one of the error codes listed below.
        data        : Receives fields when result code is zero

    Error codes:

        ObopayPayments.UNSUPPORTED_SDK - You need to upgrade bundled SDK
        ObopayPayments.AMOUNT_INVALID  - Invalid amount (negative or above max)
        ObopayPayments.MESSAGE_INVALID - Invalid message (invalid length of message)
        ObopayPayments.USER_CANCELED   - User pressed cancel
        ObopayPayments.INVALID_STATE   - Multiple reasons like: 
                                         - Network unhealthy
                                         - Obopay SDK missing
                                         - Not enough balance
                                         - Device Not authenticated

    Except when result code is zeros, you needn't do anything. The error codes are for your information only.

  - Check transaction status

    On getting the onActivityResult callback with result code zeros, the app should request it's server for the payment status. Obopay server sends the result of the payment to merchant server on a pre-configured URL. This is done to ensure that payment responses are not lost in case of loss of connectivity between client and server.

### Payment Context Id

`PaymentContextId` represents unique context of purchase across all users. Obopay allows only single payment under this Id in its payment system. PaymentContextId helps solve duplicate payment problem from a user, while providing a context that can later be queried upon for status.

PaymentContextId is generated by Merchant's server. Here are few examples on how to generate PaymentContextId:

- For Merchant system that use shopping card checkout, should ideally use the 'order number' as PaymentContextId. This will help ensure single payment from user against an order. Orders that are abandoned by the users for payment can be retried with same order number.

- Payment against an invoice (bill): similar to 'shopping cart checkout' should use invoice number / bill number as PaymentContextId

- Direct purchase of an item: If the item cannot be repurchased, itemId can be used as the PaymentContextId. If item can be repurchased after few days, PaymentContextId can be set like dateOfPurchase + '|' + itemId

- In a rare situation, for adhoc payment without any context, PaymentContextId can be uniquely generated like userId + '|' + timestamp. This solution is undesirable, you should strictly avoid this.

