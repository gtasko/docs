## Windows Phone 8

### Using the Windows Phone 8 library

**Requirements**

1. Visual Studio 2012 for Windows Phone (Update 1 or later)
2. NuGet (latest version)
3. .NET Framework 4.5 or later.

**Installation**

To install the BugSense NuGet package by using the [Package Manager Console](http://docs.nuget.org/docs/start-here/using-the-package-manager-console), do the following:

1. Open the project you want to work with BugSense.
2. On the TOOLS menu, point to Library Package Manager, and then click Package Manager Console.
3. In the Package Manager Console at the PM> prompt, type the following: 

```Install-Package BugSense.WP8```

To install the BugSense SDK by using the Package Manager window in Visual Studio:

1. Open the project you want to work with BugSense.
2. On the **TOOLS** menu, point to **Library Package Manager**, and then click **Manage NuGet Packages for Solution**
3. In the **Manage NuGet Packages** window, click **Online** from the list on the left, and then enter "BugSense.WP8" into the Search Online field in the upper-right corner. Several BugSense packages appear in the list.
4. Click the **Install** button for the BugSense package that you want to install.
5. In the **Select Projects** window, select the checkboxes next to the projects in which you want to install the package, and then click **OK**

You can also use the [Package Management Dialog](http://docs.nuget.org/docs/start-here/managing-nuget-packages-using-the-dialog) in Visual Studio to install the 'BugSense.WP8' package.

![Installing BugSense via NuGet in Visual Studio 2012](/static/images/landing/screens/install.jpg "Installing BugSense via NuGet in Visual Studio 2012")

Alternatively, you can download the <a href="">BugSense-WP8-<strong></strong>.zip</a><strong> <a href="/releases/windows8" id="releases">(Release Notes)</a></strong> file & add a reference to **BugSense-WP8.dll** in your Windows Project.

Then, inside the **App.xaml.cs** file, add the following code inside the constructor.

```c#
public App()
{
  // Initialize BugSense
  BugSenseHandler.Instance.InitAndStartSession(new ExceptionManager(Current), "API_KEY");
  // Other Windows Store specific operations
}
```

You can remove any **UnhandledException** handlers as follows:

```c#
UnhandledException += Application_UnhandledException;
```

If you're in debugging mode, and you don't want to send the exceptions to BugSense, you can use the **HandleWhileDebugging** property.

```c#
BugSenseHandler.Instance.HandleWhileDebugging = false;
```

Now you can ship your application and stay cool. We will make sure you won't miss a bug.

### End user communication

To send custom UI messages to the user there is more than one method.

When immediately send an exception to BugSense you can examine the BugSenseResponseResult return object where is the property IsResolved, which corresponds to the specific error that has occurred. You'll also find the ContentText, ContentTitle, TickerText properties, which are set in your dashboard settings. You should first make sure that the exception is sent successfully with the corresponding ResultState property.

#### Log handled exceptions and send custom data

To use BugSense to log handled exceptions and send useful custom data:

```c#
// Get the global LimitedCrashExtraData list.
LimitedCrashExtraDataList extras = BugSenseHandler.Instance.CrashExtraData;

// Add some global extras. 
extras.Add(new CrashExtraData
{
   Key = "TestApp1",
   Value = "After Refactor 1"
 });
extras.Add(new CrashExtraData
{
   Key = "TestApp2",
   Value = "After Refactor 2"
});

try
{
   // The exception is forced here
   String a = null;
   a.ToCharArray();
}
catch (Exception ex)
{
    // Invoke the LogException method with additional extra data that will be merged with the global extras in the request.
BugSenseLogResult logResult = BugSenseHandler.Instance.LogException(ex, "TestApp3", "Send Exception Message");
   // Examine the ResultState to determine whether it was successful.
Debug.WriteLine("Result: {0}", logResult.ResultState.ToString());
}
```

There is also a **LogException** overload method that accepts the Exception object and a **LimitedCrashExtraData** object for extra data which is optional.

```c#
LimitedCrashExtraDataList extrasExtraDataList = new LimitedCrashExtraDataList
{
  new CrashExtraData("TestApp1", "Log Exception Message1"),
  new CrashExtraData("TestApp2", "Log Exception Message2")
};
BugSenseLogResult logResult = BugSenseHandler.Instance.LogException(ex, extrasExtraDataList);
```

Additionally, you can use the **async** method. Note that you can’t use the **await** operator inside a catch block; instead get the **Exception** object reference and log outside the catch block.

```c#
// The Exception object reference
Exception exception;
try
{
  // force throw an exception for test
  throw new NotSupportedException("Not supported exception.");
}
catch (NotSupportedException ex)
{
  // get the exception object reference
  exception = ex;
}
// Add some global custom extra data
BugSenseHandler.Instance.AddCrashExtraData(new CrashExtraData { Key = "TestKey", Value = "TestValue" });
// Log the exception async
BugSenseResponseResult result = await BugSenseHandler.Instance.LogExceptionAsync(exception);
// Examine the raw ServerResponse
Debug.WriteLine("Response: {0}", result.ServerResponse);
```

Be aware that the parameter for extra data is optional; simply invoke the method with the **Exception** object only.

#### Send an immediate exception with custom data

You can use BugSense to immediately try to send an exception with custom data.

Just as there are **LogException** and **LogExceptionAsync** methods, there are **SendException** and **SendExceptionAsync** methods.

These methods will try to send the exception. If that fails for any reason, it will log the exception to be sent upon application restart.

```c#
LimitedCrashExtraDataList extrasExtraDataList = new LimitedCrashExtraDataList
{
  new CrashExtraData("TestApp1", "Log Exception Message1"),
  new CrashExtraData("TestApp2", "Log Exception Message2")
};
// Synchronous
BugSenseResponseResult logResult = BugSenseHandler.Instance.SendException(exception, extrasExtraDataList);
// Asynchronous
BugSenseResponseResult logResult = await BugSenseHandler.Instance.SendExceptionAsync(exception, extrasExtraDataList);
```

The APM asynchronous .NET API is used by BugSense and so the return BugSenseResponseResult object will not contain any useful information, instead register to the BugSenseRequestCompleted event and get notified for the actual result of the request when finished.

#### Search by username

BugSense enables you to closely track the experience of users. Use the **UserIdentifier** property to provide a user identifier such as an ID number from a customer database, e-mail address, push ID, or username. In your dashboard, you search for errors that affect that user. This feature is ideal for apps with high average revenue per user (ARPU) or apps deployed in a mobile device management (MDM) environment or during your app's quality assurance (QA) phase.

```c#
BugSenseHandler.UserIdentifier = "A VALUE";
```

#### Add breadcrumbs and custom data to crash reports

You can add "breadcrumbs" and other custom data to your crash reports.

```c#
// Add breadcrumb to the global breadcrumb list
BugSenseHandler.Instance.LeaveBreadCrumb("A Breadcrumb");
// Clear global breadcrumb list
BugSenseHandler.Instance.ClearBreadCrumbs();

// Adding global custom data can be achieved using the following methods
// Here you can get global LimitedCrashExtraDataList and add instances
LimitedCrashExtraDataList extras = BugSenseHandler.Instance.CrashExtraData;
extras.Add(new CrashExtraData
{
  Key = "TestApp1",
  Value = "After Refactor 1"
});

// or with the following method adding an instance directly
BugSenseHandler.Instance.AddCrashExtraData(new CrashExtraData { Key = "TestKey", Value = "TestValue" });

// Clear global custom data
BugSenseHandler.Instance.ClearCrashExtraData ();

// Remove a custom extra data instance with key
BugSenseHandler.Instance.RemoveCrashExtraData("A Key To Remove");
```

#### Set an action to be executed before the app crashes (save state)

You can specify an action that is to be executed just before an app crashes.

```c#
BugSenseHandler.Instance.LastActionBeforeTerminate(Debug.WriteLine("Last Action Invoked!"));
```

#### Use BugSense along with the Visual Studio debugger

You can use BugSense alongside the Visual Studio debugger.

```c#
// You can set the HandleWhileDebugging property to true
// This property is true by default
BugSenseHandler.Instance.HandleWhileDebugging = true;
```

#### Track an event

You can use BugSense to track a specific event.

```c#
// Log an event using the following method
BugSenseLogResult result = BugSenseHandler.Instance.LogEvent("Test Log Event!");

// Try to send an event immediately
BugSenseResponseResult result = BugSenseHandler.Instance.SendEvent("Test Send Event!");

// Async equivalent
BugSenseLogResult result = await BugSenseHandler.Instance.LogEventAsync("Test Log Event!");
BugSenseResponseResult result = await BugSenseHandler.Instance.SendEventAsync("Test Send Event!");
```

#### CrashOnLastRun example

Add the following code to the **App.xaml.cs** file, or to your **MainPage.xaml.cs** file's **NavigateTo** or **Loaded** events. Where you place it depends on where you first want to check for the total crashes that have occurred in the client's app install, and then act. Since these methods are asynchronous, you should always take any actions as late as possible, while giving BugSense time to send all the logged unhandled exception crashes of the last session to the server and to update the properties, so the results are accurate.

```c#
this.Loaded += async (sender, args) =>
{
  await Task.Delay(5000);
  int totalCrashes = await BugSenseHandler.Instance.GetTotalCrashesNum();
  Debug.WriteLine("TotalCrashes: {0}", totalCrashes);
  if (totalCrashes > 3)
  {
    await BugSenseHandler.Instance.ClearTotalCrashesNum();
    Debug.WriteLine("TotalCrashes after clear: {0}", await BugSenseHandler.Instance.GetTotalCrashesNum());
  }
  Debug.WriteLine("Last Crash Error ID: {0}", await BugSenseHandler.Instance.GetLastCrashId());
};
```

#### Async void and unobserved task exceptions

BugSense will log any asynchronous unawaited void or unobserved task exceptions.

```c#
// Example
this.Loaded += async (sender, args) =>
{
BugSenseHandler.Instance.RegisterAsyncHandlerContext();
}
```

#### Send logged requests on demand

You can send all logged requests on demand with the following method:

```c#
await BugSenseHandler.Instance.Flush();
```

#### Event notifications

You can register to BugSense events and get notified when unhadled exceptions and logged requests are handled.

```c#
// Register to the UnhandledExceptionHandled event to get information when an unhandled exception is handled by BugSense.
BugSenseHandler.Instance.UnhandledExceptionHandled += (sender, response) =>
                Debug.WriteLine("Exception of type {0} handled by BugSense {1}\r\nAction: {2} with ClientRequest: {3}",
                    response.ExceptionObject.GetType(), response.HandledSuccessfully ? "successfully." : "unsuccessfully.",
                    response.HandleAction, response.ClientJsonRequest);

// Register to the LoggedRequestHandled event to get notified when a logged request is handled by BugSense.
BugSenseHandler.Instance.LoggedRequestHandled += (sender, args) =>
                Debug.WriteLine("Logged request {0} processed with server response: {1}",
                    args.BugSenseLoggedResponseResult.ClientRequest,
                    args.BugSenseLoggedResponseResult.ServerResponse);
```

#### Close a session

Send a GNIP request to close a session.

```c#
BugSenseHandler.Instance.CloseSession();
// Async equivalent
await BugSenseHandler.Instance.CloseSessionAsync();
```
