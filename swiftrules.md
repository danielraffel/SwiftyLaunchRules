# SwiftyLaunch: iOS App Development Accelerator

## What is SwiftyLaunch?
SwiftyLaunch is an iOS app generator designed to streamline the development process by creating a new Xcode project with foundational code pre-written. By taking care of repetitive setup work, SwiftyLaunch allows developers to focus on what makes their app unique.

## SwiftyLaunch Modules
SwiftyLaunch offers a collection of powerful, optional modules that developers can select based on their app's specific needs:

# Project Modules Overview

## 1. [App Module](swiftrules.md) (Included in all projects)
- App initialization and module integration
- Customizable launch screen
- Developer view with feature toggles
- Native settings screen
- Privacy manifest generation
- Onboarding and "What's New" screens

## 2. [DatabaseKit](databasekit.md) (Optional)
- Firebase Firestore wrapper
- Data management examples
- Automatic event tracking
- BackendKit integration

## 3. [AuthKit](authkit-module.md) (Optional)
- Firebase Authentication integration
- Email and Apple Sign-in
- Authentication-based view and action protection
- Account settings integration
- Analytics tracking

## 4. [BackendKit](backendkit.md) (Optional)
- Firebase Cloud Functions
- Pre-built server-side functions
- Client-side backend interaction utilities

## 5. [AIKit](aikit.md) (Optional)
- AI-powered mini-app templates
- Computer vision
- Voice translation
- AI chatbot
- Secure API key handling

## 6. [NotifKit](notifkit-module.md) (Optional)
- OneSignal SDK integration
- Push notification management
- In-app notification routing
- User ID sharing with AuthKit

## 7. [AnalyticsKit](analyticskit-rules.md) (Optional)
- PostHog SDK integration
- Automatic event tracking
- View tracking and session recording
- Crash detection and reporting
- [SwiftyLaunch Analytics Integration](swiftylaunch-analytics-integration.md) explains how AnalyticsKit and other SwiftyLaunch modules integrate with the app's core infrastructure, including analytics initialization, tracking patterns, data consistency, and module interdependencies. Understanding these integration patterns is essential for effective analytics implementation across your app.

## 8. [InAppPurchaseKit](inapppurchasekit-module.md) (Optional)
- RevenueCat SDK integration
- Customizable paywall
- Subscription management
- Action and view monetization

## 9. [AdsKit](adskit.md) (Optional)
- Google AdMob integration
- Native ad banners
- Analytics tracking
- Premium plan ad removal

## 10. [SharedKit](complete-sharedkit-guide.md) (Included in all projects)
- Cross-module utilities
- Permission request helpers
- In-app review and authentication
- Haptic feedback
- UI components and design styles

---

# Developer Guidelines for SwiftyLaunch

This section consolidates all rules and guidelines for integrating, managing, and troubleshooting SwiftyLaunch projects. All content from updated‑swiftrules.md is included below without any condensation so that every detail remains intact.

---

## Environment-Specific Code

1. **Preview Environments**
   ```swift
   // Using the isPreview helper from SharedKit
   struct ContentView: View {
       @State private var userData: UserData
       
       init() {
           if isPreview {
               // Use mock data for previews
               _userData = State(initialValue: UserData.mockUser())
           } else {
               // Use real data in the actual app
               _userData = State(initialValue: UserData())
           }
       }
       
       var body: some View {
           UserProfileView(user: userData)
       }
   }
   ```

2. **Adaptive UI for Platforms**
   ```swift
   // Using the currentPlatform helper from SharedKit 
   var body: some View {
       Stack(currentPlatform == .phone ? .vertical : .horizontal) {
           // Content adapts based on device type
           ImageView()
           DetailView()
       }
       .padding(currentPlatform == .phone ? 16 : 24)
   }
   ```

3. **Conditional Compilation**
   ```swift
   #if os(iOS)
   // iOS-specific implementation
   #elseif os(macOS)
   // macOS-specific implementation
   #endif
   ```

---

## Analytics and Tracking

1. **Event Naming and Consistency**
   - Use descriptive snake_case for event IDs (e.g., `user_signed_up`, `content_viewed`).
   - Prefix with action or result (e.g., `button_tapped`, `payment_completed`).
   - Document event names in a central location for team reference.
   - Use predefined constants for event names, properties, and values.

2. **UI Component Tracking**
   ```swift
   // Track view appearances
   struct ContentView: View {
       var body: some View {
           VStack {
               // View content
           }
           .captureViewActivity(as: "ContentView")
       }
   }
   // Track user interactions
   Button("Sign In") {
       // Action
   }
   .captureTaps("sign_in_button", fromView: "AuthView")
   ```

3. **Analytics Event Example**
   ```swift
   // Client-side tracking
   Analytics.capture(
       .success,             // Event type (.info, .error, .success)
       id: "purchase_completed", // Event ID in snake_case
       longDescription: "User purchased premium plan",
       source: .inAppPurchase,
       fromView: "PurchaseView"
   )
   
   // Server-side tracking (when using BackendKit)
   Analytics.captureEvent({
     data: {
       eventType: "success",
       id: "payment_processed",
       longDescription: "Payment processed for premium plan",
       source: "backend"
     },
     fromUserID: userId
   });
   ```

