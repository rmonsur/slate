---
title: Piecewise API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - API Response

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Piecewise API: Introduction

This is Piecewise API. The API is used to consume, store and send data about a user's login credentials, personal information, loan and loan payments information. 

The API is divided into 3 parts as of now:

Users at /user/ endpoint

This is where we create, validate, verify, authenticate new and existing users and retrieve user related information. 
The /user/ endpoint is entirely for Piecewise's internal use and won't be made public/ 

Payments at /payments/ endpoint

This is where we process payment requests to loan servicers. We are currently integrated with Great Lakes, Nelnet and Navient

Loans at /loans/ endpoint

This is where we retrieve loan provider login credentials as well as loan balance information of a user. 



# Payments
## Sending One Time/Lump Sum Payment

If a user would like to send a one time/lump sum payment, the /payments/onetime endpoint is used. Based on the loan servicer
of the user, the endpoint will vary slightly as shown below. 

```
Example response on success:
({
  status: 200
  success: true,
  message: "Thanks for sending a one time payment of $30.00",
  payment_id: 5a394894ert67opo39r4d,
  user_id: 5b4543a47c89d506836a230e,
  servicer: Nelnet 
});
```

### HTTP Request
`Great Lakes POST '/payments/onetime/greatlakes'`

`Nelnet POST '/payments/onetime/nelnet'`

`Navient POST '/payments/onetime/navient'`


### Query Parameters

Parameter | Description
--------- | -----------
token | An encoded parameter to verify the user who is requesting the payment
payment_amount | The payment amount being sent to the loan servicer.

<aside class="warning">
   The minimum payment_amount has to be $5 for the GreatLakes endpoint as Great Lakes only allow for $5 or more amounts.
</aside>

<aside class="warning">
   The payment route for Great Lakes is rate limited to one payment every 3 days since that is the time period taken by Great Lakes
   to process each payment request. 
</aside>

## Sending Monthly Payments

The monthly payment route follows a similar convention to one time payment standard. 

```
Example response on success:
({
  status: 200
  success: true,
  message: "We just sent your monthly payment of $305.19",
  payment_id: 5c404124rsg6712u39r2q,
  user_id: 5b4543a47c89d506836a230e,
  servicer: Great Lakes 
});
```

### HTTP Request
`Great Lakes POST '/payments/monthly/greatlakes'`

`Nelnet POST '/payments/monthly/nelnet'`


### Query Parameters

Parameter | Description
--------- | -----------
token | An encoded parameter to verify the user who is requesting the payment

<aside class="notice">
   We only need the token because our database stores the monthly payment due of each user. 
</aside>

<aside class="notice">
   Monthly payment is not available for Nelnet at this moment. 
</aside>


## Sending Other types of Payments

Piecewise API can send other forms of payments too- spare change round up payments, payments that are 
a set percentage of all incoming deposits of users, set autopay amount payments etc. The key concept
is that from a high level, all these payments are in essence a type of one time payment. Thats why we 
use the '/payment/onetime/' endpoint for these kind of payments as well. 

Here is a list of all the payment options, Piecewise API can handle:

### Spare Change Round Ups

For every purchase and/or debit transaction (in-person and online), the transaction is rounded up to
the nearest dollar, and once the spare changes accumulate to $10 or more, we send a payment to the 
user's loan servicer.

```
Example response on success:
({
  status: 200
  success: true,
  message: "We just sent $14.19 in spare changes to your loans!",
  payment_id: 5c404124rsg6712u39r2q,
  user_id: 5b4543a47c89d506836a230e,
  servicer: Great Lakes 
});
```


### 1% of Every Incoming Deposits

For every incoming wire transfer or incoming deposit in a user's checking account (could be bi-weekly
wage, cash transfer, PayPal transfer etc), we calculate 1% of each deposit and every month, and send
the accumulated amount to the user's respective loan servicer. 

### Send $10 every week

Self-explainatory- we send an automated $10 weekly payment to the user's respective loan servicer.

### HTTP Request - Great Lakes
`Great Lakes POST '/payments/onetime/sparechange/greatlakes'`

`Great Lakes POST '/payments/onetime/ofdeposits/greatlakes'`

`Great Lakes POST '/payments/onetime/recurring/weekly/greatlakes'`


### HTTP Request - Nelnet
`Nelnet POST '/payments/onetime/sparechange/nelnet'`

`Nelnet POST '/payments/onetime/ofdeposits/nelnet'`

`Nelnet POST '/payments/onetime/recurring/weekly/nelnet'`


### HTTP Request -Navient
`Navient POST '/payments/onetime/sparechange/navient'`

`Navient POST '/payments/onetime/ofdeposits/navient'`

`Navient POST '/payments/onetime/recurring/weekly/navient'`

### Query Parameters

Parameter | Description
--------- | -----------
token | An encoded parameter to verify the user who is requesting the payment

<aside class="notice">
   For all these payment types, we only need the token to ID the user making the payment. 
</aside>


# Loans
## Onboarding GreatLakes Credentials

This is part of the onboarding process where we ask users for their Greatlakes Login credentials
and we save the information in the database. 
### HTTP Request
`POST '/loans/glcredentials'`

### Query Parameters

Parameter | Description
--------- | -----------
id | The id of the user sending the payment
loanProviderUsername | The username of the Greatlakes account
loanProviderPIN | The 4 digit PIN of the Greatlakes account
loanProviderPassword | The password of the Greatlakes account
