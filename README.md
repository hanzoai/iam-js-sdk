# iam-js-sdk

[![NPM version][npm-image]][npm-url]
[![NPM download][download-image]][download-url]
[![codebeat badge](https://codebeat.co/badges/6f2ad052-7fc8-42e1-b40f-0ca2648530c2)](https://codebeat.co/projects/github-com-iam-iam-js-sdk-master)
[![GitHub Actions](https://github.com/hanzoai/iam-js-sdk/actions/workflows/release.yml/badge.svg)](https://github.com/hanzoai/iam-js-sdk/actions/workflows/release.yml)
[![GitHub Actions](https://github.com/hanzoai/iam-js-sdk/actions/workflows/build.yml/badge.svg)](https://github.com/hanzoai/iam-js-sdk/actions/workflows/build.yml)
[![Coverage Status](https://codecov.io/gh/iam/iam-js-sdk/branch/master/graph/badge.svg)](https://codecov.io/gh/iam/iam-js-sdk)
[![Release](https://img.shields.io/github/release/iam/iam-js-sdk.svg)](https://github.com/hanzoai/iam-js-sdk/releases/latest)
[![Discord](https://img.shields.io/discord/1022748306096537660?logo=discord&label=discord&color=5865F2)](https://discord.gg/5rPsrAzK7S)

[npm-image]: https://img.shields.io/npm/v/iam-js-sdk.svg?style=flat-square
[npm-url]: https://npmjs.com/package/iam-js-sdk

[download-image]: https://img.shields.io/npm/dm/iam-js-sdk.svg?style=flat-square
[download-url]: https://npmjs.com/package/iam-js-sdk

This is IAM's SDK for js will allow you to easily connect your application to the [IAM authentication system](https://iam.org) without having to implement it from scratch.

IAM SDK is very simple to use. We will show you the steps below.

## Usage in NPM environment

### Installation

~~~shell script
# NPM
npm i iam-js-sdk

# Yarn
yarn add iam-js-sdk
~~~

### Init SDK

Initialization requires 5 parameters, which are all string type:

| Name (in order)  | Must | Description                                                                                    |
|------------------|------|------------------------------------------------------------------------------------------------|
| serverUrl        | Yes  | your IAM server URL                                                                        |
| clientId         | Yes  | the Client ID of your IAM application                                                      |
| appName          | Yes  | the name of your IAM application                                                           |
| organizationName | Yes  | the name of the IAM organization connected with your IAM application                   |
| redirectPath     | No   | the path of the redirect URL for your IAM application, will be `/callback` if not provided |
| signinPath       | No   | the path of the signin URL for your IAM application, will be `/api/signin` if not provided |

```typescript
import {SDK, SdkConfig} from 'iam-js-sdk'

const sdkConfig: SdkConfig = {
    serverUrl: "https://door.casbin.com",
    clientId: "014ae4bd048734ca2dea",
    appName: "app-casnode",
    organizationName: "casbin",
    redirectPath: "/callback",
    signinPath: "/api/signin",
}
const sdk = new SDK(sdkConfig)
// call sdk to handle
```

## Usage in vanilla Javascript

### Import and init SDK

Initialization parameters are consistent with the previous node.js section:

```html
<!--init the SDK-->
<script type="module">
  //Import from cdn(you can choose the appropriate cdn source according to your needs), or just from the local(download the iam-js-sdk first)
  import SDK from 'https://unpkg.com/iam-js-sdk@latest/lib/esm/sdk.js'
  const sdkConfig = {
    serverUrl: "https://door.casbin.com",
    clientId: "014ae4bd048734ca2dea",
    appName: "app-casnode",
    organizationName: "casbin",
    redirectPath: "/callback",
    signinPath: "/api/signin",
  }
  window.sdk = new SDK(sdkConfig)
</script>
```

### Call functions in SDK

```html
<script type="text/javascript">
  function gotoSignUpPage() {
    window.location.href = sdk.getSigninUrl()
  }
</script>
```

## API reference interface

#### Get sign up url

```typescript
getSignupUrl(enablePassword)
```

Return the iam url that navigates to the registration screen

#### Get sign in url

```typescript
getSigninUrl()
```

Return the iam url that navigates to the login screen

#### Get user profile page url

```typescript
getUserProfileUrl(userName, account)
```

Return the url to navigate to a specific user's iam personal page

#### Get my profile page url

```typescript
getMyProfileUrl(account)
```

#### Sign in

```typescript
signin(serverUrl, signinPath)
```

Handle the callback url from iam, call the back-end api to complete the login process

#### Determine whether silent sign-in is being used

```typescript
isSilentSigninRequested()
```

We usually use this method to determine if silent login is being used. By default, if the silentSignin parameter is included in the URL and equals one, this method will return true. Of course, you can also use any method you prefer.

#### silentSignin


````typescript
silentSignin(onSuccess, onFailure)
````

First, let's explain the two parameters of this method, which are the callback methods for successful and failed login. Next, I will describe the execution process of this method. We will create a hidden "iframe" element to redirect to the login page for authentication, thereby achieving the effect of silent sign-in.

#### popupSignin


````typescript
popupSignin(serverUrl, signinPath)
````
Popup a window to handle the callback url from iam, call the back-end api to complete the login process and store the token in localstorage, then reload the main window. See Demo: [iam-nodejs-react-example](https://github.com/iam/iam-nodejs-react-example).

### OAuth2 PKCE flow sdk (for SPA without backend)

#### Start the authorization process

Typically, you just need to go to the authorization url to start the process. This example is something that might work in an SPA.

```typescript
signin_redirect();
```

You may add additional query parameters to the authorize url by using an optional second parameter:

```typescript
const additionalParams = {test_param: 'testing'};
signin_redirect(additionalParams);
```

#### Trade the code for a token

When you get back here, you need to exchange the code for a token.

```typescript
sdk.exchangeForAccessToken().then((resp) => {
    const token = resp.access_token;
    // Do stuff with the access token.
});
```

As with the authorizeUrl method, an optional second parameter may be passed to the exchangeForAccessToken method to send additional parameters to the request:

```typescript
const additionalParams = {test_param: 'testing'};

sdk.exchangeForAccessToken(additionalParams).then((resp) => {
    const token = resp.access_token;
    // Do stuff with the access token.
});
```

#### Parse the access token

Once you have an access token, you can parse it into JWT header and payload.

```typescript
const result = sdk.parseAccessToken(accessToken);
console.log("JWT algorithm: " + result.header.alg);
console.log("User organization: " + result.payload.owner);
console.log("User name: " + result.payload.name);
```

#### Get user info

Once you have an access token, you can use it to get user info.

```typescript
getUserInfo(accessToken).then((resp) => {
    const userInfo = resp;
    // Do stuff with the user info.
});
```

#### Refresh access token

You could use a refresh token, to get a new token from the oauth server when token expired.

```typescript
sdk.refreshAccessToken(refreshToken).then((resp) => {
    const token = resp.access_token;
    // Do stuff with new access token
});
```

#### A note on Storage
By default, this package will use sessionStorage to persist the pkce_state. On (mostly) mobile devices there's a higher chance users are returning in a different browser tab. E.g. they kick off in a WebView & get redirected to a new tab. The sessionStorage will be empty there.

In this case it you can opt in to use localStorage instead of sessionStorage:

```typescript
import {SDK, SdkConfig} from 'iam-js-sdk'

const sdkConfig = {
  // ...
  storage: localStorage, // any Storage object, sessionStorage (default) or localStorage
}

const sdk = new SDK(sdkConfig)
```

## More examples

To see how to use iam frontend SDK with iam backend SDK, you can refer to examples below:

[casnode](https://github.com/casbin/casnode): iam-js-sdk + iam-go-sdk

[iam-python-vue-sdk-example](https://github.com/iam/iam-python-vue-sdk-example): iam-vue-sdk + iam-python-sdk



A more detailed description can be moved to:[iam-sdk](https://iam.org/docs/how-to-connect/sdk)
