# SwiftyLaunch Analytics Integration Guidelines

This document explains how AnalyticsKit and other SwiftyLaunch modules integrate with the app's core infrastructure, including analytics initialization, tracking patterns, data consistency, and module interdependencies. Understanding these integration patterns is essential for effective analytics implementation across your app.

## Analytics Initialization Process

AnalyticsKit is designed to be initialized early in the app lifecycle to ensure proper tracking of all user activities and app events.

### PostHog Initialization in AppDelegate

The `AppDelegate` initializes PostHog before other modules to ensure analytics tracking is available throughout the app lifecycle:

```swift
class AppDelegate: NSObject, UIApplicationDelegate {
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil
    ) -> Bool {
        // Analytics initialization must happen first
        Analytics.initPostHog()  // AnalyticsKit
        
        // Other module initializations follow
        DB.initFirebase()        // DatabaseKit
        
        // Connect user IDs across modules when AuthKit is present
        Auth.auth().addStateDidChangeListener { auth, user in
            if let user = user {
                // Associate user ID with analytics, notifications, etc.
                Analytics.associateUserWithID(user.uid)
            } else {
                // Handle signed-out state
                Analytics.removeUserIDAssociation()
            }
        }
        
        return true
    }
}
```

### Environment Objects in MainApp

The Analytics controller is attached as an environment object to make it accessible throughout the app:

```swift
@main
struct MainApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate
    @StateObject var analytics = Analytics()
    @StateObject var db = DB()
    @StateObject var iap = InAppPurchases()
    // Other module controllers...
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                // App-wide modifiers
                .environmentObject(analytics)
                .environmentObject(db)
                .environmentObject(iap)
                // Other environment objects...
        }
    }
}
```

## Analytics Integration Patterns

AnalyticsKit provides standardized patterns for tracking events across all SwiftyLaunch modules.

### Event Capture Pattern

All modules should use the consistent capture pattern for analytics tracking:

```swift
Analytics.capture(
    .success,             // EventType: .info, .error, .success
    id: "event_name",     // Snake case string identifier
    longDescription: "Detailed event description", // Optional
    source: .general,     // EventSource: .general, .auth, .db, etc.
    fromView: "ViewName", // Optional, name of the originating view
    relevancy: .medium    // Optional, defaults to .medium
)
```

### Event Source Consistency

Each module should use its designated EventSource when capturing events:

```swift
public enum EventSource: String {
    case general
    case auth       // AuthKit
    case db         // DatabaseKit
    case push       // NotifKit
    case purchases  // InAppPurchaseKit
    case adsKit     // AdsKit
    case aiKit      // AIKit
    case backendKit // BackendKit
    // Add your custom sources here
}
```

### User Property Standard

User properties should be tracked consistently using predefined keys in the Constants:

```swift
public struct Constants {
    // Analytics-specific constants
    public struct Analytics {
        public static let userPropertyKeys = [
            "subscription_status",
            "app_opens_count",
            "user_id",
            "account_created_date"
        ]
        
        public static let subscriptionStatusValues = [
            "free_tier",
            "basic_subscription",
            "premium_subscription",
            "trial"
        ]
    }
}
```

## Module-Specific Analytics Integration

### AuthKit Analytics Integration

When AuthKit is present, it tracks the following user authentication events:

```swift
// During sign-in
Analytics.capture(
    .success,
    id: "user_sign_in",
    source: .auth,
    fromView: "SignInView"
)

// During account creation
Analytics.capture(
    .success,
    id: "account_created",
    source: .auth,
    fromView: "SignUpView"
)

// Setting user properties after authentication
Analytics.setUserProperty("account_created_date", to: Date().timeIntervalSince1970)
```

### NotifKit Analytics Integration

Push notification interactions are tracked with the following events:

```swift
// When permission is granted
Analytics.capture(
    .success,
    id: "notification_permission_granted",
    source: .push
)

// When notification is received
Analytics.capture(
    .info,
    id: "notification_received",
    longDescription: "Notification type: \(notificationType)",
    source: .push
)

// When notification is opened
Analytics.capture(
    .info,
    id: "notification_opened",
    longDescription: "Notification type: \(notificationType)",
    source: .push
)
```

### InAppPurchaseKit Analytics Integration

Purchase events are tracked with the following pattern:

```swift
// When a purchase is initiated
Analytics.capture(
    .info,
    id: "purchase_initiated",
    longDescription: "Product: \(product.id)",
    source: .purchases,
    fromView: "PaywallView"
)

// When a purchase succeeds
Analytics.capture(
    .success,
    id: "purchase_completed",
    longDescription: "Product: \(product.id)",
    source: .purchases,
    fromView: "PaywallView"
)

// When subscription status changes
Analytics.setUserProperty("subscription_status", to: "premium_subscription")
```

### AdsKit Analytics Integration

Ad interactions are tracked with the following pattern:

```swift
// When an ad is shown
Analytics.capture(
    .info,
    id: "ad_impression",
    longDescription: "Ad type: \(adType), placement: \(placement)",
    source: .adsKit,
    fromView: viewName
)

// When an ad is clicked
Analytics.capture(
    .info,
    id: "ad_click",
    longDescription: "Ad type: \(adType), placement: \(placement)",
    source: .adsKit,
    fromView: viewName
)
```

## View-Based Analytics

SwiftyLaunch provides view modifiers for tracking screen views and user interactions.

### View Tracking

