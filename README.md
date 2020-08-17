# Onfido

[![Version](https://img.shields.io/cocoapods/v/Onfido.svg?style=flat)](http://cocoapods.org/pods/Onfido)
[![Carthage](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](http://cocoapods.org/pods/Onfido)
[![Build Status](https://app.bitrise.io/app/d04e3a422799521b/status.svg?token=vBI0wpdUSfh25wctd1MHfA&branch=master)](https://www.bitrise.io/app/d04e3a422799521b)
[![License](https://img.shields.io/cocoapods/l/Onfido.svg?style=flat)](http://cocoapods.org/pods/Onfido)
[![Platform](https://img.shields.io/cocoapods/p/Onfido.svg?style=flat)](http://cocoapods.org/pods/Onfido)

## Table of contents

*   [Overview](#overview)
*   [Getting started](#getting-started)
*   [Handling callbacks](#handling-callbacks)
    *   [Success handling](#success-handling)
    *   [Error handling](#run-exceptions)
*   [Customising SDK](#customising-sdk)
    *   [Flow customisation](#flow-customisation)
    *   [UI customisation](#ui-customisation)
    *   [Localisation](#localisation)
    *   [Language customisation](#language-customisation)
*   [Creating checks](#creating-checks)
*   [User Analytics](#user-analytics)
*   [Going live](#going-live)
*   [Migrating](#migrating)
*   [Security](#security)
*   [Accessibility](#accessibility)
*   [Licensing](#licensing)
*   [More information](#more-information)

## Overview

This SDK provides a drop-in set of screens and tools for iOS applications to allow capturing of identity documents and face photos/live videos for the purpose of identity verification with [Onfido](https://onfido.com/). The SDK offers a number of benefits to help you create the best on-boarding/identity verification experience for your customers:

-   Carefully designed UI to guide your customers through the entire photo/video-capturing process
-   Modular design to help you seamlessly integrate the photo/video-capturing process into your application flow
-   Advanced image quality detection technology to ensure the quality of the captured images meets the requirement of the Onfido identity verification process, guaranteeing the best success rate
-   Direct image upload to the Onfido service, to simplify integration\*

\*Note: the SDK is only responsible for capturing and uploading photos/videos. You still need to access the [Onfido API](https://documentation.onfido.com/) to create and manage checks.

### Enterprise Features

This SDK supports enterprise-only features such as:
-   Hide Onfido Logo

These features must be enabled on your account before they can be used, otherwise you will get runtime exceptions. Details for implementation can be found [here](#5.1-enterprise-features).

![Capture Document and face](assets/Overview.png)

## Getting started

* SDK supports iOS 10+
* SDK supports Xcode 11.0.0
* SDK has full bitcode support
* SDK supports following presentation styles:
  - Only full screen style for iPhones
  - Full screen and form sheet styles for iPads

### 1. Obtaining an API token

In order to start integration, you will need the **API token**. You can use our [sandbox](https://documentation.onfido.com/#sandbox-testing) environment to test your integration, and you will find these two sandbox tokens inside your [Onfido Dashboard](https://onfido.com/dashboard/api/tokens). You can create sandbox tokens inside your Onfido Dashboard.

#### 1.1 Regions

Onfido offers region-specific environments. Refer to the [Regions](https://documentation.onfido.com/#regions) section in the API documentation for token format and API base URL information.

### 2. Creating an Applicant

You must create an Onfido [applicant](https://documentation.onfido.com/#applicants) before you start the flow.

For a document or face check the minimum applicant details required are `firstName` and `lastName`.

You must create applicants from your server:

```shell
$ curl https://api.onfido.com/v3/applicants \
    -H 'Authorization: Token token=YOUR_API_TOKEN' \
    -d 'first_name=Theresa' \
    -d 'last_name=May'
```

The JSON response has an `id` field containing a UUID that identifies the applicant. You will pass the applicant ID to the SDK and all documents or live photos/videos uploaded by that instance of the SDK will be associated with that applicant.

**Note**: If you're using API v2, please check out [API v2 to v3 migration guide](https://developers.onfido.com/guide/v2-to-v3-migration-guide#applicant-creation) to understand which changes need to be applied before starting to use API v3.

### 3. Configuring SDK with Tokens

We now support two token mechanisms:

`SDK token`   
`Mobile token`

We strongly recommend using a SDK token. It provides a more secure means of integration, as the token is temporary and applicant id-bound. Note that, if you're using SDK tokens, you shouldn't call withApplicantId function.

#### 3.1 SDK Tokens

You will need to generate and include a short-lived JSON Web Token (JWT) every time you initialise the SDK. To generate an SDK Token you should perform a request to the SDK Token endpoint in the Onfido API:

To generate an SDK Token you should perform a request to the SDK Token endpoint in the Onfido API:

```
$ curl https://api.onfido.com/v3/sdk_token \
  -H 'Authorization: Token token=YOUR_API_TOKEN' \
  -F 'applicant_id=YOUR_APPLICANT_ID' \
  -F 'application_id=YOUR_APPLICATION_BUNDLE_IDENTIFIER'
```

Make a note of the token value in the response, as you will need it later on when initialising the SDK.

**Warning:** SDK tokens expire 90 minutes after creation. So SDK token configurator function has an optional parameter called `expireHandler` which can be used to generate and pass SDK token when it expires. By this means, with using this parameter you can ensure that SDK will continue its flow even after SDK token has expired.

**Note**: If you're using API v2, please check out [API v2 to v3 migration guide](https://developers.onfido.com/guide/v2-to-v3-migration-guide) to understand which changes need to be applied before starting to use API v3.

##### Example Usage

##### Swift

```swift
func getSDKToken(_ completion: @escaping (String) -> Void) {
    <Your network request logic to retrieve SDK token goes here>
    completion(myNewSDKtoken)
}

let config = try! OnfidoConfig.builder()
    .withSDKToken("YOUR_SDK_TOKEN_HERE", expireHandler: getSDKToken)


```

##### Objective C

```Objective-C

-(void) getSDKToken: (void(^)(NSString *)) handler {
  <Your network request logic to retrieve SDK token goes here>
   handler(sdkToken);
}

ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];
[configBuilder withSdkToken:@"YOUR_SDK_TOKEN_HERE" expireHandler:^(void (^ handler)(NSString *  expireHandler)) {
        [self getSDKToken:handler];
}];
```

#### 3.2 Mobile Tokens

**Note:**  Mobile token usage is still supported, but it **will be deprecated** in the future. If you are starting a project, we would strongly recommend that you use SDK tokens instead.

In order to start integration, you will need the **API token** and the **mobile token**. You can use our [sandbox](https://documentation.onfido.com/#sandbox-testing) environment to test your integration, and you will find these two sandbox tokens inside your [Onfido Dashboard](https://onfido.com/dashboard/api/tokens).

**Warning:** You **MUST** use the **mobile token** and not the **API token** when configuring the SDK itself.

##### Example Usage

##### Swift

```swift
let config = try! OnfidoConfig.builder()
    .withToken("YOUR_MOBILE_TOKEN_HERE")
    .withApplicantId("APPLICANT_ID_HERE")
```

##### Objective C

```Objective-c
ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];
[configBuilder withToken:@"YOUR_MOBILE_TOKEN_HERE"];
[configBuilder withApplicantId:@"APPLICANT_ID_HERE"];
```


### 4. App permissions

The Onfido SDK makes use of the device Camera. You will be required to have the `NSCameraUsageDescription` and `NSMicrophoneUsageDescription` keys in your application's `Info.plist` file:

```xml
<key>NSCameraUsageDescription</key>
<string>Required for document and facial capture</string>
<key>NSMicrophoneUsageDescription</key>
<string>Required for video capture</string>
```
**Note**: Both keys will be required for app submission.

### 5. Adding the SDK dependency

#### Using Cocoapods

The SDK is available on Cocoapods and you can include it in your projects by adding the following to your Podfile:

```ruby
pod 'Onfido'
```

Run `pod install` to get the sdk.

#### Using Carthage

The SDK is available on Carthage and you can include it in your projects by adding the following to your Cartfile:

```ruby
binary "https://raw.githubusercontent.com/onfido/onfido-ios-sdk/master/onfido-carthage-spec.json"
```

Run `carthage update` to get the sdk.

#### Manual Installation

The SDK is available in [Github Releases tab](https://github.com/onfido/onfido-ios-sdk/releases) where you can download the compressed framework, you can find the latest release [here](https://github.com/onfido/onfido-ios-sdk/releases/latest).

1. [Download](https://github.com/onfido/onfido-ios-sdk/releases/latest) the compressed debug zip file containing the `Onfido.framework`.
2. Uncompress the zip file and then move the `Onfido.framework` artefact into your project.
3. Add `Onfido.framework` located within your project to the `Embedded binaries` section in the `General` tab of you iOS app target.
4. Open your app's project file in Xcode. Then select your app's target under target list. Next select `Build Phases` tab and under `Embed Frameworks` step add a new `Run Script Phase`. Name it `Onfido Framework Archive`. In the text area add the following code:
```bash
if [[ "$ACTION" != "install" ]]; then
exit 0;
fi

FRAMEWORK_DIR="${CONFIGURATION_BUILD_DIR}/${FRAMEWORKS_FOLDER_PATH}"
ONFIDO_FRAMEWORK="${FRAMEWORK_DIR}/Onfido.framework"

cd "${ONFIDO_FRAMEWORK}"

lipo -remove i386 Onfido -o Onfido
lipo -remove x86_64 Onfido -o Onfido
```

#### Non-Swift apps

If your app is not Swift based then you must create a new Swift file inside of your project with the following contents:
```
/*
 This file is required to force Xcode to package Swift runtime libraries required for
 the Onfido iOS SDK to run
 */
import Foundation
import AVFoundation
import CoreImage
import UIKit
import Vision

func fixLibSwiftOnoneSupport() {
    // from https://stackoverflow.com/a/54511127/2982993
    print("Fixes dyld: Library not loaded: @rpath/libswiftSwiftOnoneSupport.dylib")
}
```
Additionally you must also set `Always Embed Swift Standard Libraries` to `Yes` in your project configuration.

The above code and configuration will force Xcode to package the required Swift runtime libraries required by the Onfido SDK to run.

### 5. Creating the SDK configuration

Once you have an added the SDK as a dependency and you have an applicant ID, you can configure the SDK:

#### Swift

```swift
let config = try! OnfidoConfig.builder()
    .withSDKToken("YOUR_SDK_TOKEN_HERE")
    .withWelcomeStep()
    .withDocumentStep()
    .withFaceStep(ofVariant: .photo(withConfiguration: nil))
    .build()

let onfidoFlow = OnfidoFlow(withConfiguration: config)
    .with(responseHandler: { results in
        // Callback when flow ends
    })
```

#### Objective-C

```Objective-C
ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];


[configBuilder withSdkToken:@"YOUR_SDK_TOKEN_HERE"];
[configBuilder withWelcomeStep];
[configBuilder withDocumentStep];

NSError *variantConfigError = NULL;
Builder *variantBuilder = [ONFaceStepVariantConfig builder];
[variantBuilder withPhotoCaptureWithConfig: NULL];
[configBuilder withFaceStepOfVariant: [variantBuilder buildAndReturnError: &variantConfigError]];

if (variantConfigError == NULL) {
  NSError *configError = NULL;
  ONFlowConfig *config = [configBuilder buildAndReturnError:&configError];

  if (configError == NULL) {
      ONFlow *onFlow = [[ONFlow alloc] initWithFlowConfiguration:config];
      [onFlow withResponseHandler:^(ONFlowResponse *response) {
          // Callback when flow ends
      }];
  }
}
```

### 6. Starting the flow

#### Swift

```swift
let onfidoRun = try! onfidoFlow.run()

self.present(onfidoRun, animated: true, completion: nil) //`self` should be your view controller
```

#### Objective-C

```Objective-C
NSError *runError = NULL;
UIViewController *onfidoController = [onFlow runAndReturnError:&runError];

if (runError == NULL) {
    [self presentViewController:onfidoController animated:YES completion:NULL];
}
```

Congratulations! You have successfully started the flow. Carry on reading the next sections to learn how to:

-   Handle callbacks
-   Customise the SDK
-   Create checks

## Handling callbacks

To receive the result from the flow, you should pass a callback to the instance of `OnfidoFlow` (`ONFlow` for Objective-C). Typically, on success, you would [create a check](#creating-checks) on your backend server.

The result object passed to the callback may include the following attributes for Swift: `.success([OnfidoResult])`, `.error(Error)` and `.cancel`. For Objective-C based interface an instance of `ONFlowResponse` is passed back to the callback with three properties: `results`, `error` and `userCanceled`. When `userCanceled` is false then `results` or `error` properties will be set.

#### Swift

```swift
let responseHandler: (OnfidoResponse) -> Void = { response in
  switch response {
    case let .error(error):
        // Some error happened
    case let .success(results):
        // User completed the flow
        // You can create your check here
    case .cancel:
        // Flow cancelled by the user
  }
}
```

#### Objective-C

```Objective-C
(^responseHandlerBlock)(ONFlowResponse *response) {

    if (response.userCanceled) {
        // Flow cancelled by the user
    } else if (response.results) {
        // User completed the flow
        // You can create your check here
    } else if (response.error) {
        // Some error happened
    }
}
```

### Success handling

Success is when the user has reached the end of the flow.

#### Swift

`[OnfidoResult]` is a list with multiple results. The results are different enum values, each with its own associated value (also known as payload). This enum, `OnfidoResult`, can have the following values:

1.  `OnfidoResult.document` and `OnfidoResult.face`: Its payload is relevant in case you want to manipulate or preview the captures in someway.

Keep reading to find out how to extract the payload of each `OnfidoResult` enum value.

#### Objective-C

`[ONFlowResult]` is a list with multiple results. The result is an instance of `ONFlowResult` containing two properties: `type`, which is an enum with values `ONFlowResultTypeDocument`, `ONFlowResultTypeFace`, and `result`, which instance type can be of `ONDocumentResult` or `ONFaceResult`. The result type can be derived by the `type` property.

#### Capture result payload

Under normal circumstances, you would not need to inspect the results of the captures themselves, as the SDK handles file uploads for you.
However, if you want to see information regarding the document and face captures, you can access the result object as follows:

##### Swift

```swift
let document: Optional<OnfidoResult> = results.filter({ result in
  if case OnfidoResult.document = result { return true }
  return false
}).first

if let documentUnwrapped = document, case OnfidoResult.document(let documentResponse) = documentUnwrapped {

  /* documentResponse
  Onfido API response to the upload of the document
  More details: https://documentation.onfido.com/#upload-document
  */
  print(documentResponse.id)

  // use documentResponse.href to fetch the captured image if required
}
```

Face follows a similar structure to document, but the `case` is `OnfidoResult.face` instead of `OnfidoResult.document`.

##### Objective-C

```Objective-C

NSPredicate *documentResultPredicate = [NSPredicate predicateWithBlock:^BOOL(id flowResult, NSDictionary *bindings) {

    if (((ONFlowResult *)flowResult).type == ONFlowResultTypeDocument) {
        return YES;
    } else {
        return NO;
    }
}];
NSArray *flowWithDocumentResults = [results filteredArrayUsingPredicate:documentResultPredicate];

if (flowWithDocumentResults.count > 0) {

    /* documentResponse
    Onfido API response to the upload of the document
    More details: https://documentation.onfido.com/#upload-document
    */
    ONDocumentResult *documentResult = ((ONFlowResult *)flowWithDocumentResults[0]).result;
    NSLog(@"%@", documentResult.id);

    // use documentResponse.href to fetch the captured image if required
}
```

Face follows a similar structure to document, change the type `ONFlowResultTypeDocument` for `ONFlowResultTypeFace`.

### Error handling

#### Response Handler Errors

##### Swift

The `Error` object returned, as part of `OnfidoResponse.error(Error)`, is of type `OnfidoFlowError`. It's an enum with multiple cases depending on the error type.

```swift
switch response {
  case let OnfidoResponse.error(error):
    switch error {
      case OnfidoFlowError.cameraPermission:
        // It happens if the user denies permission to the sdk during the flow
      case OnfidoFlowError.failedToWriteToDisk:
        // It happens when the SDK tries to save capture to disk, maybe due to a lack of space
      case OnfidoFlowError.microphonePermission:
        // It happens when the user denies permission for microphone usage by the app during the flow
      case OnfidoFlowError.upload(let OnfidoApiError):
        // It happens when the SDK receives an error from a API call see [https://documentation.onfido.com/#errors](https://documentation.onfido.com/#errors) for more information
      case OnfidoFlowError.exception(withError: let error, withMessage: let message):
        // It happens when an unexpected error occurs, please contact [ios-sdk@onfido.com](mailto:ios-sdk@onfido.com?Subject=ISSUE%3A) when this happens
      default: // necessary because swift
    }
}
```

Note: Not all cases part of `OnfidoFlowError` will be passed to `OnfidoResponse.error`, there is one case that error will be returned as an exception, see [Run Exceptions](#run-exceptions) and [Configuration errors](#configuration-errors).

##### Objective-C

The `error` property of the `ONFlowResponse` returned to the callback block is of type `NSError`. You can easily identify the error by comparing the `code` property of the `NSError` instance with `ONFlowError`, i.e. `response.code == ONFlowErrorCameraPermission`. You could also find out more about the error by printing or logging the `userInfo` property of the `NSError` instance.  The `NSError` contained within the `ONFlowResponse`'s `error` property can be handled such as:

```Objective-C
switch (error.code) {
    case ONFlowErrorCameraPermission:
        // It happens if the user denies permission to the sdk during the flow
        break;
    case ONFlowErrorFailedToWriteToDisk:
        // It happens when the SDK tries to save capture to disk, maybe due to a lack of space
        break;
    case ONFlowErrorMicrophonePermission:
        // It happens when the user denies permission for microphone usage by the app during the flow
        break;
    case ONFlowErrorUpload:
        // It happens when the SDK receives an error from a API call see [https://documentation.onfido.com/#errors](https://documentation.onfido.com/#errors) for more information
        // you can find out more by printing or logging userInfo from error
        break;
    case ONFlowErrorException:
        // It happens when an unexpected error occurs, please contact [ios-sdk@onfido.com](mailto:ios-sdk@onfido.com?Subject=ISSUE%3A) when this happens
        break;
}
```

Note: Not all cases part of `ONFlowError` will be passed to response handler block, there is one case that error will be returned as an exception, see [Run Exceptions](#run-exceptions) and [Configuration errors](#configuration-errors).

#### Run exceptions

When initiating the SDK there can be an exception.

##### Swift

You can handle run exceptions in Swift with a `do/catch` as shown below:

```swift
do {
  let onfidoRun = try self.onfidoFlow!.run()
  self.present(onfidoRun, animated: true, completion: nil)
}
catch let error {
  switch error {
    case OnfidoFlowError.cameraPermission:
      // do something about it here
    case OnfidoFlowError.microphonePermission:
      // do something about it here
    default:
      // should not happen, so if it does, log it and let us know
  }
}
```

##### Objective-C

You can handle run exceptions in Objective-C as shown below:

```Objective-C
NSError *runError = NULL;
UIViewController *onfidoController = [onFlow runAndReturnError:&runError];

if (runError) {
    switch (runError.code) {
        case ONFlowErrorCameraPermission:
            // do something about it here
            break;
        case ONFlowErrorMicrophonePermission:
            // do something about it here
            break;
        default:
            // do something about it here
            break;
    }
} else {
    [self presentViewController:onfidoController animated:YES completion:NULL];
}
```

#### Configuration errors

The following are required when configuring the Onfido iOS SDK:

- Mobile token
- Applicant
- At least one capture step

Otherwise you may encounter the following errors when calling the `build()` function on the OnfidoConfig.Builder (`ONFlowConfigBuilder` in Objective-C) instance:

- `OnfidoConfigError.missingToken` (`ONFlowConfigErrorMissingSteps` in Objective-C), when no or empty string token is provided
- `OnfidoConfigError.missingApplicant` (`ONFlowConfigErrorMissingApplicant` in Objective-C), when no applicant instance is provided
- `OnfidoConfigError.missingSteps` (`ONFlowConfigErrorMissingSteps` in Objective-C), when no step is provided
- `OnfidoConfigError.multipleTokenTypes` (`ONFlowConfigErrorMultipleTokenTypes` in Objective-C), when both an SDK Token and a Mobile Tokens are provided
- `OnfidoConfigError.applicantProvidedWithSDKToken` (`ONFlowConfigErrorApplicantProvidedWithSDKToken` in Objective-C), when both an SDK Token and an applicant provided
- `OnfidoConfigError.invalidDocumentFormatAndCountryCombination` (`ONFlowConfigErrorInvalidDocumentFormatAndCountryCombination` in Objective-C), when unsupported document format for the specified country provided. See [Document Type Configuration](#document-type-configuration) section to check supported combinations.

## Customising SDK

### Flow customisation

The SDK can be customised by specifying to show a welcome screen and the steps to capture when configuring.

You can show the welcome screen by calling `configBuilder.withWelcomeStep()` in Swift or `[configBuilder withWelcomeStep]` in Objective-C.

#### Swift

```swift
let config = try! OnfidoConfig.builder()
    .withWelcomeStep()
    ...
    .build()
```

#### Objective-C

```Objective-C
ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];
[configBuilder withSdkToken:@"YOUR_SDK_TOKEN_HERE"];
...
[configBuilder withWelcomeStep];

NSError *configError = NULL;
ONFlowConfig *config = [configBuilder buildAndReturnError:&configError];

if (configError) {
    // Handle config build error
} else {
    // use config
}
```

You can either specify to capture the document and/or face of the user.

The face step has two variants for Swift interface:

- `FaceStepVariant.photo(with: PhotoStepConfiguration?)`
- `FaceStepVariant.video(with: VideoStepConfiguration?)`

For Objective-C interface, you should use ONFaceStepVariantConfig as below.

To configure with video variant:

```
NSError * error;
Builder * variantBuilder = [ONFaceStepVariantConfig builder];
[variantBuilder withVideoCaptureWithConfig:
 [[VideoStepConfiguration alloc] initWithShowIntroVideo: YES]];
[configBuilder withFaceStepOfVariant: [variantBuilder buildAndReturnError: &error]];
```

To configure with photo variant:

```
NSError * error;
Builder * variantBuilder = [ONFaceStepVariantConfig builder];
[variantBuilder withPhotoCaptureWithConfig: NULL];
[configBuilder withFaceStepOfVariant: [variantBuilder buildAndReturnError: &error]];
```



#### Swift

```swift
let config = try! OnfidoConfig.builder()
    .withSDKToken("YOUR_SDK_TOKEN_HERE")
    .withWelcomeStep()
    .withDocumentStep()
    .withFaceStep(ofVariant: .photo(withConfiguration: nil))  // specify the face capture variant here
    .build()
```

#### Objective-C

```Objective-C
ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];

[configBuilder withSdkToken:@"YOUR_SDK_TOKEN_HERE"];
[configBuilder withWelcomeStep];
[configBuilder withDocumentStep];
NSError *variantError = NULL;
Builder * variantBuilder = [ONFaceStepVariantConfig builder];
[variantBuilder withVideoCaptureWithConfig: [[VideoStepConfiguration alloc] initWithShowIntroVideo: YES]];
[configBuilder withFaceStepOfVariant: [variantBuilder buildAndReturnError: &variantError]];

if (variantError) {
  // Handle variant config error
} else {
  NSError *configError = NULL;
  ONFlowConfig *config = [configBuilder buildAndReturnError:&configError];

  if (configError) {
      // Handle config build error
  } else {
      // use config
  }
}
```

The document step can be further configured to capture single document types with specific configuration. The document types supported are:

| Document Type        | Configuration Class           | Configurable Properties    |
|----------------------|-------------------------------|----------------------------|
| passport             | PassportConfiguration         |                            |
| drivingLicence       | DrivingLicenceConfiguration   | - country<br> - documentFormat |
| nationalIdentityCard | NationalIdentityConfiguration | - country<br> - documentFormat |
| residencePermit      | ResidencePermitConfiguration  | - country                  |
| visa                 | VisaConfiguration             | - country                  |
| workPermit           | WorkPermitConfiguration       | - country                  |
| generic              | GenericDocumentConfiguration  | - country                  |

**Note**: `Generic` document type doesn't offer an optimised capture experience for a desired document type.

### Document Type Configuration

As you can see in the table above, each document type has it's own configuration class. While configuring document type, you can optionally pass configuration object along with type.

#### Configuring Country

**Note**: You can specify country for all document types except **Passport**.       

Please refer to the [ISO 3166-1 alpha-3]("https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3") 3 letter country codes to find out what you need to pass as
country code to the SDK.

Let's say that you would like to capture only driving licenses from the United Kingdom. The following code shows how to do this:

##### Swift

```swift
let config = try! OnfidoConfig.builder()
    .withSDKToken("YOUR_SDK_TOKEN_HERE")
    .withDocumentStep(ofType: .drivingLicence(config: DrivingLicenceConfiguration(country: "GBR")))
    .build()
```

##### Objective-C

```Objective-C
ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];

[configBuilder withSdkToken:@"YOUR_SDK_TOKEN_HERE"];
NSError *documentVariantError = NULL;
DocumentConfigBuilder * documentVariantBuilder = [ONDocumentTypeVariantConfig builder];
[documentVariantBuilder withDrivingLicenceWithConfig:[[DrivingLicenceConfiguration alloc] initWithCountry: @"GBR"]];
ONDocumentTypeVariantConfig *documentStepVariant = [variantBuilder buildAndReturnError: documentVariantError];

if (documentVariantError) {
  // Handle variant config error
} else {
  NSError *configError = NULL;
  ONFlowConfig *config = [configBuilder buildAndReturnError:&configError];
}

```


#### Configuring Document Format

Some documents are not in card format. SDK provides an optimised experience for some folded documents, such as the Italian national identity or French driving licence.

Typically the user picks the format for these, but you can specify a specific format by configuring the Onfido SDK.

`Card` is the default document format value for all document types.   

Please check the table below to see supported document formats for each document type.      

**Note:** If you configure the SDK with unsupported document type/option combination, SDK will throw an `OnfidoConfigError.invalidDocumentFormatAndCountryCombination` (`ONFlowConfigErrorInvalidDocumentFormatAndCountryCombination` in Objective-C)  error during runtime.

| Document Type/ Document Format | card          |folded                           |
|--------------------------|---------------|------------------------|
| drivingLicence           | ALL COUNTRIES | Only France (Country code "FRA") |
| nationalIdentityCard     | ALL COUNTRIES | Only Italy (Country code "ITA")  |

| Document Type/ Document Format |           |
|--------------------------|---------------|
| passport                 | NOT CONFIGURABLE |
| residencePermit          | NOT CONFIGURABLE |
| visa                     | NOT CONFIGURABLE |
| workPermit               | NOT CONFIGURABLE |
| generic                  | NOT CONFIGURABLE |


Let's say that you would like to capture a folded national identity document from Italy. The following code shows how to do this:

##### Swift

```swift
let config = try! OnfidoConfig.builder()
    .withSDKToken("YOUR_SDK_TOKEN_HERE")
    configBuilder.withDocumentStep(ofType: .nationalIdentityCard(config: NationalIdentityConfiguration(documentFormat: .folded, country: "ITA"))
    .build()
```

##### Objective-C

```Objective-C
ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];

[configBuilder withSdkToken:@"YOUR_SDK_TOKEN_HERE"];
NSError *documentVariantError = NULL;
DocumentConfigBuilder * documentVariantBuilder = [ONDocumentTypeVariantConfig builder];
[documentVariantBuilder withNationalIdentityCardWithConfig:[[NationalIdentityConfiguration alloc] initWithDocumentFormat:DocumentFormatFolded country: @"ITA"]];

ONDocumentTypeVariantConfig *documentStepVariant = [variantBuilder buildAndReturnError: documentVariantError];

if (documentVariantError) {
  // Handle variant config error
} else {
  NSError *configError = NULL;
  ONFlowConfig *config = [configBuilder buildAndReturnError:&configError];
}

```

### UI customisation

In order to enhance the user experience on the transition between your application and the SDK, you can customise some of the colors and fonts used in the SDK flow.

To customise:

#### Swift
```Swift
let appearance = Appearance(
              primaryColor: <DESIRED_UI_COLOR_HERE>,
              primaryTitleColor: <DESIRED_UI_COLOR_HERE>,
              primaryBackgroundPressedColor: <DESIRED_UI_COLOR_HERE>,
              secondaryBackgroundPressedColor: <DESIRED_UI_COLOR_HERE>,
              fontRegular: <DESIRED_FONT_NAME_HERE>,
              fontBold: <DESIRED_FONT_NAME_HERE>),
              supportDarkMode: <true | false>))

let configBuilder = OnfidoConfig.builder()
configBuilder.withAppearance(appearance)
```

#### Objective-C
```Objective-C
ONAppearance *appearance = [[ONAppearance alloc]
                                initWithPrimaryColor:<DESIRED_UI_COLOR_HERE>
                                primaryTitleColor:<DESIRED_UI_COLOR_HERE>
                                primaryBackgroundPressedColor:<DESIRED_UI_COLOR_HERE>
                                secondaryBackgroundPressedColor:<DESIRED_UI_COLOR_HERE>
                                fontRegular: <DESIRED_FONT_NAME_HERE>
                                fontBold: <DESIRED_FONT_NAME_HERE>
                                supportDarkMode: <true | false>>]];

ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];
[configBuilder withAppearance:appearance];
```

`primaryColor`: Defines the background color of views such as document type icon and capture confirmation buttons and back navigation button.   
`primaryTitleColor`: Defines the text color of labels included in views such as capture confirmation buttons.   
`primaryBackgroundPressedColor`: Defines the background color of capture confirmation buttons when pressed.   
`secondaryBackgroundPressedColor`: Defines the background color of capture cancel buttons when pressed.   
`fontRegular`: Defines the custom font name for the regular style labels.   
`fontBold`: Defines the custom font name for the bold style labels.   
`supportDarkMode`: Defines if iOS Dark Mode will be supported on SDK screens. The value is true by default. **This property applicable only for Xcode 11 built apps and has effect for the users whose device is running on iOS 13 and above.**

#### Dark Mode only UI customisation

If you just need to change supportDarkMode value, you can use initialiser below:   

##### Swift

```Swift
let appearance = Appearance(supportDarkMode: <true|false>)
let configBuilder = OnfidoConfig.builder()
configBuilder.withAppearance(appearance)
```

##### Objective-C

```Objective-C
ONAppearance *appearance = [[ONAppearance alloc] initWithSupportDarkMode:<true|false>];

```

### Localisation

Onfido iOS SDK already comes with out-of-the-box translations for the following locales:

 - English (en) 🇬🇧
 - Spanish (es) 🇪🇸
 - French (fr) 🇫🇷
 - German (de) 🇩🇪

In case you would like us to add translations for some other locales we don't provide yet, please contact us through [ios-sdk@onfido.com](mailto:ios-sdk@onfido.com?Subject=ISSUE%3A).

### Language customisation

**Note**:
- If the strings translations change it will result in a MINOR version change, therefore you are responsible for testing your translated layout in case you are using this feature. If you want a language translated you can also get in touch with us at [ios-sdk@onfido.com](mailto:ios-sdk@onfido.com).
- When adding custom translations, please make sure you add the whole set of keys we have on `Localizable.strings` file. In particular, `onfido_locale`, which identifies the current locale being added, must be included. The value for this string should be the ISO 639-1 2-letter language code corresponding to the translation being added.

Examples:
- When strings file added for Russian language, the `onfido_locale` key should have `ru` as its value.
- When strings file added for American English language (en-US), the `onfido_locale` key should have `en` as its value.

Without this string correctly translated, we won't be able to determine which language the user is likely to use when doing the video liveness challenge. It may result in our inability to correctly process the video, and the check may fail.

The strings used within the SDK can be customised by having a `Localizable.strings` in your app for the desired language and by configuring the flow using `withCustomLocalization()` method on the configuration builder. i.e.

#### Swift

```swift
let config = try! OnfidoConfig.builder()
    .withSDKToken("YOUR_SDK_TOKEN_HERE")
    .withWelcomeStep()
    .withDocumentStep(ofType: .drivingLicence, andCountryCode: "GBR")
    .withFaceStep(ofVariant: .photo(withConfiguration: nil))
    .withCustomLocalization() // will look for localizable strings in your Localizable.strings file
    .build()
```

#### Objective-C

```Objective-C
ONFlowConfigBuilder *configBuilder = [ONFlowConfig builder];

[configBuilder withSdkToken:@"YOUR_SDK_TOKEN_HERE"];
[configBuilder withWelcomeStep];
[configBuilder withDocumentStepOfType:ONDocumentTypeDrivingLicence andCountryCode:@"GBR"];
NSError *variantError = NULL;
Builder * variantBuilder = [ONFaceStepVariantConfig builder];
[variantBuilder withPhotoCaptureWithConfig: NULL];
[configBuilder withFaceStepOfVariant: [variantBuilder buildAndReturnError: &variantError]];

if (variantError) {
  // Handle variant config error
} else {
  [configBuilder withCustomLocalization]; // will look for localizable strings in your Localizable.strings file
  NSError *configError = NULL;
  ONFlowConfig *config = [configBuilder buildAndReturnError:&configError];
}

```

You can find the keys for the localizable strings under the example [`Localizable.strings`](Localizable.strings) file in this repo. You can supply partial translations, meaning if you don’t include a translation to particular key our translation will be used instead. You can also name the strings file with the translated keys as you desire but the name of the file will have to be provided to the SDK as a parameter to the `withCustomLocalization()` method i.e. `withCustomLocalization(andTableName: "MY_CUSTOM_STRINGS_FILE")` (`[configBuilder withCustomLocalizationWithTableName:@"MY_CUSTOM_STRINGS_FILE"];` for Objective-C). Addtionally you can specify the bundle from which to read the strings file i.e `withCustomLocalization(andTableName: "MY_CUSTOM_STRINGS_FILE", in: myBundle)` (`[configBuilder withCustomLocalizationWithTableName:@"MY_CUSTOM_STRINGS_FILE" in: myBundle];` for Objective-C).

## Creating checks

As the SDK is only responsible for capturing and uploading photos/videos, you would need to start a check on your backend server using the [Onfido API](https://documentation.onfido.com/).

### 1. Obtaining an API token

All API requests must be made with an API token included in the request headers. You can find your API token (not to be mistaken with the mobile token) inside your [Onfido Dashboard](https://onfido.com/dashboard/api/tokens).

Refer to the [Authentication](https://documentation.onfido.com/#authentication) section in the API documentation for details. For testing, you should be using the sandbox, and not the live, token.

### 2. Creating a check

You will need to create a check by making a request to the [create check endpoint](https://documentation.onfido.com/#create-check), using the applicant id. If you are just verifying a document, you only have to include a [document report](https://documentation.onfido.com/#document-report) as part of the check. On the other hand, if you are verifying a document and a face photo/live video, you will also have to include a [facial similarity report](https://documentation.onfido.com/#facial-similarity-report) with the corresponding values: `facial_similarity_photo` for the photo option and `facial_similarity_video` for the video option.

```shell
$ curl https://api.onfido.com/v3/checks \
    -H 'Authorization: Token token=YOUR_API_TOKEN' \
    -d 'applicant_id=YOUR_APPLICANT_ID' \
    -d 'report_names=[document,facial_similarity_photo]'
```

Note: you can also submit the POST request in JSON format.

You will receive a response containing the check id instantly. As document and facial similarity reports do not always return actual [results](https://documentation.onfido.com/#results) straightaway, you need to set up a webhook to get notified when the results are ready.

Finally, as you are testing with the sandbox token, please be aware that the results are pre-determined. You can learn more about sandbox responses [here](https://documentation.onfido.com/#pre-determined-responses).

**Note**: If you're using API v2, please check out [API v2 to v3 migration guide](https://developers.onfido.com/guide/v2-to-v3-migration-guide#checks-in-api-v3) to understand which changes need to be applied before starting to use API v3.

### 3. Setting up webhooks

Refer to the [Webhooks](https://documentation.onfido.com/#webhooks) section in the API documentation for details.

## User Analytics

The SDK allows you to track the user's journey through the verification process via an definable hook. This is meant to give some insight into how your user's make use of the SDK screens.

### Overriding the hook

In order to expose the user's progress through the SDK an hook method must be defined while creating the `OnfidoFlow.swift` instance using a `.with(eventHandler: EventHandler)` call. This might look something like the following:

```swift
OnfidoFlow(withConfiguration: config)
    .with(eventHandler: {
        (event: Event) -> () in
        // Your code here
    })
```

The code inside of the defined method will now be called when a particular event is triggered, usually when the user reaches a new screen. For a full list of events see the bottom of this section.

The parameter being passed in is an `OnfidoFlow.Event` struct which contains the following:
- `eventName`: A `String` indicating the type of event. Currently will always this return as `"Screen"` as each tracked event is a user visiting a screen. In the future more event types may become available for tracking.
- `properties`: A `Dictionary` object containing the specific details of an event. This will contain things such as the `name` of the screen visited.

### Using the data

Currently we recommend using the above hook to keep track of how many user's reach each screen in your flow. This can be done by storing the count of users that reach each screen and comparing them to the amount of user's who've made it to the `Welcome` screen.

### Tracked events

Below is the list of potential events currently being tracked by the hook:
```
WELCOME - User reached the "Welcome" screen
DOCUMENT_CAPTURE - User reached the "document capture" screen (for one-sided document)
DOCUMENT_CAPTURE_FRONT - User reached the "document capture" screen for the front side (for two-sided document)
DOCUMENT_CAPTURE_BACK - User reached the "document capture" screen for the back side (for two-sided document)
DOCUMENT_CAPTURE_CONFIRMATION - User reached the "document confirmation" screen (for one-sided document)
DOCUMENT_CAPTURE_CONFIRMATION_FRONT - User reached the "document confirmation" screen for the front side (for two-sided document)
DOCUMENT_CAPTURE_CONFIRMATION_BACK - User reached the "document confirmation" screen for the back side (for two-sided document)
DOCUMENT_UPLOAD - User's document is uploading
FACIAL_INTRO - User reached the "selfie intro" screen
FACIAL_CAPTURE - User reached the "selfie capture" screen
FACIAL_CAPTURE_CONFIRMATION - User reached the "selfie confirmation" screen
FACIAL_UPLOAD - User's selfie is uploading
VIDEO_FACIAL_INTRO - User reached the "liveness intro" screen
VIDEO_FACIAL_CAPTURE - User reached the "liveness video capture" screen
VIDEO_FACIAL_CAPTURE_STEP_1 - User reached the 1st challenge during "liveness video capture", challenge_type can be found in eventProperties
VIDEO_FACIAL_CAPTURE_STEP_2 - User reached the 1st challenge during "liveness video capture", challenge_type can be found in eventProperties
VIDEO_FACIAL_CAPTURE_CONFIRMATION - User reached the "liveness video confirmation" screen
VIDEO_FACIAL_UPLOAD - User's liveness video is uploading
```

## Going live

Once you are happy with your integration and are ready to go live, please contact [client-support@onfido.com](mailto:client-support@onfido.com) to obtain live versions of the API token and the mobile token. You will have to replace the sandbox tokens in your code with the live tokens.

A few things to check before you go live:

- Make sure you have set up [webhooks](https://documentation.onfido.com/#webhooks) to receive live events
- Make sure you have entered correct billing details inside your [Onfido Dashboard](https://onfido.com/dashboard/)

### Size Impact

| User iOS Version | SDK Size Impact (MB)              |
|------------------|-----------------------------------|
| 12.2 and above   | 3.69|
| Below 12.2       | up to 3.69* or up to 12.691**|


**\*** If the application is in Swift but doesn't include any Swift libraries that Onfido iOS SDK requires  
**\*\*** If the application doesn't include any Swift code, i.e. written completely in Objective-C, and Onfido iOS SDK is the only
 Swift library that application integrates with

**Note**: These calculations was performed based on a single application architecture

## Migrating

You can find the migration guide at [MIGRATION.md](MIGRATION.md) file

## Security

This section is dedicated to every security aspect of the SDK

### Certificate Pinning

**Note**: Certificate pinning works only on devices running on iOS 10.3 or above.

We provide integrators the ability to pin any communications between our SDK and server, through a .withCertificatePinning() method in our OnfidoConfig.Builder configuration builder. This method accepts as parameter an CertificatePinningConfiguration with sha-256 hashes of certificate's public keys.
In case you are interested in using this feature, for more information about the hashes, please reach out to us at ios-sdk@onfido.com.

#### Swift

```swift
    let config = try! OnfidoConfig.builder()
    ...
    do {
      config.withCertificatePinning(try CertificatePinningConfiguration(hashes: ["EXAMPLE_HASH"]))
    } catch {
      // handle CertificatePinningConfiguration initialisation failures. i.e Providing empty array causes initialiser to be failed.
    }
    ...
    configBuilder.build()
```

#### Objective-C

```Objective-C
    ONFlowConfigBuilder * builder = [ONFlowConfig builder];
    ...
    NSError * error = NULL;
    ONCertificatePinningConfiguration * pinningConf = [[ONCertificatePinningConfiguration alloc] initWithHashes: @[@"EXAMPLE_HASH"] error: &error]];
    if(error != NULL) {
      // handle ONCertificatePinningConfiguration initialisation failures. i.e Providing empty array causes initialiser to be failed.

    }
    [builder withCertificatePinningConfiguration: pinningConf];

    ...
```

#### Handling Certificate Pinning Error

If you want to identify certificate pinning error from others, check `message` property of returned OnfidoFlowError.exception object, which should be `invalid_certificate` for certificate pinning related errors.  

```
let responseHandler: (OnfidoResponse) -> Void = { response in
  switch response {
    case let .error(error):
        // Some error happened
        if case OnfidoFlowError.exception(withError: _, withMessage: let optionalMessage) = error, let message = optionalMessage {
            if message == "invalid_certificate" {
                // HANDLE INVALID CERTIFICATE CASE HERE
            }
        }        
    case let .success(results):
        // User completed the flow
        // You can create your check here
    case .cancel:
        // Flow cancelled by the user
  }
}
```

## Accessibility

The Onfido iOS SDK has been optimised to provide the following accessibility support by default:

- Screen reader support: accessible labels for textual and non-textual elements available to aid VoiceOver navigation, including dynamic alerts
- Dynamic font size support: all elements scale automatically according to the device's font size setting
- Sufficient color contrast: default colors have been tested to meet the recommended level of contrast
- Sufficient touch target size: all interactive elements have been designed to meet the recommended touch target size

## Licensing

Due to API-design constraints, and to avoid possible conflicts during the integration, we bundle some of our 3rd party dependencies.  For those, we include the licensing information inside our bundle, with the file named `onfido_licenses.json`.
This file contains a summary of our bundled dependencies and all the licensing information required, including links to the relevant license texts contained in the same folder.
Integrators of our library are then responsible for keeping this information along with their integrations.

Example on how to access the licenses:
```
let onfidoBundle = Bundle(for: OnfidoFlow.self)
guard let licensesPath = onfidoBundle.path(forResource: "onfido_licenses", ofType: "json", inDirectory: nil),
    let licensesData = try? Data(contentsOf: URL(fileURLWithPath: licensesPath)),
    let licensesContent = String(data: licensesData, encoding: .utf8) else {
        return
}

print(licensesContent)

guard let mitLicensePath = onfidoBundle.path(forResource: "onfido_licenses_mit", ofType: "txt", inDirectory: nil),
    let mitLicenseData = try? Data(contentsOf: URL(fileURLWithPath: mitLicensePath)),
    let mitLicenseFileContents = String(data: mitLicenseData, encoding: .utf8) else {
        return
}

print(mitLicenseFileContents)
```

## More Information

### Sample App

We have included sample apps to show how to integrate with the Onfido SDK using both Swift and Objective-C. Check out respectively the `SampleApp` and `SampleAppObjC` directories.

### Support

Please open an issue through [GitHub](https://github.com/onfido/onfido-ios-sdk/issues). Please be as detailed as you can. Remember **not** to submit your token in the issue. Also check the closed issues to check whether it has been previously raised and answered.

If you have any issues that contain sensitive information please send us an email with the `ISSUE:` at the start of the subject to [ios-sdk@onfido.com](mailto:ios-sdk@onfido.com?Subject=ISSUE%3A)

Previous version of the SDK will be supported for a month after a new major version release. Note that when the support period has expired for an SDK version, no bug fixes will be provided, but the SDK will keep functioning (until further notice).

Copyright 2018 Onfido, Ltd. All rights reserved.