4. **Error Logging**
   ```swift
   do {
       // Operation that might fail
   } catch {
       Analytics.capture(
           .error,
           id: "data_fetch_failed",
           longDescription: "Failed to fetch user data: \(error.localizedDescription)",
           source: .networking
       )
       // Handle error
   }
   ```

5. **Performance Metrics**
   ```swift
   // Start timing an operation
   Analytics.startTiming("image_processing")
   processImage()
   // Stop timing and record the duration
   Analytics.stopTiming("image_processing", source: .general)
   ```

6. **App Lifecycle Events**
   ```swift
   // App foreground
   Analytics.capture(.info, id: "app_foregrounded", source: .general)
   
   // App background
   Analytics.capture(.info, id: "app_backgrounded", source: .general)
   ```

---

## Deep Linking and External Navigation

1. Create a consistent deep linking structure for your app.
2. Handle links from multiple sources (Universal Links, push notifications, custom URL schemes).
3. Process deep links through the Router layer to maintain VIPER architecture.
4. **Deep Link Handling Example**
   ```swift
   // Define deep link routes
   enum DeepLinkRoute {
       case notification(id: String)
       case userProfile(userId: String)
       case article(id: String)
       case settings(section: SettingsSection?)
       
       // Parse from URL or notification payload
       static func from(url: URL) -> DeepLinkRoute? {
           // Parse URL components and return appropriate route
       }
       
       static func from(notification: [AnyHashable: Any]) -> DeepLinkRoute? {
           // Parse notification payload and return appropriate route
       }
   }
   
   // In AppDelegate or SceneDelegate
   func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
       if let route = DeepLinkRoute.from(url: url) {
           router.handleDeepLink(route: route)
           return true
       }
       return false
   }
   ```
   
5. **Push Notification Example**
   ```swift
   func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse) {
       let userInfo = response.notification.request.content.userInfo
       if let route = DeepLinkRoute.from(notification: userInfo) {
           router.handleDeepLink(route: route)
       }
   }
   ```
   
6. Ensure deep link handling respects authentication state and, if needed, redirects to sign-in. For example:
   ```swift
   func handleDeepLinkWithAuth(route: DeepLinkRoute) {
       if db.authState != .signedIn {
           // Store intended destination
           pendingDeepLink = route
           db.showSignInSheet()
       } else {
           handleDeepLink(route: route)
       }
   }
   
   // After successful sign-in
   func authenticationCompleted() {
       if let pendingRoute = pendingDeepLink {
           handleDeepLink(route: pendingRoute)
           pendingDeepLink = nil
       }
   }
   ```

---

## General Guidelines

1. Use the latest SwiftUI APIs and features whenever possible.
2. Implement `async/await` for asynchronous operations.
3. Write clean, readable, and well-structured code.
4. Follow the VIPER architecture to ensure modularity and scalability.
5. Properly integrate SwiftyLaunch modules with VIPER components based on their primary function (UI components in Views, data processing in Interactors).

---

## VIPER Architecture Overview

Each module in your SwiftUI app should follow the VIPER structure:

### 1. **View**
- **Responsibilities**: Render UI and handle user interactions.
- **Best Practices**:
  - Import the `Inject` framework.
  - Use `@ObserveInjection` for hot reloading.
  
  **Example:**
  ```swift
  import SwiftUI
  import Inject
  
  struct YourViewName: View {
      @ObserveInjection var inject
      @EnvironmentObject var db: DB
      let presenter: YourPresenterProtocol
      
      var body: some View {
          VStack {
              // UI elements here
          }
          .enableInjection()
      }
  }
  ```

### 2. **Interactor**
- **Responsibilities**: Handle business logic and data fetching.
- **Example:**
  ```swift
  protocol YourInteractorProtocol {
      func fetchData() async throws -> [YourEntity]
  }
  
  final class YourInteractor: YourInteractorProtocol {
      private let db: DB
      
      init(db: DB) {
          self.db = db
      }
      
      func fetchData() async throws -> [YourEntity] {
          guard let userId = db.currentUser?.uid else {
              throw AuthError.notAuthenticated
          }
          return try await db.collection("userItems")
              .whereField("userId", isEqualTo: userId)
              .getDocumentsAs(YourEntity.self)
      }
  }
  ```