All navigation should be automatically tracked using view modifiers:

```swift
struct ContentView: View {
    var body: some View {
        NavigationView {
            List {
                // View content
            }
            .navigationTitle("Home")
        }
        .captureViewActivity(as: "HomeView")
    }
}
```

### Button and Control Interaction Tracking

User interactions should be tracked with the `captureTaps` modifier:

```swift
Button("Sign In") {
    // Action code
}
.captureTaps("sign_in_button", fromView: "AuthView")
```

## Data Consistency Across Modules

### User Identification Pattern

When a user signs in, their ID should be shared with analytics and other modules:

```swift
// In AuthKit
func signInUser(user: User) {
    // First associate the user with analytics
    Analytics.associateUserWithID(user.uid)
    
    // Then associate with other systems that need identity
    if let notifKit = NotifKit.shared {
        notifKit.associateUserWithID(user.uid)
    }
    
    // Continue with other sign-in processing
}
```

### Session Management Integration

AnalyticsKit provides hooks for critical app lifecycle events:

```swift
class AppDelegate: NSObject, UIApplicationDelegate {
    func applicationWillEnterForeground(_ application: UIApplication) {
        Analytics.capture(.info, id: "app_foregrounded", source: .general)
    }
    
    func applicationDidEnterBackground(_ application: UIApplication) {
        Analytics.capture(.info, id: "app_backgrounded", source: .general)
    }
    
    func applicationWillTerminate(_ application: UIApplication) {
        Analytics.capture(.info, id: "app_terminated", source: .general)
    }
}
```

## Feature Flag Integration

AnalyticsKit integrates with the app's feature flag system to provide data-driven feature rollouts:

```swift
// Check if a feature is enabled for the current user
if Analytics.isFeatureEnabled("new_onboarding_flow") {
    showNewOnboarding()
} else {
    showLegacyOnboarding()
}

// Track feature usage with feature flag context
Analytics.capture(
    .info,
    id: "onboarding_completed",
    longDescription: "Used new flow: \(Analytics.isFeatureEnabled("new_onboarding_flow"))",
    source: .general,
    fromView: "OnboardingView"
)
```

## Error Tracking Pattern

Errors should be tracked consistently across all modules:

```swift
do {
    try await performOperation()
} catch {
    // Capture the error with proper formatting
    Analytics.capture(
        .error,
        id: "operation_failed",
        longDescription: "Error: \(error.localizedDescription)",
        source: .backendKit,
        fromView: "OperationView",
        relevancy: .high // Errors typically have high relevancy
    )
    
    // Handle the error gracefully
    showErrorMessage(error)
}
```

## Performance Monitoring

AnalyticsKit includes methods for tracking performance metrics:

```swift
// Start timing an operation
Analytics.startTiming("image_processing")

// Perform the operation
processImage()

// Stop timing and record the duration
Analytics.stopTiming("image_processing", source: .general)
```

## Developer Mode Integration

The Developer settings screen provides tools for testing analytics:

```swift
struct DeveloperSettingsView: View {
    var body: some View {
        Form {
            Section("Analytics Testing") {
                Button("Capture Test Event") {
                    Analytics.capture(.info, id: "developer_test_event", source: .general)
                }
                
                Button("Simulate Error") {
                    Analytics.capture(.error, id: "simulated_error", source: .general)
                }
                
                Toggle("Show Analytics Debug Overlay", isOn: $showAnalyticsDebug)
                    .onChange(of: showAnalyticsDebug) { newValue in
                        Analytics.setDebugMode(enabled: newValue)
                    }
            }
            
            // Other developer settings sections
        }
    }
}
```

## Privacy Considerations

AnalyticsKit respects user privacy settings and App Store requirements:

```swift
// Check if analytics collection is permitted before tracking
if Analytics.isTrackingPermitted {
    Analytics.capture(.info, id: "sensitive_action", source: .general)
}

// Respect App Tracking Transparency
func requestTrackingPermission() {
    ATTrackingManager.requestTrackingAuthorization { status in
        if status == .authorized {
            Analytics.enableTracking()
        } else {
            Analytics.disableTracking()
        }
    }
}
```

## Best Practices for Analytics Integration

1. **Always initialize analytics first** in the app startup sequence
2. **Use consistent event naming** with snake_case format
3. **Include source information** with each event to categorize by module
4. **Track view activity** automatically with the `.captureViewActivity(as:)` modifier
5. **Log errors with context** including the error message and source
6. **Associate user IDs** across all modules when authentication occurs
7. **Set user properties** for important user attributes
8. **Use predefined constants** for event names, property keys, and values
9. **Group related events** with consistent prefixing (e.g., `purchase_initiated`, `purchase_completed`)
10. **Track feature flag** states when capturing related events

## Troubleshooting Analytics Integration

1. **Events not appearing**: Verify PostHog initialization happens before relevant code execution
2. **User properties missing**: Check that user identification happened with `associateUserWithID`
3. **Incomplete event data**: Ensure proper parameters are provided in `Analytics.capture` calls
4. **Inconsistent naming**: Use the predefined constants for event names and properties
5. **Performance issues**: Avoid capturing too many events in tight loops
6. **Missing view tracking**: Verify `.captureViewActivity(as:)` is applied to navigation containers
7. **Duplicate events**: Check for multiple capture calls in view lifecycles
8. **Privacy conflicts**: Ensure tracking respects user opt-out preferences
