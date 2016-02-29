Amplitude Unity Plugin
============

This plugin simplifies the integration of Amplitude iOS and Android SDKs into your Unity project. This respository also contains a sample project with the Unity plugin integrated. Open the project and replace the API key in AmplitudeDemo.cs with your key to see how the integration works.

For iOS and Android specific documentation, see [iOS README](https://github.com/amplitude/Amplitude-iOS/blob/master/README.md) and [Android README](https://github.com/amplitude/Amplitude-Android/blob/master/README.md)

# Setup #

1. If you haven't already, go to https://amplitude.com and register for an account. You will receive an API Key.

2. [Download the plugin package](https://github.com/amplitude/unity-plugin/raw/master/amplitude-unity.unitypackage).

3. Import the package into your Unity project. Select 'Import Package > Custom Package' from the 'Assets' menu.

4. In the Awake method in your main script, initialize and enable the Amplitude plugin in the:

    ```C#
    void Awake () {
      Amplitude amplitude = Amplitude.Instance;
      amplitude.logging = true;
      amplitude.init("YOUR_API_KEY_HERE");
    }
    ```

5. To track an event anywhere in your app, call:

    ```C#
    Amplitude.Instance.logEvent("EVENT_IDENTIFIER_HERE");
    ```

6. Events are saved locally. Uploads are batched to occur every 30 events and every 30 seconds. After calling `logEvent()` in your app, you will immediately see data appear on the Amplitude website.

NOTE if you are building your Unity project as an iOS app: Amplitude's iOS SDK requires the SQLite library, which is included in iOS but requires an additional build flag to enable. In Xcode, in your project's `Build Settings` and your Target's `Build Settings`, under `Linking` -> `Other Linker Flags`, add the flag `-lsqlite3`.

# Tracking Events #

It's important to think about what types of events you care about as a developer. You should aim to track between 20 and 100 types of events within your app. Common event types are different screens within the app, actions a user initiates (such as pressing a button), and events you want a user to complete (such as filling out a form, completing a level, or making a payment). Contact us if you want assistance determining what would be best for you to track.

# Tracking Sessions #

A session is a period of time that a user has the app in the foreground. Events that are logged within the same session will have the same `session_id`. Sessions are handled automatically now; you no longer have to manually call `startSession()` or `endSession()`.

You can also log events as out of session. Out of session events have a `session_id` of `-1` and are not considered part of the current session, meaning they do not extend the current session (useful for things like push notifications). You can log events as out of session by setting input parameter `outOfSession` to `true` when calling `logEvent()`:

```C#
Amplitude.Instance.logEvent("EVENT", null, true);
```

By default start and end session events are no longer sent. To renable add this line before initializing the SDK:
```C#
Amplitude amplitude = Amplitude.Instance;
amplitude.trackSessionEvents(true);
amplitude.init("YOUR_API_KEY_HERE");
```

# Setting Custom User IDs #

If your app has its own login system that you want to track users with, you can call `setUserId()` at any time:

```C#
Amplitude.Instance.setUserId("USER_ID_HERE");
```

A user's data will be merged on the backend so that any events up to that point on the same device will be tracked under the same user.

You can also add a user ID as an argument to the `init()` call:

```
Amplitude.Instance.init("YOUR_API_KEY_HERE", "USER_ID_HERE");
```

# Setting Event Properties #

You can attach additional data to any event by passing a JSONObject as the second argument to `logEvent()`:

```C#
Dictionary<string, object> demoOptions = new Dictionary<string, object>() {
  {"Bucket" , "A" },
  {"Credits" , 9001}
};
Amplitude.Instance.logEvent("Sent Message", demoOptions);
```

# Setting User Properties #

To add properties that are associated with a user, you can set user properties:

```C#
Dictionary<string, object> userProperties = new Dictionary<string, object>() {
  {"float_gprop", 1.0}
};
Amplitude.Instance.setUserProperties(userProperties);
```

# Clearing User Properties #

To clear properties that are associated with a user, you can unset user properties:

```C#
string[] userProperties = new[] { "float_gprop" };
Amplitude.Instance.unsetUserProperties(userProperties);
```

# Tracking Revenue #

To track revenue from a user, call `logRevenue()` each time a user generates revenue. For example:

```C#
Amplitude.Instance.logRevenue("com.company.productid", 1, 3.99);
```

`logRevenue()` takes a takes a string to identify the product (the product ID from Google Play), an int with the quantity of product purchased, and a double with the dollar amount of the sale. This allows us to automatically display data relevant to revenue on the Amplitude website, including average revenue per daily active user (ARPDAU), 1, 7, 14, 30, 60, and 90 day revenue, lifetime value (LTV) estimates, and revenue by advertising campaign cohort and daily/weekly/monthly cohorts.

The logRevenue method also supports revenue validation. See the iOS and Android specific docs for more details. Here is a simple example:

```C#
if (Application.platform == RuntimePlatform.IPhonePlayer) {
  Amplitude.Instance.logRevenue("sku", 1, 1.99, "cmVjZWlwdA==", null);
} else if (Application.platform == RuntimePlatform.Android) {
  Amplitude.Instance.logRevenue("sku", 1, 1.99, "receipt", "receiptSignature");
}
```

# Allowing Users to Opt Out

To stop all event and session logging for a user, call setOptOut:

```C#
Amplitude.Instance.setOptOut(true);
```

Logging can be restarted by calling setOptOut again with enabled set to false.
No events will be logged during any period opt out is enabled.