### 3. **Presenter**
- **Responsibilities**: Prepare data for the View, handle UI updates, and manage authentication state.
- **Example:**
  ```swift
  protocol YourPresenterProtocol: ObservableObject {
      var data: [YourEntity] { get }
      var errorMessage: String? { get }
      var isAuthenticated: Bool { get }
      func loadData() async
      func handleAuthentication()
  }
  
  final class YourPresenter: YourPresenterProtocol {
      @Published private(set) var data: [YourEntity] = []
      @Published private(set) var errorMessage: String? = nil
      @Published private(set) var isLoading: Bool = false
      @Published private(set) var isAuthenticated: Bool = false
      
      private let interactor: YourInteractorProtocol
      private let db: DB
      
      init(interactor: YourInteractorProtocol, db: DB) {
          self.interactor = interactor
          self.db = db
          Task { @MainActor in
              for await state in db.$authState.values {
                  isAuthenticated = (state == .signedIn)
                  if isAuthenticated {
                      await loadData()
                  } else {
                      data = []
                  }
              }
          }
      }
      
      func loadData() async {
          isLoading = true
          do {
              data = try await interactor.fetchData()
              errorMessage = nil
          } catch {
              errorMessage = error.localizedDescription
          }
          isLoading = false
      }
      
      func handleAuthentication() {
          if isAuthenticated {
              Task { await loadData() }
          } else {
              db.showSignInSheet()
          }
      }
  }
  ```

### 4. **Entity**
- **Responsibilities**: Define data models and handle encoding/decoding.
- **Example:**
  ```swift
  struct YourEntity: Identifiable, Codable {
      let id: UUID
      let name: String
      
      // Add custom initialization from backend data when needed
      init?(from dict: [String: Any]) {
          guard let idString = dict["id"] as? String,
                let id = UUID(uuidString: idString),
                let name = dict["name"] as? String else {
              return nil
          }
          self.id = id
          self.name = name
      }
  }
  
  // For Firestore-based apps
  struct FirestoreEntity: Identifiable, Codable {
      @DocumentID var id: String?
      let title: String
      let createdAt: Date
      let userId: String
  }
  ```

### 5. **Router**
- **Responsibilities**: Manage navigation and module creation.
- **Example:**
  ```swift
  protocol YourRouterProtocol {
      func createModule() -> YourViewName
      func showAuthenticationIfNeeded()
      func handleDeepLink(route: DeepLinkRoute)
  }
  
  final class YourRouter: YourRouterProtocol {
      private let db: DB
      
      init(db: DB) {
          self.db = db
      }
      
      func createModule() -> YourViewName {
          let interactor = YourInteractor(db: db)
          let presenter = YourPresenter(interactor: interactor, db: db)
          return YourViewName(presenter: presenter)
      }
      
      func showAuthenticationIfNeeded() {
          if db.authState != .signedIn {
              db.showSignInSheet()
          }
      }
      
      func handleDeepLink(route: DeepLinkRoute) {
          // Navigate to appropriate view based on the deep link route
          switch route {
          case .notification(let id):
              navigateToNotificationDetail(id: id)
          case .userProfile(let userId):
              navigateToUserProfile(userId: userId)
          }
      }
      
      private func navigateToNotificationDetail(id: String) {
          // Navigation implementation
      }
      
      private func navigateToUserProfile(userId: String) {
          // Navigation implementation
      }
  }
  ```

---

## Hot Reloading Setup

To enable hot reloading in all SwiftUI views:
1. **Import the Inject framework** in the `View`.
2. **Add the `@ObserveInjection` property wrapper**.
3. **Use the `.enableInjection()` modifier** in the main body of the view.

**Example:**
```swift
import SwiftUI
import Inject

struct ExampleView: View {
    @ObserveInjection var inject
    var body: some View {
        Text("Hello, VIPER!")
            .enableInjection()
    }
}
```

---

## State Management

1. Use `@Published` properties in the Presenter for state management.
2. Pass dependencies via initializers for clarity and testability.
3. Implement loading and error states.
4. For app-wide services, use `@EnvironmentObject` to share dependencies.

**Example:**
```swift
@main
struct MainApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate
    @StateObject var analytics = Analytics()
    @StateObject var db = DB()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(analytics)
                .environmentObject(db)
        }
    }
}
```

---

## Performance Optimization

1. Use `LazyVStack`, `LazyHStack`, or `LazyVGrid` for large lists or grids to improve performance.
2. Optimize `ForEach` loops by providing stable and unique identifiers.
3. Reduce Firebase function cold starts by optimizing backend calls.
4. Be mindful of app size impact when including large assets (e.g., on-device AI models).
5. Consider network conditions and battery usage for data-intensive operations.
6. Use background threads for computationally intensive tasks.
7. Implement pagination for large datasets to improve loading times and reduce memory usage.
8. Use batch operations for multiple database writes to ensure atomicity and improve performance.
9. Structure database queries to minimize data transfer (fetch only what you need).
10. Implement proper offline support with caching and synchronization:
    - Cache critical data for offline access.
    - Queue operations that require connectivity.
    - Synchronize state when connectivity is restored.

---

## Reusable Components

1. Implement custom view modifiers for shared styling and behavior.
2. Use extensions to add reusable functionality to existing types.
3. Create reusable network and data processing components.
4. Leverage SharedKit utilities for common UI patterns.

