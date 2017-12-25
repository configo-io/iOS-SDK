![alt tag](https://s3.eu-central-1.amazonaws.com/configo.io/overview.png)

# Configo iOS SDK Documentation (1.9.x)

### Table of Contents

<!-- MarkdownTOC -->

- [Overview](#overview)
- [Getting Started](#getting-started)
- [Initialize](#initialize)
- [Target Users](#target-users)
- [Trigger Refresh](#trigger-refresh)
  - [Asynchronous refresh](#asynchronous-refresh)
  - [Synchronous Refresh](#synchronous-refresh)
- [Dynamic Data Refresh](#dynamic-data-refresh)
- [SDK Events](#sdk-events)
- [Manual Data Access](#manual-data-access)
  - [Dynamic Variables](#dynamic-variables)
  - [Feature Flags](#feature-flags)
  - [Popups](#popups)

<!-- /MarkdownTOC -->


<a name="overview"></a>
## Overview
Configo enables customer centric mobile experience that drives conversion, engagement and sales. 
Personalize and optimize the user interface, so every user gets the most engaging and familiar experience.
Launch new marketing and engagement campaigns to your users within minutes, engage your users in the right moment by triggering the campaign presentation based on the users activity and actions in the app.
Achieve with great speed and flexibility using Configo's unique screen scanning and tagging technology.


<a name="getting-started"></a>
## Getting Started
Adding the Configo SDK to your Xcode project.
  1. Request an iOS SDK package from Configo. 
  2. Unzip the file and drag the *ConfigoSDK.framework* directory to your Xcode project window, make sure to tick the "copy items if needed" checkbox.
  3. Make sure *ConfigoSDK.framework* shows up in your Xcode project settings *Linked Frameworks and Libraries* section.
  4. Add the following frameworks to your projects dependencies, from your target's "General" configuration tab, under "Linked Frameworks and Libraries".
      ```
      SystemConfiguration.framework
      CoreTelephony.framework
      CoreLocation.framework
      ```
  5. Add Configo's URL scheme to the app's Xcode target configuration under the "Info" tab, "URL Types". The scheme can be found in the dashboard in the `settings` screen.
  6. To have the SDK code load fully and correctly, 
  Add the following "Other Linker Flags" in the target's "Build Settings" tab:
      ```
      -ObjC
      -all_load
      ```
 
 
<a name="initialize"></a>
## Initialize 
1. In your app delegate, add the following import: 
    ```objective-c
    #import <ConfigoSDK/ConfigoSDK.h>
    ```

    ```swift
    import ConfigoSDK //Swift
    ```
  
2. Add the following line in your app delegate's `application:didFinishLaunchingWithOptions:` method with your `AppID` and `DeveloperKey` (dashboard `Settings` screen) and an optional block of code (i.e. `callback`). The callback will be executed every time new data was loaded, **on the main thread**:    
    ```objective-c
    [Configo initWithDevKey: @"DEV_KEY" appId: @"APP_ID" callback: ^(NSError *err) {
        if(err) NSLog(@"Failed to load data");
        else NSLog(@"The data was loaded successfully");
    }];
    ```

    ```swift
    //Swift
    Configo.initWithDevKey("DEV_KEY", appId: "APP_ID", callback: { err in
            if(err != nil) {
                print("Failed to load data");
            } else {
                print("The data was loaded successfully");
            }
    })
    ```
  
    **NOTE:** The initialization should be called only once in the lifetime of the app. It has no effect on any consecutive calls.

3. Add the following line in your app delegate's `application:openURL:options:` method:
    ```objective-c
    [[Configo sharedInstance] handleInterfacesUrl: url];
    ```

    ```swift
    Configo.sharedInstance().handleInterfacesUrl(url) //Swift
    ```

4. Add the following line in your app delegate's `application:didRegisterForRemoteNotificationsWithDeviceToken:` method:
    ```objective-c
    [[Configo sharedInstance] setPushToken: deviceToken];
    ```

    ```swift
    Configo.sharedInstance().setPushToken(deviceToken) //Swift
    ```

5. Finally, in the app delegate's `application:didReceiveRemoteNotification:fetchCompletionHandler:` add the following:
    ```objective-c
    [[Configo sharedInstance] handlePushNotification: userInfo withHandler: completionHandler];
    ```

    ```swift
    Configo.sharedInstance().handlePushNotification(userInfo, withHandler:  completionHandler) //Swift
    ```

6. **Optional**: configure the different options of the SDK using the following methods (do this before initializing for optimal results):
    ```objective-c
    //Set the log output level
    [Configo setLoggingLevel: (CFGLogLevel enum)value];

    //Set the base url of the Configo server, useful for switching environments in an on-premises setup
    [Configo setBaseUrl: (NSString *)value];

    //Set if the changes in the dashboard will take effect in real-time or in next launch
    [Configo setDynamicallyRefreshData: (boolean)value];

    //Set if Configo should skip status checks and pull data immediately, useful fo tests
    [Configo setShouldCheckStatusOnPolling: (boolean)value];

    //Set the interval in which Configo polls the server for updates
    [Configo setPollingInterval: (NSTimeInterval)value];

    //Set the time the app needs to be in the background to count as a new session
    [Configo setBackgroundSessionRenewThreshold: (NSTimeInterval)value];
    ```

    ```swift
    //Swift

    Configo.setLoggingLevel(CFGLogLevel.value)

    Configo.setBaseUrl("https://configo.api.path")

    Configo.setDynamicallyRefreshData(true/false)

    Configo.setShouldCheckStatusOnPolling(true/false)

    Configo.setPollingInterval(NSTimeInterval)

    Configo.setBackgroundSessionRenewThreshold(NSTimeInterval)
    ```

<a name="target-users"></a>
## Target Users
All users will be tracked as anonymous users unless a custom id and attributes are set.

Anonymous users can be segmented by their device attributes:

* Platform (OS)
* System Version (OS Version)
* Device Model
* Screen Resolution
* Application Name
* Application Identifier
* Application Version
* Application Build Number
* Connection Type (WiFi/Cellular)
* Carrier Name
* Device Language
* Time Zone
* Location, if available (Country / City)
* Permissions: Contacts, GPS, Notifications

**NOTE:** The SDK will collect the location only if authorized by the user. The SDK will **not** prompt the user for using location services. This must be triggered by the developer.

Identifying and segmenting users for targeted dynamic variables and features can be done with the following:

* Passing a user identifier such as an email or a username (We advise using a unique value):

  ```objective-c
  [[Configo sharedInstance] setCustomUserId: @"john@gmail.com"];
  ```

  ```swift
  Configo.sharedInstance().setCustomUserId("john@gmail.com") //Swift
  ```

* Passing user context that can give more specific details about the user and targeting the user more precisely, using either of two ways:
  Passing an `NSDictionary`:
  
  ```objective-c
  [[Configo sharedInstance] setUserContext: @{@"first-name" : @"John", @"account-number": @"12345687"}];
  ```

  ```swift
  Configo.sharedInstance().setUserContext([AnyHashable : Any]); //Swift
  ```
  
  Setting every attribute individually (in different classes through out the app):
  
  ```objective-c
  [[Configo sharedInstance] setUserContextValue: @"first-name" forKey: @"John"];
  ```

  ```swift
  Configo.sharedInstance().setUserContextValue("John", forKey: "first-name") //Swift
  ```
  
All values set in the `userContext` must be JSON compatible ([Apple Docs](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSJSONSerialization_Class/)):
* `NSDictionary` or `NSArray` (All objects must be JSON compatible as well)
* `NSString`
* `NSNumber` (non NaN or infinity)
* `NSNull`

<a name="trigger-refresh"></a>
## Trigger Refresh

Configo constantly synchronizes with the dashboard upon app launch and by using the push mechanism. Configo looks out for any local changes that need to be synchronized and updates accordingly.
Sometimes a manual refresh of the configurations is required:

<a name="asynchronous-refresh"></a>
### Asynchronous refresh

This type of refresh will not stop your application from running is best suited for background refreshes.

```objective-c
[[Configo sharedInstance] pullData: ^(NSError *err) {
      //Code for handling dynamic variables / feature updates
}];
```

```swift
//Swift
Configo.sharedInstance().pullData({
    err in
    //code
})
```

**NOTE:** The callback set here will only be executed once, when that specific call was made. It will have no effect on the callback set using the `setCallback:` method.

<a name="synchronous-refresh"></a>
### Synchronous Refresh

This type of refresh will stop the application main functionality, best suited for loading screens where the code relies on the data provided by Configo:

```objective-c
[[Configo sharedInstance] pullDataSyncWithTimeout: 10.0];
//Code for handling dynamic variables / feature updates
```

```swift
Configo.sharedInstance().pullDataSync(withTimeout: NSTimeInterval) //Swift
//Code for handling dynamic variables / feature updates
```

**NOTE:** This will stop the main thread from executing any further code and wait for Configo to return a response.


<a name="dynamic-data-refresh"></a>
## Dynamic Data Refresh
The data is updated and loaded every time the app opens, to avoid inconsistency at runtime. The data will be updated at runtime in the following scenarios:

1. Calling `pullData:` with a valid `callback`.
2. Calling `setDynamicallyRefreshData:` with `YES`.
3. Calling `forceRefreshData`.

<a name="sdk-events"></a>
## SDK Events
Configo's operational state can be retrieved using several methods:

1. Callbacks
2. Polling for state

##### Callbacks

Using a Objective-C `blocks` is a convenient way to execute code in response to events. Configo expects all blocks to be of the `CFGCallback` type with the following definition:

```objective-c
typedef void(^CFGCallback)(NSError *error);
```

Callback APIs:
- `addCallback:withId:` To add a callback with a certain id.
- `removeCallbackWithId:` To remove a callback with a certain id.
- During intialization `+ initWithDevKey:appId:callback:` This will add a callback with the id `main`.

If a manual data refresh is triggered an optional callback can be set as well `pullData:`. This will set a "temporary" callback that will only be called once when the manual refresh is complete. This will not affect the callbacks set using `addCallback:withId`.

**NOTE:** All callbacks will be triggered upon an update, even on a manual update.

##### Configo State

Configo also holds a property named state that can hold either of the following values:

<pre>
//There is no data available.
<b>CFGNotAvailable</b>

//The data was loaded from local storage (possibly outdated).
<b>CFGLoadedFromStorage</b>

//The data is being loaded from the server. If there is an old, local data - it is still avaiable to use.
<b>CFGLoadingInProgress</b>

//The data is has being loaded from the server and is ready for use. Might not be active if dynamicallyRefreshValues is false.
<b>CFGLoadedFromServer</b>

//An error was encountered when loading the data from the server (Possibly no data is available).
<b>CFGFailedLoadingFromServer</b>
</pre>

<a name="manual-data-access"></a>
## Manual Data Access
When not using Configo's scanning technology, all of the functionality can be achieved using the APIs detailed below.

<a name="dynamic-variables"></a>
### Dynamic Variables
Dynamic variables are values that can be bound to your app's code and changed in runtime from the dashboard. Variables are set on a segment basis, so we can customize the application look and behaviour based on the target the user belongs to.
Dynamic variables can be retrieved using:

```objective-c
[[Configo sharedInstance] dynamicVariableForKey: @"key" fallbackValue: @"fallbackString"];
```

```swift
Configo.sharedInstance().dynamicVariable(forKey: "key", fallbackValue: "fallback") //Swift
```

**NOTE:** The fallback value will be returned if an error was encountered or the variable was not found.

The variables are stored in a JSON document form and retrieved by the SDK as an `NSDictionary` collection.

Accessing the configuration value can be done using dot notation and brackets, e.g.:

In a `JSON` of the form:

```json
{
  "object": {
    "array": [1,2,3]
  }
}
```

The second value in the array can be accessed like so:

```objective-c
[[Configo sharedInstance] dynamicVariableForKey: @"object.array[1]" fallbackValue: nil];
```

```swift
Configo.sharedInstance.dynamicVariableForKey(@"object.array[1]") //Swift
```

<a name="feature-flags"></a>
### Feature Flags
Feature flags are like dyanmic variables but with boolean values only and with the additional options of distributing randomly to a portion of the users or to multiple segments and optional feature analytics.

Feature flags can be retrieved like so:

```objective-c
[[Configo sharedInstance] featureFlagForKey: @"feature-key" fallback: YES];
```

```swift
Configo.sharedInstance().featureFlag(forKey: "feature-key", fallback: true/false) //Swift
```

**NOTE:** The fallback BOOL will be returned if an error occurred or the feature was not found.

#### Feature Tracking

Configo analytics is used to measure feature adoption and performance using in conjuction with a feature's distribution method (random, gradual / targeted rollouts).

To track a feature analytics event, use the following snippet of code:

```objective-c
[[Configo sharedInstance] trackFeature: @"feature-key" action: (CFGEventActionType)action];
```

```swift
Configo.sharedInstance().trackFeature(featureKey: "feature-key", action: CFGEventActionType.value) //Swift
```

The possible event action types are as follows:

<pre>
//Feature viewed
<b>CFGEventActionView</b>

//Feature clicked / entered
<b>CFGEventActionClick</b>

//Feature used
<b>CFGEventActionUse</b>
</pre>

<a name="popups"></a>
### Popups
Popups can be designed and deployed in runtime from the dashboard and later easily presented in the applications. Popup analytics are collected automatically and presented for each popup individually in the dashboard.
To manually trigger a popup presentation use the following:
```objective-c
[[Configo sharedInstance] presentPopupWithName: @"popup-name" callback: ^(NSString *href) {
    //Handle callback
}];
```

```swift
Configo.sharedInstance().presentPopup(withName: "popup-name", callback: { href in
    print("The url from the campaign or 'close'");
}) //Swift
```

