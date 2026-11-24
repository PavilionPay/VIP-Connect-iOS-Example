# VIP Connect SDK Integration for iOS

This repository contains a sample application that demonstrates integration and use of the VIP Connect SDK for iOS.

More instructions about the VIP Connect SDK can be found in our [main documentation](https://developer.vippreferred.com/).

## Integration Requirements

To interact with the VIP Connect services, first setup an [operator](https://developer.vippreferred.com/operator-onboarding/operator-setup) with Pavilion Payments.
This will allow you to create an [authentication token](https://developer.vippreferred.com/integration-steps/operator-requirements) for use with the Pavilion APIs
to [create a patron session](https://developer.vippreferred.com/APIS/SDK/create-patron-session) for your customer. 

After getting a session id, [launch the VIP SDK web component via URL](https://developer.vippreferred.com/integration-steps/invoke-web-component) inside
an [`ASWebAuthenticationSession`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession). The `SessionSetupViewController`
class in this demo app demonstrates a simple way to do this.

The sample runs the `ASWebAuthenticationSession` in private (incognito) mode by setting
[`prefersEphemeralWebBrowserSession`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/prefersephemeralwebbrowsersession)
to `true`. The session shares no cookies or website data with Safari or with earlier sessions, and iOS does not show its
"Wants to Use ... to Sign In" prompt before each transaction.

VIP Connect uses [Finicity Connect WebSDK](https://developer.mastercard.com/open-finance-us/documentation/connect/integrating/sdk/web-sdk/sdk/) or [Plaid Hosted Link](https://plaid.com/docs/link/hosted-link/) to securely connect your customer's bank accounts with VIP Connect; launching
the VIP Connect SDK inside an `ASWebAuthenticationSession` (in private mode, as described above) is necessary to provide the best experience to your customers. 

## Returning to the app from VIP Connect

Upon completion or cancellation, VIP Connect will navigate to the address at the `returnURL` param passed during [session creation](https://developer.vippreferred.com/APIS/SDK/create-patron-session).
The sample app uses `closevip://done` as an example `returnURL`, which your app may use or modify as needed. 

## Running the sample app

The sample launches the VIP Connect flow as a demonstration. It uses fixed test data to create a real session on your operator, so it needs your
operator's values before it can run.

### What you need

- A Mac with Xcode, and an iPhone or an iOS simulator running iOS 16.4 or later.
- An Apple developer account, if you run on an iPhone.
- From [operator setup](https://developer.vippreferred.com/operator-onboarding/operator-setup): your operator's **JWT Secret** and **JWT Issuer**,
  and the name of the test environment your operator is in (such as `cert`). Please contact your Pavilion Payments representative if you need help
  obtaining these values.
- A test patron that exists for your operator (see [The test patron](#the-test-patron)).

### Steps

1. Clone this repository and open `VIPConnectDemo.xcodeproj` in Xcode.
2. Open `VIPConnectDemo/BuildEnvironmentVariables.plist` and fill in the three fields:

   | Field | Value |
   |---|---|
   | `Secret` | Your operator's JWT Secret, exactly as provided (a Base64 string) |
   | `Issuer` | Your operator's JWT Issuer |
   | `Environment` | The test environment name, such as `cert`. The app calls `https://<Environment>.api-gaming.paviliononline.io/sdk` |

   Do not commit this file with your values in it.
3. To run on an iPhone, select the `VIPConnectDemo` target, open **Signing & Capabilities**, and choose your own **Team**. If Xcode reports that the
   bundle identifier is not available, change it to one of your own. A simulator needs neither change.
4. If you are not using the built-in test patron, replace its values in `OperatorServer.swift` (see [The test patron](#the-test-patron)).
5. Choose the `iGamingDemo` scheme and your iPhone or simulator, then run the app.
6. On the home screen, tap **Launch VIP Connect SDK**. VIP Connect opens in an `ASWebAuthenticationSession` running in private (incognito)
   mode, so iOS does not ask for permission to sign in.
7. Complete or cancel the transaction in VIP Connect. When VIP Connect navigates to `closevip://done`, the session closes by itself and you are
   back in the sample app.

### If it doesn't start

| What you see | Cause |
|---|---|
| "Build Environment variables not set" | `Environment` is empty in `BuildEnvironmentVariables.plist` |
| "Operator token invalid" | The API rejected the request. Check `Secret`, `Issuer` and `Environment`, and that the test patron exists for your operator |
| The app closes when you tap the launch button | `Secret` is not a valid Base64 string |

### The test patron

The sample creates a session for one fixed, existing patron. The values are in `patronTransactionData` in `OperatorServer.swift`:

| Field | Value |
|---|---|
| `patronID` | `cb7c887d-6687-4aa5-a664-31cf6c810df7` |
| `vipCardNumber` | `7210806973` |
| `dateOfBirth` | `04/15/1980` |

The session is created with [`patronsession/existing`](https://developer.vippreferred.com/APIS/SDK/create-patron-session), so the patron must
already exist for your operator in the environment you chose. If this patron does not exist for your operator, replace the three values with a
patron of your own. The sample does not enroll new patrons.

The same function sets the rest of the request: a 10.00 deposit, a new `transactionID` for every session, `productType` `0` and the `returnURL`
`closevip://done`.

### Not for production

While this sample app creates its JWT token locally and accesses VIP Mobility APIs directly, this is NOT a recommended practice for production apps!
Your app should not have access to your operator secrets, and should use a more secure method of obtaining a session id such as getting it from a backend
service that holds the secret values and acts as a middle layer between the app and VIP Connect's APIs.