**Examples:**
```swift
WebView(url: URL(string: "https://www.apple.com")!)
    .clipShape(RoundedRectangle(cornerRadius: 20))

Stack(currentPlatform == .phone ? .vertical : .horizontal, spacing: 20) {
    // Content here
}

ProfileImage(url: user.profileImageURL, width: 60)

HeroView(
    sfSymbolName: "star.fill",
    title: "Welcome!",
    subtitle: "Let's get started"
)
```

5. Use consistent styling for common UI elements:
   ```swift
   // Primary action button
   Button("Sign Up") { /* action */ }
       .buttonStyle(.cta())
   
   // Secondary action button
   Button("Cancel") { /* action */ }
       .buttonStyle(.secondary())
   
   // Consistent text fields
   TextField("Email", text: $email)
       .textFieldStyle(CommonTextField())
   ```

---

## Accessibility

1. Add accessibility modifiers to all UI elements.
2. Support Dynamic Type for text scaling.
3. Provide clear accessibility labels and hints.

---

## SwiftUI Lifecycle

1. Use the `App` protocol and `@main` for the app entry point.
2. Implement `Scene` for managing app structure.
3. Use lifecycle methods like `onAppear` and `onDisappear`.

---

## Data Flow

1. Use protocols to define communication between VIPER components.
2. Implement proper error handling and data propagation in the Interactor.
3. Follow a consistent approach to data serialization between Swift and backend services.
4. For cross-module data flow, establish clear patterns for data passing and state updates.
5. Handle binary data (images, audio) efficiently with appropriate encodings and compression.
6. Implement real-time data updates with appropriate patterns:
   - Use listeners/observers for real-time database changes.
   - Update published properties to propagate changes to the View.
   - Consider performance implications of real-time listeners.
7. For notification and status monitoring:
   - Use reactive patterns (`publisher`, `for await...in`, observables) to monitor state changes.
   - Update UI asynchronously based on state changes.
   - Maintain consistent state across the application.
   - Monitor permission state changes across app lifecycle transitions:
   ```swift
   // In Presenter
   func monitorPermissionState() {
       // Using Swift Concurrency
       Task {
           for await permission in NotificationCenter.default.notifications(named: UIApplication.didBecomeActiveNotification).values {
               await updatePermissionState()
           }
       }
       
       // Or using Combine
       NotificationCenter.default.publisher(for: UIApplication.didBecomeActiveNotification)
           .sink { [weak self] _ in
               Task {
                   await self?.updatePermissionState()
               }
           }
           .store(in: &cancellables)
   }
   
   private func updatePermissionState() async {
       let hasPermission = await interactor.checkNotificationPermission()
       // Update UI state
   }
   ```
8. For app lifecycle events:
   ```swift
   // Track app state transitions for analytics and state management
   func applicationWillEnterForeground(_ application: UIApplication) {
       Analytics.capture(.info, id: "app_foregrounded", source: .general)
       refreshAppState()  // Refresh data when returning to foreground
   }
   
   func applicationDidEnterBackground(_ application: UIApplication) {
       Analytics.capture(.info, id: "app_backgrounded", source: .general)
       saveAppState()  // Save any pending changes before backgrounding
   }
   ```

---

## Backend Integration

1. Place backend calls within the Interactor layer of VIPER.
2. Implement structured error handling for network and Firebase operations.
3. Follow the SwiftyLaunch module-specific guidance for detailed implementation.
4. Use async/await for all asynchronous API calls.

**Example:**
```swift
// Example of BackendKit integration in an Interactor
func fetchUserData(for userId: String) async throws -> UserData {
    do {
        let parameters: [String: Any] = ["userId": userId]
        let result = try await Functions.functions().httpsCallable("getUserData").call(parameters)
        
        guard let dict = result.data as? [String: Any],
              let userData = UserData(from: dict) else {
            throw NSError(domain: "ParsingError", code: 1)
        }
        
        return userData
    } catch {
        // Log error with AnalyticsKit if available
        throw error
    }
}
```

---

## Security Best Practices

1. Validate all user inputs before processing.
2. Never store sensitive API keys in client-side code.
3. Implement proper authentication checks.
4. Follow data protection regulations.
5. Use server-side validation for sensitive operations.
6. For in-app purchases:
   - Verify purchase receipts server-side when possible.
   - Never grant premium features based solely on client-side checks.
   - Implement proper subscription status caching and verification.
7. For analytics and tracking:
   - Follow consistent event naming conventions across client and server.
   - Never include personal identifiable information (PII) in analytics events.
   - Track meaningful events without excessive logging that could impact performance.
   - Consider privacy regulations when configuring analytics features.
8. For sensitive content:
   - Use biometric authentication when appropriate:
     ```swift
     // Authenticate using Face ID or Touch ID
     if await BiometricAuth.authenticate() {
         // Show sensitive content
     }
     
     // Or use the utility method
     await BiometricAuth.executeIfSuccessfulAuth {
         // Code that runs after successful authentication
     }
     ```
   - Protect sensitive views when the app enters the background:
     ```swift
     // Apply the sensitiveView modifier to hide content when app is in background
     VStack {
         Text("Account balance: $10,000")
         // Other sensitive information
     }
     .sensitiveView()
     
     // With biometric protection
     VStack {
         Text("Account number: 1234-5678-9012-3456")
     }
     .sensitiveView(protectWithBiometrics: true)
     ```
   - Store sensitive information in the keychain, not UserDefaults.

