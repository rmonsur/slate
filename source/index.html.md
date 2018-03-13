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

Payments at /payments/ endpoint

This is where we send all payment requests to GreatLakes Loan Provider. 

Loans at /loans/ endpoint

This is where we retrieve loan provider login credentials as well as loan balance information of a user. 


# Users
## Creating new users
This is when a new user signs up for Piecewise. 

```
On success:
({
  status: 200
  success: true,
  message: "Thanks for signing up for Piecewise!"
  msg: doc._id     
});
```

### HTTP Request
`POST '/user/signup/newuser'`

```
On failure:

({
  status: 11000,
  success: false,
  msg: "There is already a user with this email."
});
```

To create a new user, a POST request must be sent with the following parameters in its request body:

### Query Parameters
Parameter | Description
--------- | -----------
firstName | The first name of the user
lastName | The surname name of the user
email | The email of the user to sign up for a new account
password | The password of the user to sign up for a new account
mobileNumber | The mobile number to validate the user. 


<aside class="notice">
  Once the user submits these information, they will have to verify the information. A code is sent to their Mobile
  Phone Number and if the code is valid, they are allowed to continue onboarding. 
</aside>

<aside class="success">
Remember — successful verification returns the user id so we can continue onboarding the user. The user id is key 
to verifying the user when the user submits the verification code. 
</aside>

## Verifying the user
This is where we verify the user after they have signed up using their name, email, password and mobile phone number. 

```
On success:
({
  status: 200
  success: true,
  message: "Thanks for vaidating your account!"
  msg: doc._id     
});
```

### HTTP Request
`POST '/user/signup/:id/verify'`

```
On failure, there are 3 options:
1. We failed to find the user information in our database

({
  status: 500,
  success: false,
  msg: "User not found in database. Please try again."
});

2. The code was invalid

({
  status: 500,
  success: true,
  msg: "The code you entered was invalid - please try again."
});

3. We failed to validate the code due to connectivity issues. 

({
  status: 500,
  success: false,
  msg: "There were connection issues. Please try again later."
});
```

To verify a user who is signing up, the POST request must have the following parameters in its request body:

### Query Parameters
Parameter | Description
--------- | -----------
code | the verification code itself
id | the user id


<aside class="success">
Remember — successful verification returns the user id so we can continue onboarding the user. We use user id to query users and add new information.
</aside>

## Authenticating the user
This is where we authenticate the user when the user is logging/signing in to their Piecewise account. 

### HTTP Request
`POST '/user/login/authenticate'`

```
On success:
({
  status: 200
  success: true,
  loggedInUser: "loggedInUser"
  token: "JWT"+ token 
});

where 
loggedInUser = {
    _id:user._id,
    email:user.email,
    firstname:user.firstName,
    lastname: user.lastName,
    accesstoken: user.plaidAccessToken,
    itemid: user.plaidItemID
};

```

To authenticate a user who is signing/logging in, the POST request must have the following parameters in its request body:

```
On failure, there are 2 options:
1. We failed to find the user information in our database

({
  status: 500,
  success: false,
  msg: "No user with that email was found."
});

2. The password was incorrect

({
  status: 500,
  success: false,
  msg: "You entered the password incorrectly."
});

```

### Query Parameters
Parameter | Description
--------- | -----------
email | the verification code itself
password | the user id


# Payments
## Sending One Time Payment

This is where we send one time payment on behalf of user to Greatlakes. 
### HTTP Request
`POST '/payments/greatlakes/onetime'`

### Query Parameters

Parameter | Description
--------- | -----------
id | The id of the user sending the payment
payment_amount | The payment amount being sent to Greatlakes.

<aside class="warning">
   The minimum payment_amount has to be $5 since GreatLakes only access $5 or more amounts.
</aside>

## Sending Monthly Payments

This is where we send the monthly payment due on behalf of user to GreatLakes. 
### HTTP Request
`POST '/payments/greatlakes/monthly'`

### Query Parameters

Parameter | Description
--------- | -----------
id | The id of the user sending the payment

<aside class="notice">
   We only need the id because our database stores the monthly payment due of each user. 
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
