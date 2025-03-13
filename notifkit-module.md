# NotifKit Integration Guidelines

## Overview
NotifKit wraps OneSignal's SDK to provide easy push notification functionality in your app, handling permission requests, notification delivery, and in-app notification display with minimal boilerplate code.

## Key Features and Components

### Installation & Configuration
- Set up Apple Push Notification Service (APNs) certificates in your Apple Developer account
- Configure a OneSignal project and obtain your OneSignal App ID
- Include the App ID in your project's OneSignal-Info.plist file
- Initialization is handled automatically at app launch, requiring no manual intervention

### Notification Permission Flow
- Display a pre-permission dialog with `PushNotifications.showNotificationsPermissionsSheet()`
- Check permission status with `PushNotifications.hasNotificationsPermission()`
- Monitor permission state through OneSignal's permission observers
- Automatically handle permission state changes with appropriate UI feedback

### Implementation Patterns
- Access NotifKit functionality through static methods on the `PushNotifications` class
- Monitor notification permission state through `OneSignal.Notifications.permission`
- Handle in-app notification display with customizable styles and interaction

## Implementation Guidelines

### Requesting Notification Permissions
```swift
// Check if we already have permission
let hasPermission = await PushNotifications.hasNotificationsPermission()

// If not, show the permission request sheet
if !hasPermission {
    PushNotifications.showNotificationsPermissionsSheet()
}
```

### Checking Permission Status
```swift
struct NotificationSettingsView: View {
    @State private var permissionStatus: OSNotificationPermission = .notDetermined
    
    var body: some View {
        VStack {
            if permissionStatus == .authorized {
                Text("Notifications are enabled")
            } else {
                Button("Enable Notifications") {
                    PushNotifications.showNotificationsPermissionsSheet()
                }
            }
        }
        .onAppear {
            permissionStatus = OneSignal.Notifications.permission
        }
    }
}
```

### Displaying In-App Notifications from Push Notifications
When your app is in the foreground, you can automatically convert push notifications to in-app notifications by including additional parameters in the notification payload:

```swift
// Backend code example (Firebase Cloud Functions)
await PushNotifications.sendNotificationToUserWithID({
    userID: userID,
    data: {
        title: "New Message",
        message: messageContent,
        additionalData: {
            inAppSymbol: "bell.fill",
            inAppColor: "#007AFF",
            inAppSize: "normal", // or "compact"
            inAppHaptics: "success" // or "warning", "error"
        }
    }
})
```

## VIPER Architecture Integration

When using NotifKit with the VIPER architecture:

### View
```swift
struct NotificationsView: View {
    @ObserveInjection var inject
    let presenter: NotificationsPresenterProtocol
    
    var body: some View {
        VStack {
            Button("Enable Notifications") {
                presenter.requestNotificationPermission()
            }
            .opacity(presenter.canRequestPermission ? 1.0 : 0.5)
            .disabled(!presenter.canRequestPermission)
        }
        .enableInjection()
    }
}
```

### Interactor
```swift
protocol NotificationsInteractorProtocol {
    func checkNotificationPermission() async -> Bool
    func requestNotificationPermission()
}

final class NotificationsInteractor: NotificationsInteractorProtocol {
    func checkNotificationPermission() async -> Bool {
        return await PushNotifications.hasNotificationsPermission()
    }
    
    func requestNotificationPermission() {
        PushNotifications.showNotificationsPermissionsSheet()
    }
}
```

### Presenter
```swift
protocol NotificationsPresenterProtocol: ObservableObject {
    var canRequestPermission: Bool { get }
    func requestNotificationPermission()
}

final class NotificationsPresenter: NotificationsPresenterProtocol {
    @Published private(set) var canRequestPermission: Bool = false
    private let interactor: NotificationsInteractorProtocol
    
    init(interactor: NotificationsInteractorProtocol) {
        self.interactor = interactor
        
        Task {
            await checkPermissionStatus()
        }
    }
    
    private func checkPermissionStatus() async {
        let hasPermission = await interactor.checkNotificationPermission()
        DispatchQueue.main.async {
            self.canRequestPermission = !hasPermission && 
                PushNotifications.canAskForNotificationsPermission()
        }
    }
    
    func requestNotificationPermission() {
        interactor.requestNotificationPermission()
    }
}
```

### Router
```swift
protocol NotificationsRouterProtocol {
    func createNotificationsModule() -> NotificationsView
}

final class NotificationsRouter: NotificationsRouterProtocol {
    func createNotificationsModule() -> NotificationsView {
        let interactor = NotificationsInteractor()
        let presenter = NotificationsPresenter(interactor: interactor)
        return NotificationsView(presenter: presenter)
    }
}
```

## Sending Push Notifications

### From the Backend (BackendKit)
Using BackendKit, you can easily send notifications to specific users:

```typescript
import * as PushNotifications from "./NotifKit/PushNotifications";

export const sendNotificationToUser = onCall(async (request) => {
  const fromUid = request.auth?.uid;
  if (!fromUid) {
    throw new Error("User Not Logged In");
  }

  const user = await getAuth().getUser(fromUid);
  const toUid = request.data?.userID;
  
  await PushNotifications.sendNotificationToUserWithID({
    userID: toUid,
    data: {
      title: `Message from ${user.displayName || fromUid}`,
      message: request.data?.message || "New message",
      additionalData: {
        inAppSymbol: "message.fill",
        inAppColor: "#007AFF",
        inAppSize: "compact",
        inAppHaptics: "success"
      }
    }
  });
  
  return { success: true };
});
```

### From the OneSignal Dashboard
You can also send notifications directly from the OneSignal dashboard:
1. Navigate to your app in the OneSignal dashboard
2. Go to Messages > Push
3. Create a new push notification with desired content
4. Select audience segment (all users, specific users, etc.)
5. Schedule for immediate or future delivery

## Integration with Other Modules

- **AuthKit**: User authentication enables Firebase User ID to be used as OneSignal external user ID for user targeting
- **BackendKit**: Send push notifications from your backend functions with simple API calls
- **AnalyticsKit**: Notification events (delivery, open, etc.) are automatically tracked
- **SharedKit**: In-app notifications leverage the same notification display system as local in-app notifications

## In-App Notification Customization

When configuring in-app notifications from push notifications, you can customize:

- **Symbol**: Any SF Symbol name (e.g., "bell.fill", "message.circle")
- **Color**: HEX color code for the symbol (e.g., "#FF0000" for red)
- **Size**: "normal" (default) or "compact"
- **Haptics**: "success", "warning", or "error" feedback types

## Permission Handling Best Practices

- Show the value of notifications before requesting permission
- Use pre-permission dialog to explain benefits before triggering system dialog
- Handle declined permissions gracefully with clear guidance on how to enable in Settings
- Check permission status on app launch and after returning from background

## Testing Considerations

- Test notifications in both foreground and background app states
- Verify proper permission handling across different initial states
- Confirm that notification taps trigger appropriate app navigation
- Test in-app notification display and interaction handling