---

## Testing

1. Write unit tests for `Interactor` and `Presenter` logic.
2. Implement UI tests for critical user flows.
3. Use `PreviewProvider` for rapid UI iteration and visual validation.
4. Test backend integrations with Firebase/Supabase emulators.
5. Create mock implementations of services for isolated testing:
   ```swift
   protocol DBServiceProtocol {
       func fetchData() async throws -> [YourEntity]
       func saveData(_ entity: YourEntity) async throws
   }
   
   class MockDBService: DBServiceProtocol {
       var mockData: [YourEntity] = []
       
       func fetchData() async throws -> [YourEntity] {
           return mockData
       }
       
       func saveData(_ entity: YourEntity) async throws {
           mockData.append(entity)
       }
   }
   ```
6. Test error scenarios and edge cases thoroughly.
7. For notification and permission-based features:
   - Test across different initial permission states.
   - Verify behavior in both foreground and background app states.
   - Confirm proper handling of declined permissions.
   - Test proper deep linking from notifications.
8. For analytics testing:
   - Include analytics verification in your test plan.
   - Use analytics debug mode during development:
     ```swift
     // In development/debug builds
     Analytics.setDebugMode(enabled: true)
     ```
   - Create test events to verify capture and transmission:
     ```swift
     // Test event capture
     Analytics.capture(.info, id: "test_event", source: .general)
     ```
   - Verify events appear correctly in the analytics dashboard.

---

## SwiftUI-specific Patterns

1. Use `@Binding` for two-way data flow when needed.
2. Implement custom `PreferenceKey` for child-to-parent communication.
3. Utilize `@Environment` for dependencies shared across multiple views.
4. Use `@EnvironmentObject` for accessing shared services like PurchaseManager or analytics.
5. Wrap third-party UI components in SwiftUI views for consistent styling and behavior.
6. Use SwiftUI's built-in components for system interactions (PhotosPicker, ShareLink).
7. Implement consistent permission handling using modifiers or utility functions.
8. Create reusable view modifiers for common UI patterns:
   ```swift
   // Authentication modifier
   struct RequireLoginModifier: ViewModifier {
       @EnvironmentObject var db: DB
       let onCancel: () -> Void
       
       func body(content: Content) -> some View {
           Group {
               if db.authState == .signedIn {
                   content
               } else {
                   LoginView(onCancel: onCancel)
               }
           }
       }
   }
   
   // Premium access modifier
   struct RequirePremiumModifier: ViewModifier {
       @EnvironmentObject var inAppPurchases: InAppPurchases
       let onCancel: () -> Void
       
       func body(content: Content) -> some View {
           Group {
               if inAppPurchases.isPremium {
                   content
               } else {
                   PaywallView(onCancel: onCancel)
               }
           }
       }
   }
   
   // Analytics view tracking modifier
   struct ViewActivityTrackingModifier: ViewModifier {
       let viewName: String
       
       func body(content: Content) -> some View {
           content
               .onAppear {
                   Analytics.capture(.info, id: "view_appeared", fromView: viewName)
               }
       }
   }
   
   // Squircle shape modifier (iOS app icon style)
   struct SquircleModifier: ViewModifier {
       let width: CGFloat
       
       func body(content: Content) -> some View {
           content
               .frame(width: width, height: width)
               .clipShape(/* squircle implementation */)
       }
   }
   
   // Extensions for easy use
   extension View {
       func requireLogin(db: DB, onCancel: @escaping () -> Void) -> some View {
           self.modifier(RequireLoginModifier(onCancel: onCancel))
       }
       
       func requirePremium(iap: InAppPurchases, onCancel: @escaping () -> Void) -> some View {
           self.modifier(RequirePremiumModifier(onCancel: onCancel))
       }
       
       func trackViewActivity(as viewName: String) -> some View {
           self.modifier(ViewActivityTrackingModifier(viewName: viewName))
       }
       
       func squircle(width: CGFloat) -> some View {
           self.modifier(SquircleModifier(width: width))
       }
       
       func accentBackground(strong: Bool = false) -> some View {
           self.background(Color.accentColor.opacity(strong ? 0.2 : 0.1))
               .cornerRadius(12)
       }
   }
   ```
9. For paywalls and subscription UI:
   - Follow App Store guidelines for in-app purchase UI.
   - Include mandatory elements (restore purchases, terms, etc.).
   - Provide clear pricing information and subscription terms.

---

## Code Style and Formatting

1. Follow Swift naming conventions and style guidelines.
2. Use tools like SwiftLint to enforce consistent code style.
3. Maintain consistent documentation across all modules.

---

## Module Integration Guidelines

When integrating SwiftyLaunch modules:

