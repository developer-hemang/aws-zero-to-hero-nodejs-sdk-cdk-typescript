# AWS Cognito Guide 


## Table Of contents 

* [What Is Cognito](#what-is-cognito)






# What Is Cognito ?

AWS Cognito is a fully managed authentication, authorization, and user management service provided by AWS.

### It helps developers:
- Add sign-up/sign-in functionality
- Manage users securely
- Implement authentication and authorization
- Integrate with social login providers
- Secure APIs and applications
- Generate temporary AWS credentials

Cognito removes the need to build authentication systems manually.



### Real-Life Definition

Suppose you build:
- E-commerce app
- Banking app
- Food delivery app
- SaaS application
- Mobile application


### Users need:

- Registration
- Login
- Forgot password
- MFA
- OTP verification
- Google/Facebook login
- Session management
- Access control

Instead of building all these manually, AWS Cognito provides them as managed services.


## Main Components of Cognito

### AWS Cognito has 2 major components:

| Component      | Purpose                       |
| -------------- | ----------------------------- |
| User Pools     | Authentication (login system) |
| Identity Pools | Authorization (AWS access)    |



# 1. Cognito User Pool

User Pool is a user directory that stores users and handles authentication.

Create a serverless database of user for your web & mobile apps.

### Think of it like:

```bash
"Managed login system by AWS"
```


### It manages:

- Simple Login/Signup: Username (or email) / password combination
- PAssword Reset
- Email & Phone Number Verification
- Multi Factor authentication (MFA)
- Federated Identities: users from Facebook, Google, SAML…
- Feature: block users if their credentials are compromised elsewhere
- Login sends back a JSON Web Token (JWT)



# User Pool Features

### 1. User Registration
Users can register with:
- Email
- Phone number
- Username

```js
Email: test@gmail.com
Password: MyPass@123
```

### 2. Authentication

Supports:
- Username/password login
- OTP login
- Passwordless authentication
- MFA


### 3. Multi-Factor Authentication (MFA)

Extra security layer.
Supported MFA:
- SMS OTP
- TOTP apps (Google Authenticator)

Example flow:

```bash
1. Enter username/password
2. Receive OTP
3. Verify OTP
4. Login success
```

### 4. Email & Phone Verification

Automatically sends:
- Verification email
- SMS verification code


### 5. Password Policies

You can configure:

- Minimum length
- Special characters
- Password expiry
- Password history

### 6. Social Login Federation

Supports:

| Provider |
| -------- |
| Google   |
| Facebook |
| Apple    |
| Amazon   |
| SAML     |
| OIDC     |

#### Example:

```bash
Login with Google
```

# 7. JWT Token Generation

### After login Cognito generates:

| Token         | Purpose             |
| ------------- | ------------------- |
| ID Token      | User identity       |
| Access Token  | API authorization   |
| Refresh Token | Generate new tokens |


Example JWT Flow

```bash
User Login
   ↓
Cognito validates credentials
   ↓
Returns JWT tokens
   ↓
Frontend sends Access Token to API Gateway
   ↓
API Gateway validates token
   ↓
Lambda executes
```


## Cognito Hosted UI

AWS provides ready-made login UI.

Features:
- Login page
- Signup page
- Forgot password
- MFA pages


Benefits:
- No frontend auth UI development
- Faster implementation



Two Ways to Use Cognito Login

| Method            | Description                             |
| ----------------- | --------------------------------------- |
| Cognito Hosted UI | AWS provides ready-made login pages     |
| Custom UI         | You build your own login/signup screens |


### 1. Cognito Hosted UI

AWS gives prebuilt pages for:
- Login
- Signup
- Forgot password
- MFA
- Social login

Example:

```bash
https://your-domain.auth.us-east-1.amazoncognito.com/login
```

Benefits

- ✅ Fast setup
- ✅ Secure
- ✅ Minimal coding
- ✅ Easy social login integration

Limitations

- ❌ Limited customization
- ❌ Branding limitations
- ❌ UI control limited


2. Custom Login Page (Most Common)

You build:
- React login page
- Angular login page
- Mobile app login screen
- Node.js frontend
- Vue frontend

Then frontend communicates with Cognito APIs.

### Architecture

```bash
Custom React Login Page
          ↓
      Cognito APIs
          ↓
     JWT Tokens
          ↓
     API Gateway
          ↓
        Lambda
``` 

### What Happens Internally?

Your frontend sends:

```json
{
  "username": "test@gmail.com",
  "password": "Password@123"
}
```

to Cognito.
Cognito:

- Validates credentials
- Generates JWT tokens
- Returns tokens to frontend


### Services/SDKs Used

Usually:

| Service           | Purpose                   |
| ----------------- | ------------------------- |
| Cognito User Pool | Authentication            |
| AWS Amplify       | Easy frontend integration |
| AWS SDK           | Direct Cognito API calls  |


## Option 1 — Using AWS Amplify (Recommended)

Amplify simplifies Cognito integration.

```js

import { Auth } from 'aws-amplify';

async function signIn() {
  try {
    const user = await Auth.signIn(
      'test@gmail.com',
      'Password@123'
    );

    console.log(user);
  } catch (err) {
    console.log(err);
  }
}
```

### Signup Example
```js
await Auth.signUp({
  username: 'test@gmail.com',
  password: 'Password@123',
  attributes: {
    email: 'test@gmail.com'
  }
});

```

Forgot Password Example

```js
await Auth.forgotPassword('test@gmail.com');
```


## Option 2 — Using AWS SDK Directly

You can directly call Cognito APIs.

Example:

```js
const AWS = require('aws-sdk');

const cognito = new AWS.CognitoIdentityServiceProvider();

const params = {
  AuthFlow: 'USER_PASSWORD_AUTH',
  ClientId: 'CLIENT_ID',
  AuthParameters: {
    USERNAME: 'test@gmail.com',
    PASSWORD: 'Password@123'
  }
};

const response = await cognito.initiateAuth(params).promise();

```

### Features You Can Build in Custom UI

| Feature            | Supported |
| ------------------ | --------- |
| Signup             | Yes       |
| Login              | Yes       |
| Social Login       | Yes       |
| MFA                | Yes       |
| Forgot Password    | Yes       |
| OTP Verification   | Yes       |
| Custom Validation  | Yes       |
| Profile Management | Yes       |


# Cognito User Pools – Lambda Triggers

## What are Cognito Lambda Triggers?

Cognito Lambda Triggers are AWS Lambda functions that execute automatically during different stages of the Cognito authentication lifecycle.


### They allow you to:

- Customize authentication flow
- Validate users
- Modify tokens
- Send custom messages
- Add business logic
- Integrate with external systems


### Why Lambda Triggers Are Used?

Without triggers:

Cognito only provides standard authentication.

With triggers:

You can customize behavior.

Example:

| Requirement               | Trigger Usage        |
| ------------------------- | -------------------- |
| Allow only company emails | Pre Sign-up          |
| Add custom claims in JWT  | Pre Token Generation |
| Send custom OTP email     | Custom Message       |
| Save login history        | Post Authentication  |
| Migrate old users         | User Migration       |


Real-Life Example

Suppose company policy says:

```js
Only @company.com emails allowed
```

During signup:
- Cognito invokes Lambda
- Lambda validates email domain
- Signup allowed or rejected


## Complete Cognito Trigger Flow

```text
User Signup/Login
        ↓
Cognito Event Occurs
        ↓
Lambda Trigger Executes
        ↓
Custom Logic Runs
        ↓
Cognito Continues Flow
```


# Full List of Cognito Triggers

| Trigger               | Purpose                       |
| --------------------- | ----------------------------- |
| Pre Sign-up           | Validate before signup        |
| Custom Message        | Customize emails/SMS          |
| Post Confirmation     | After user confirms account   |
| Pre Authentication    | Before login                  |
| Post Authentication   | After successful login        |
| Define Auth Challenge | Custom auth flow              |
| Create Auth Challenge | Create challenge/OTP          |
| Verify Auth Challenge | Validate challenge            |
| Pre Token Generation  | Modify JWT token              |
| User Migration        | Migrate users from old system |
| Custom Sender         | Custom email/SMS providers    |






# 2. Cognito Identity Pool

Definition

Identity Pool provides:

Temporary AWS credentials for users

It allows users to access AWS services securely.




# Why Identity Pool Needed?

Suppose user uploads image directly to S3.

### Without Identity Pool:

### ❌ You cannot expose AWS IAM credentials to frontend.

### With Identity Pool:

### ✅ Cognito gives temporary limited credentials.


Example

Mobile app user uploads image to S3:

```js
User Login
   ↓
Cognito User Pool authenticates
   ↓
Identity Pool generates temporary AWS credentials
   ↓
User uploads image to S3
```


| AWS Service | Usage                  |
| ----------- | ---------------------- |
| API Gateway | Secure APIs            |
| Lambda      | Backend processing     |
| IAM         | Access control         |
| S3          | File upload            |
| AppSync     | GraphQL authentication |
| CloudFront  | Secure web apps        |
| ALB         | Authentication         |
| EventBridge | Authentication events  |
| SES         | Email verification     |
| SNS         | SMS OTP                |
| CloudWatch  | Monitoring             |
| WAF         | Security protection    |



Authentication Flow in Cognito

Basic Authentication Flow



# Services Commonly Used with Cognito


COgnito USer pool lamda triggers

user pool flow operations description for each operations mention all event triggers 


hosted authentication UI

Hosted UI on custom domain with acm

Adaptive Authentication 

