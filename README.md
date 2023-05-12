# Flutter Background Service

A flutter plugin for execute dart code in background.

## Android

1. No additional setting is required.

2. To change notification icon, just add drawable icon with name ic_bg_service_small.

### Using custom notification for Foreground Service

You can make your own custom notification for foreground service. It can give you more power to make notifications more attractive to users, for example adding progressbars, buttons, actions, etc. The example below is using flutter_local_notifications plugin, but you can use any other notification plugin. You can follow how to make it below:

### Notification Channel

   ```
   Future<void> main() async {
    WidgetsFlutterBinding.ensureInitialized();
    await initializeService();

    runApp(MyApp());
}

// this will be used as notification channel id
const notificationChannelId = 'my_foreground';

// this will be used for notification id, So you can update your custom notification with this id.
const notificationId = 888;

Future<void> initializeService() async {
  final service = FlutterBackgroundService();

  const AndroidNotificationChannel channel = AndroidNotificationChannel(
    notificationChannelId, // id
    'MY FOREGROUND SERVICE', // title
    description:
        'This channel is used for important notifications.', // description
    importance: Importance.low, // importance must be at low or higher level
  );

  final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin =
      FlutterLocalNotificationsPlugin();

  await flutterLocalNotificationsPlugin
      .resolvePlatformSpecificImplementation<
          AndroidFlutterLocalNotificationsPlugin>()
      ?.createNotificationChannel(channel);

  await service.configure(
    androidConfiguration: AndroidConfiguration(
      // this will be executed when app is in foreground or background in separated isolate
      onStart: onStart,

      // auto start service
      autoStart: true,
      isForegroundMode: true,

      notificationChannelId: notificationChannelId, // this must match with notification channel you created above.
      initialNotificationTitle: 'AWESOME SERVICE',
      initialNotificationContent: 'Initializing',
      foregroundServiceNotificationId: notificationId,
    ),
    ...
   ```

### Update notification info

   ```
   Future<void> onStart(ServiceInstance service) async {
  // Only available for flutter 3.0.0 and later
  DartPluginRegistrant.ensureInitialized();

  final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin =
      FlutterLocalNotificationsPlugin();

  // bring to foreground
  Timer.periodic(const Duration(seconds: 1), (timer) async {
    if (service is AndroidServiceInstance) {
      if (await service.isForegroundService()) {
        flutterLocalNotificationsPlugin.show(
          notificationId,
          'COOL SERVICE',
          'Awesome ${DateTime.now()}',
          const NotificationDetails(
            android: AndroidNotificationDetails(
              notificationChannelId,
              'MY FOREGROUND SERVICE',
              icon: 'ic_bg_service_small',
              ongoing: true,
            ),
          ),
        );
      }
    }
  });
}
   ```

## iOS

1. Enable background_fetch capability in xcode (optional), if you wish ios to execute IosConfiguration.onBackground callback.

2. For iOS 13 and Later (using BGTaskScheduler), insert lines below into your ios/Runner/Info.plist

   ```
   <key>BGTaskSchedulerPermittedIdentifiers</key>
   <array>
   <string>dev.flutter.background.refresh</string>
   </array>
   ```
   
3. You can also using your own custom identifier In ios/Runner/AppDelegate.swift add line below

   ```
   import UIKit
   import Flutter
   import flutter_background_service_ios // add this

   @UIApplicationMain
   @objc class AppDelegate: FlutterAppDelegate {
   override func application(
   _ application: UIApplication,
   didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
   ) -> Bool {
   /// Add this line
   SwiftFlutterBackgroundServicePlugin.taskIdentifier = "your.custom.task.identifier"

    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
   }
   }
   ```
   
## Usage

1. Call FlutterBackgroundService.configure() to configure handler that will be executed by the Service.

2. It's highly recommended to call this method in main() method to ensure the callback handler updated.

3. Call FlutterBackgroundService.start to start the Service if autoStart is not enabled.

4. Since the Service using Isolates, You won't be able to share reference between UI and Service. You can communicate between UI and Service using invoke() and on(String method).

## Migration

1. sendData() renamed to invoke(String method)

2. onDataReceived() renamed to on(String method)

3. Now you have to use ServiceInstance object inside onStart method instead of creating a new FlutterBackgroundService object. See the example project.

4. Only use FlutterBackgroundService class in UI Isolate and ServiceInstance in background isolate.