1. **Module Dependencies**:
   - Be aware of dependencies between modules (e.g., AIKit requires BackendKit).
   - Initialize modules in the correct order to respect dependencies:
     ```swift
     // Proper initialization order in AppDelegate
     Analytics.initPostHog()   // Initialize analytics first
     DB.initFirebase()         // Initialize database services next
     // Other module initializations follow
     ```
   - Consider conditional functionality when optional module pairs are present (e.g., AIKit+AnalyticsKit).
   - Use appropriate abstraction patterns for modules with multiple implementations (e.g., DatabaseKit with Firebase/Supabase).
   - Understand cross-module integrations:
     - AuthKit with InAppPurchaseKit (user identification)
     - AuthKit with NotifKit (user-targeted notifications)
     - AnalyticsKit with all modules (automatic event tracking, error logging)
     - AdsKit with InAppPurchaseKit (hiding ads for premium users)
     - BackendKit with NotifKit (sending push notifications)
     - AuthKit with AnalyticsKit (user identification across systems):
       ```swift
       Auth.auth().addStateDidChangeListener { auth, user in
         if let user = user {
           // Associate user ID with analytics
           Analytics.associateUserWithID(user.uid)
           // Other user identity associations
         } else {
           // Handle signed-out state
           Analytics.removeUserIDAssociation()
         }
       }
       ```
     - SharedKit with other modules (providing utility functions and UI components):
       ```swift
       // Using SharedKit to show a notification after a purchase
       func completePurchase() async {
           do {
               let result = try await inAppPurchases.purchase(productID: "premium")
               if result {
                   // Use haptic feedback for purchase success
                   Haptics.notification(type: .success)
                   
                   // Show in-app notification
                   showInAppNotification(
                       .success,
                       content: .init(
                           title: "Purchase Successful",
                           message: "You now have access to premium features."
                       )
                   )
               }
           } catch {
               // Provide feedback for purchase failure
               Haptics.notification(type: .error)
               showInAppNotification(.error, content: .init(title: "Purchase Failed", message: error.localizedDescription))
           }
       }
       ```

2. **General Principles**:
   - Place modules in the appropriate VIPER layer based on their function:
     - UI components (AdsKit, etc.) in the View layer.
     - Business logic and data processing (BackendKit, DatabaseKit, etc.) in the Interactor layer.
     - User authentication (AuthKit) spanning both Presenter and Interactor layers.
     - Complex modules (AIKit) that span multiple VIPER layers should be integrated at appropriate points.
   - For shared service modules (like DatabaseKit):
     - Initialize them at the app level.
     - Access them via Environment Objects in Views.
     - Wrap access in protocol-based abstractions in Interactors.
   - Follow each module's specific documentation for detailed implementation.
   - Use dependency injection to provide module instances.
   - Respect cross-module dependencies and integrations (e.g., AIKit with BackendKit, AdsKit with InAppPurchaseKit).

3. **Error Handling**:
   - Implement consistent error handling across all modules.
   - Propagate errors from Interactors to Presenters with appropriate context.
   - Use AnalyticsKit to log errors when available:
     ```swift
     do {
         // Operation that might fail
         data = try await interactor.fetchData()
     } catch {
         // Log error with AnalyticsKit
         Analytics.capture(
             .error,
             id: "data_fetch_failed",
             longDescription: "Error fetching data: \(error.localizedDescription)",
             source: .networking
         )
         
         // Update UI state
         errorMessage = error.localizedDescription
     }
     ```

4. **Authentication**
   - Handle authentication state and user sessions appropriately.
   - Protect sensitive actions and views with authentication checks.
   - Inject authentication services into VIPER components.
   ```swift
   db.executeIfSignedIn {
       // Authenticated code here
   }
   ```
   
5. **Data Serialization**
   - Create Swift models that match backend data structures.
   - Implement proper encoding and decoding.
   - Safely handle optional values.

6. **Configuration and Setup**
   - Follow module-specific configuration requirements (Info.plist entries, initialization code).
   - Implement necessary setup code at the correct app lifecycle stages.
   - Use utility functions to access configuration values safely:
     ```swift
     do {
         let apiKey = try getPlistEntry("API_KEY", in: "Service-Info")
     } catch {
         print("Configuration error: \(error.localizedDescription)")
     }
     ```
   - For modules with external dashboards (RevenueCat, OneSignal, Firebase):
     - Maintain consistency between app code and dashboard configurations.
     - Document external configuration requirements.
     - Use appropriate environment settings (development/production).
   - For notification and background capabilities:
     - Configure required capability entitlements (Push Notifications, Background Modes).
     - Include necessary Info.plist entries.
     - Set up authentication certificates and keys as required.

