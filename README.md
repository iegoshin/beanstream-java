Beanstream Java SDK
===============

Integration with Beanstream’s payments gateway is a simple, flexible solution.

You can choose between a straightforward payment requiring very few parameters; or, you can customize a feature-rich integration.

In addition to credit card transactions, Canadian merchants can process INTERAC payments. To assist as a centralized record of all your sales, we also accept cash and cheque transactions.

For very detailed information on the Payments API, look at the Beanstream developer portal's [documentation](http://developer.beanstream.com/documentation/take-payments/purchases-pre-authorizations/).

# Setup
Once you have cloned the repository you will need to load the project in your favourite IDE (NetBeans, Eclipse, etc.)

The project uses [Gradle](http://www.gradle.org/) to build the project. If you are using Netbeans it is very easy to install the Gradle plugin. Just navigate to Tools->Plugins, click on the Available Plugins tab and locate the 'Gradle Support' plugin. Click the Install button.

With the gradle plugin installed you can open the java project by navigating to File->Open Project and locating the java project. 


# Make a Payment
Before you begin making purchases, you need to create a Beanstream API object. It holds your user credentials and provides access to the various APIs.

```java
Gateway beanstream = new Gateway("v1", YOUR_MERCHANT_ID, "YOUR_API_KEY");
```

Next you can create the Card Payment Request, which is a payment using a credit card, and pass that to the Gateway object.

```java
CardPaymentRequest req = new CardPaymentRequest();
	req.setAmount( "100.00" );
	req.setMerchant_id( "300200578" );
	req.setOrder_number( "402" );
	req.getCard().setName( "John Doe" )
		.setNumber( "5100000010001004" )
		.setExpiry_month( "12" )
		.setExpiry_year( "18" )
		.setCvd( "123" );

try {
            
	PaymentResponse response = beanstream.payments().makePayment(req);
	System.out.println("Aproved? "+response.isApproved());
	
} catch (BeanstreamApiException ex) {
	Logger.getLogger(this.getClass().getName()).log(Level.SEVERE, "An error occurred", ex);
}			
```


# Void a Payment
To void a payment you will need a valid paymentId and the amount, and a Beanstream API (you can use the same Beanstream API instance for multiples requests even for different operations like (makePayment, voidPayment,...))


As with makePayment using your configured Beanstream API instance (Gateway object instance) call the voidPayment method of the PaymentAPI.
if  the voidPayment is success you won't have any exception, if no you need to handle the exception.
you can get a list of all Beanstream response codes at http://support.beanstream.com/docs/response-message-codes-descriptions.htm.

```java
String paymentId = ....;
double amount = ....;

try {
	
	PaymentResponse response = beanstream.payments().voidPayment(paymentId, amount);
	// void payment success, your response contains the payment transaction but witht he type 'VP'
} catch (BeanstreamApiException ex) {
	// void payment request failed, handle exception here
	if(ex.getHttpStatusCode()==400) { // bad request
		BeanstreamResponse resp = new BeanstreamResponse(ex.getResponseMessage());
		// resp.code
		// resp.category
		// resp.message
		// resp.reference
	}
	//
}	
```

# Pre Authorize a purchase
Pre authorize is the same process as doing a normal payment, with the only difference that with pre-authorize you have to steps
Pre-authorize a payment. Use this if you want to know if a customer has sufficient funds before processing a payment.
A real-world example of  this is pre-authorizing at the gas pump for $100 before you fill up, then end up only using $60 of gas; the customer is only charged $60. The final payment is used with preAuthCompletion() method.

Step 1 do the pre authorization
```java

		CardPaymentRequest paymentRequest = ...;

		try {
			PaymentResponse response = beanstream.payments()
				.preAuth(paymentRequest);

		} catch (BeanstreamApiException ex) {
			BeanstreamResponse response = new BeanstreamResponse(ex.getResponseMessage());
			//response.code response code from beanstream
			//response.category category of the response code
			//response.message description of the response code
		}

```

Step 2 complete the purchase with the final amount

```java
		try{
			PaymentResponse authResp = beanstream.payments()
						.preAuthCompletion(preAuthorizedPaymentId, finalAmount ,order_number);
		} catch (BeanstreamApiException ex) {
			BeanstreamResponse response = new BeanstreamResponse(ex.getResponseMessage());
			//response.code response code from beanstream
			//response.category category of the response code
			//response.message description of the response code
		}
```