7. **User Experience**
   - Follow module-specific UX guidelines (e.g., ad placement best practices).
   - Maintain consistent UI patterns across the application.
   - Consider how modules interact from a UX perspective.
   - Implement appropriate loading states for asynchronous operations, especially AI-based features.
   - Handle permissions (camera, microphone, photos, notifications) using consistent patterns:
     - Explain value before requesting permission.
     - Use pre-permission dialogs where appropriate.
     - Check permission status before making requests.
     - Provide clear guidance when permissions are declined.
     - Use SharedKit permission utilities:
       ```swift
       // Request camera permission
       askUserFor(.cameraAccess) {
           // Permission granted, take action
           startCamera()
       } onDismiss: {
           // Permission denied, handle gracefully
           showAlternativeOption()
       }
       
       // Require permission for a view
       CameraView()
           .requireCapabilityPermission(
               of: .cameraAccess,
               onSuccess: { /* Handle success */ }
           )
       ```
   - Provide consistent feedback to users:
     - Use haptic feedback for important interactions:
       ```swift
       // Success feedback
       Haptics.notification(type: .success)
       
       // Error feedback
       Haptics.notification(type: .error)
       
       // Button press feedback
       Haptics.impact(style: .medium)
       ```
     - Use in-app notifications for non-blocking feedback:
       ```swift
       // Success notification
       showInAppNotification(
           .success,
           content: .init(
               title: "Document Saved",
               message: "Your changes have been saved."
           )
       )
       
       // Error notification
       showInAppNotification(
           .error,
           content: .init(
               title: "Upload Failed",
               message: "Please check your connection."
           )
       )
       ```
   - For monetization features:
     - Clearly communicate pricing and subscription terms.
     - Implement all required App Store compliance features.
     - Provide seamless upgrade/downgrade paths.
     - Include clear cancellation instructions.
   - For notifications:
     - Respect user preferences and context.
     - Use appropriate notification styles based on content.
     - Implement proper notification handling in both foreground and background states.
     - Customize in-app notification appearance to match your app's design:
       ```swift
       // Backend example (in BackendKit)
       await PushNotifications.sendNotificationToUserWithID({
         userID: userID,
         data: {
           title: "New Message",
           message: messageContent,
           additionalData: {
             inAppSymbol: "bell.fill",
             inAppColor: "#007AFF",
             inAppSize: "normal",
             inAppHaptics: "success"
           }
         }
       })
       ```
     - Implement proper deep linking from notification taps to specific content.

---

## Common Implementation Errors

### Authentication Errors

1. **Authentication State Confusion**
   ```swift
   // Wrong:
   func loadUserData() {
       // This might crash if user isn't authenticated
       let userId = db.currentUser!.uid
       // ...
   }
   
   // Correct:
   func loadUserData() {
       guard let userId = db.currentUser?.uid else { return }
       // ...
   }
   ```

2. **Using Client-Side Authentication as the Only Security Measure**
   ```swift
   // Wrong - Relying only on client-side checks
   view.requireLogin(db: db)
   
   // Right - Also implementing server-side validation
   // In Firebase Rules:
   // match /documents/{docId} {
   //   allow read, write: if request.auth != null && request.auth.uid == resource.data.userId;
   // }
   ```

### In-App Purchase Errors

1. **Failing to Handle Offline Purchase Scenarios**
   ```swift
   // Wrong - No offline handling
   func purchasePremium() async throws {
       try await inAppPurchases.purchase(productID: "premium_monthly")
       // Success assumed
   }
   
   // Right - Proper state synchronization
   func purchasePremium() async throws {
       let result = try await inAppPurchases.purchase(productID: "premium_monthly")
       if result {
           // Purchase confirmed
           updateUIForPremiumState()
       } else {
           // Purchase pending or failed
           handlePendingPurchase()
       }
   }
   ```

2. **Missing Restore Purchases Functionality (Required by App Store)**
   ```swift
   // Missing required functionality
   struct SubscriptionView: View {
       var body: some View {
           VStack {
               // Purchase buttons
               // Missing restore purchases button
           }
       }
   }
   
   // Correct implementation with restore option
   struct SubscriptionView: View {
       var body: some View {
           VStack {
               // Purchase buttons
               Button("Restore Purchases") {
                   Task {
                       try? await inAppPurchases.restorePurchases()
                   }
               }
           }
       }
   }
   ```

### Permission Errors

1. **Requesting Permission Without Context**
   ```swift
   // Wrong - Requesting without explaining why
   func onAppear() {
       PushNotifications.showNotificationsPermissionsSheet()
   }
   
   // Correct - Explaining value before requesting
   func requestPermission() {
       showPermissionExplanation(message: "Enable notifications to stay updated") {
           PushNotifications.showNotificationsPermissionsSheet()
       }
   }
   ```

2. **Failing to Check Permission Status Before Requesting**
   ```swift
   // Wrong - Requesting without checking
   Button("Enable Notifications") {
       PushNotifications.showNotificationsPermissionsSheet()
   }
   
   // Right - Checking status before requesting
   Button("Enable Notifications") {
       Task {
           let hasPermission = await PushNotifications.hasNotificationsPermission()
           if !hasPermission {
               PushNotifications.showNotificationsPermissionsSheet()
           }
       }
   }
   ```

3. **Not Handling Declined Permissions Gracefully**
   ```swift
   // Wrong - No guidance when declined
   if !hasPermission {
       // Silently fail
   }
   
   // Right - Clear guidance on how to proceed
   if !hasPermission {
       showSettingsGuidance(message: "Please enable notifications in Settings")
   }
   ```

---

## Error Handling and Troubleshooting

### General Error Handling Principles
1. Use Swift's native error handling with `do-try-catch` for operations that might fail.
2. Create specific error types and error codes for different categories of errors.
3. Log errors with appropriate context using AnalyticsKit.
4. Present user-friendly error messages while logging detailed information for debugging.
5. Handle anticipated errors gracefully to prevent app crashes.

### SwiftyLaunch Error Codes
SwiftyLaunch modules may produce specific error codes that provide insights into what went wrong. Here are the most common error codes and how to address them:

#### Project Generation Errors
- **Error 102**: Issue creating temporary folders—ensure SwiftyLaunch has required system access.
- **Error 105**: General project generation error—verify all checklist items and try again.
- **Error 106**: Error preparing project template—upgrade SwiftyLaunch to the latest version.
- **Error 107**: Error finalizing the project—try generating a new project.
- **Error 108**: Error moving the project to destination—try a different location.
- **Error 114**: Issue preparing module files—try changing the selected modules.
- **Error 115**: Error formatting project—regenerate the project.
- **Error 116**: Error during project cleanup—generate a new project.
- **Error 117/122**: Issue adding BackendKit—try disabling BackendKit.
- **Error 119/124**: Error decoding resources—reinstall SwiftyLaunch.
- **Error 125/126**: Incompatible module or deployment target—change selections.
- **Error 127/128**: General project generation error—review requirements.
- **Error 129**: Error cleaning project without BackendKit—enable BackendKit.
- **Error 130-132**: Error setting up project assets—regenerate the project.
- **Error 133**: Xcode license agreement not accepted—open Xcode and accept terms.

#### Module-Specific Errors
- **AIKit Errors**: Issues with Whisper AI model—check if the model is installed in Additional Components or disable AIKit.

### Common Runtime Errors

1. **Authentication errors**:
   ```swift
   // Handle authentication failures
   do {
       try await db.signIn(email: email, password: password)
   } catch let error as AuthError {
       switch error {
       case .invalidCredentials:
           showErrorMessage("Invalid email or password")
       case .userDisabled:
           showErrorMessage("Your account has been disabled")
       case .networkError:
           showErrorMessage("Network connection error")
       default:
           // Log unexpected errors
           Analytics.capture(.error, id: "auth_error", longDescription: error.localizedDescription)
           showErrorMessage("Authentication failed: \(error.localizedDescription)")
       }
   }
   ```

2. **Network errors**:
   ```swift
   // Handle network request failures
   do {
       let data = try await networkService.fetchData(from: url)
       processData(data)
   } catch NetworkError.noConnection {
       showOfflineMessage()
       loadCachedData()
   } catch NetworkError.serverError(let statusCode) {
       if statusCode == 429 {
           showRateLimitedMessage()
       } else {
           showServerErrorMessage(statusCode)
       }
   } catch {
       // Handle unexpected errors
       Analytics.capture(.error, id: "network_error", longDescription: error.localizedDescription)
       showGenericErrorMessage()
   }
   ```

3. **Database errors**:
   ```swift
   // Handle database operation failures
   do {
       try await db.collection("users").document(userId).updateData(userData)
   } catch DBError.permissionDenied {
       promptForAuthentication()
   } catch DBError.documentNotFound {
       showDocumentNotFoundMessage()
   } catch {
       // Log unexpected errors
       Analytics.capture(.error, id: "db_update_error", longDescription: error.localizedDescription)
       showDatabaseErrorMessage()
   }
   ```

### Error Recovery Strategies

1. **Graceful degradation**:
   - Provide offline functionality when network is unavailable.
   - Fall back to cached data when fresh data cannot be fetched.
   - Disable features that depend on unavailable services.

2. **Retry mechanisms**:
   ```swift
   // Implement retry with exponential backoff
   func fetchWithRetry(maxRetries: Int = 3) async throws -> Data {
       var retryCount = 0
       var delay = 1.0 // Initial delay in seconds
       
       while true {
           do {
               return try await networkService.fetchData()
           } catch NetworkError.serverError(let statusCode) where statusCode >= 500 {
               // Only retry on server errors (5xx)
               retryCount += 1
               if retryCount >= maxRetries {
                   throw NetworkError.maxRetriesExceeded
               }
               
               // Exponential backoff with jitter
               let jitter = Double.random(in: 0...0.3)
               try await Task.sleep(nanoseconds: UInt64((delay + jitter) * 1_000_000_000))
               delay *= 2 // Exponential backoff
           } catch {
               // Don't retry other errors
               throw error
           }
       }
   }
   ```

3. **User feedback and recovery options**:
   - Provide clear error messages with action steps.
   - Offer retry buttons for transient failures.
   - Guide users to settings/preferences for permission issues.
   - Direct users to support resources for persistent problems.