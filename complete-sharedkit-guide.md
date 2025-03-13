# SwiftyLaunch SharedKit Utilities & Helpers

This document explains the various helper functions, utilities, and components available in the SharedKit module to facilitate common development tasks across your SwiftyLaunch app.

## Environment Detection

### Detecting SwiftUI Previews

SwiftUI Previews are invaluable during development, but sometimes you need to provide different behavior or mock data when running in preview mode.

```swift
public var isPreview: Bool {
    return ProcessInfo.processInfo.environment["XCODE_RUNNING_FOR_PREVIEWS"] == "1"
}
```

This global variable returns `true` when your code is being executed in a SwiftUI preview environment, making it easy to provide preview-specific implementations.

#### Usage Example

```swift
import SharedKit
import SwiftUI

struct UserProfileView: View {
    @State private var userData: UserData
    @EnvironmentObject var db: DatabaseManager
    
    init() {
        if isPreview {
            // Use mock data for previews
            _userData = State(initialValue: UserData.mockUser())
        } else {
            // Use actual user data in the real app
            _userData = State(initialValue: db.currentUser)
        }
    }
    
    var body: some View {
        VStack {
            Text("Welcome \(userData.name)")
            // Rest of the view
        }
    }
}
```

### Identifying Current Platform

As SwiftyLaunch supports multiple platforms, you'll often need to adapt your UI based on the device type:

```swift
public var currentPlatform: Platform {
    return UIDevice.current.userInterfaceIdiom == .pad ? .pad : .phone
}

public enum Platform {
    case phone
    case pad
}
```

This helps create responsive layouts that behave differently on iPhone versus iPad.

#### Usage Example

```swift
import SharedKit
import SwiftUI

struct CameraView: View {
    var body: some View {
        // VStack on iPhone, HStack on iPad for better space utilization
        Stack(currentPlatform == .phone ? .vertical : .horizontal, spacing: 25) {
            PreviewView()
            
            // Control buttons are horizontal on phone, vertical on iPad
            Stack(currentPlatform == .phone ? .horizontal : .vertical) {
                Button("Capture") { /* capture action */ }
                Button("Switch") { /* switch camera */ }
            }
        }
    }
}
```

## Configuration Access

### Property List Access

SwiftyLaunch stores module configurations in property list files. To easily access these values:

```swift
public func getPlistEntry(
    _ plistEntry: String,
    in plistName: String
) throws -> String { 
    // Implementation details
}

public enum PlistReadError: String, LocalizedError {
    case noPlist = "PlistReadError: No .plist file found"
    case noEntry = "PlistReadError: No entry found in Property List"
    
    public var errorDescription: String? { self.rawValue }
}
```

This function provides a standardized way to access configuration values stored in property lists.

#### Usage Example

```swift
import SharedKit

func initializeService() {
    do {
        // Get the API key from the property list
        let apiKey = try getPlistEntry("SERVICE_API_KEY", in: "Service-Info")
        
        // Initialize the service with the API key
        ServiceManager.initialize(apiKey)
    } catch {
        // Handle the error
        print("Failed to initialize service: \(error.localizedDescription)")
    }
}
```

## User Feedback

### Haptic Feedback

Haptics provide tactile feedback to the user, enhancing the interactive experience of your app:

```swift
public class Haptics {
    // For feedback after an action is completed
    static public func notification(type: UINotificationFeedbackGenerator.FeedbackType) {
        // Implementation
    }
    
    // For feedback during user interaction
    static public func impact(style: UIImpactFeedbackGenerator.FeedbackStyle) {
        // Implementation
    }
}
```

#### Usage Example

```swift
import SharedKit

// Success feedback
func completePurchase() {
    // Process purchase
    if purchaseSuccessful {
        Haptics.notification(type: .success)
        showSuccessMessage()
    } else {
        Haptics.notification(type: .error)
        showErrorMessage()
    }
}

// Interactive feedback for button presses
Button("Submit") {
    Haptics.impact(style: .medium)
    submitForm()
}
```

### In-App Notifications

SwiftyLaunch provides a system for displaying non-intrusive notifications that appear at the top of the screen:

```swift
public func showInAppNotification(
    _ type: InAppNotificationType,
    content: InAppNotificationContent,
    size: InAppNotificationStyle.NotificationSize? = nil,
    onTapAction: (() -> Void)? = nil
) { }

public func showInAppNotification(
    content: InAppNotificationContent,
    style: InAppNotificationStyle,
    onTapAction: (() -> Void)? = nil
) { }
```

The first function uses predefined styles for common notification types (success, error, warning, info), while the second allows complete customization of the notification appearance.

#### Usage Example

```swift
import SharedKit

// Using predefined types
func saveDocument() {
    if successful {
        showInAppNotification(
            .success,
            content: .init(
                title: "Document Saved",
                message: "Your document has been saved successfully."
            )
        )
    } else {
        showInAppNotification(
            .error,
            content: .init(
                title: "Save Failed",
                message: "Unable to save your document. Please try again."
            )
        )
    }
}

// Using custom styling
func showCustomNotification() {
    showInAppNotification(
        content: .init(
            title: "New Message",
            message: "You have a new message from your team."
        ),
        style: .init(
            sfSymbol: "message.fill",
            symbolColor: .blue,
            size: .normal,
            hapticsOnAppear: .success
        )
    )
}
```

You can customize the appearance duration and maximum number of notifications in `Constants.swift`:

```swift
public struct Constants {
    public struct InAppNotifications {
        public static let maxShownAtOnce: Int = 3
        public static let showingDuration: Int = 6
    }
}
```

## UI Components

### WebView Component

Since SwiftUI doesn't provide a native web view, SharedKit includes a `WebView` component that wraps WKWebView for displaying web content:

```swift
public struct WebView: UIViewRepresentable {
    public init(url: URL) { }
}
```

Usage example:

```swift
import SharedKit
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Text("Apple's Website:")
                .font(.title)
            WebView(url: URL(string: "https://www.apple.com")!)
                .clipShape(RoundedRectangle(cornerRadius: 20))
        }
        .padding(20)
    }
}
```

You can also show a web view in a sheet using the `webViewSheet()` modifier:

```swift
struct ContentView: View {
    @State private var showSheet = false
    
    var body: some View {
        Button("Show Apple's Website") {
            showSheet.toggle()
        }
        .webViewSheet(
            $showSheet,
            url: URL(string: "https://www.apple.com")!,
            title: "Apple's Website"
        )
    }
}
```

### HEVC Video with Transparency

For efficient animations with transparency, SwiftyLaunch provides a component to play HEVC videos with alpha channel:

```swift
public struct VideoPlayerAlphaView: UIViewRepresentable {
    public init(
        url: URL,
        repeatCount: RepeatCount = .never,
        keepsAspectRatio: Bool = true
    ) { }
}

public enum RepeatCount: Equatable {
    case infinite       // Will keep looping
    case constant(Int)  // How many times it should play, so `.constant(7)` will play 7 times
    case never          // Will play only once, equivalent to `.constant(1)`
}
```

This is more efficient than GIFs and allows for high-quality animations with transparency.

#### Usage Example

```swift
import SharedKit
import SwiftUI

struct OnboardingView: View {
    var body: some View {
        VStack {
            // Play a welcome animation with transparency
            VideoPlayerAlphaView(
                url: Bundle.main.url(forResource: "welcome-animation", withExtension: "mov")!,
                repeatCount: .infinite,
                keepsAspectRatio: true
            )
            .frame(height: 200)
            
            Text("Welcome to the App!")
                .font(.title)
                .padding()
            
            // Onboarding content
        }
    }
}
```

### Flexible Stack View

SwiftyLaunch provides a `Stack` view that can be either a `VStack`, `HStack`, or `ZStack` depending on the parameters you provide:

```swift
public struct Stack<Content: View>: View {
    public init(
        _ stackType: StackType,
        spacing: CGFloat? = nil,
        @ViewBuilder content: @escaping () -> Content
    ) {
        self.stackType = stackType
        self.spacing = spacing
        self.content = content
    }
}
```

This allows you to create adaptive layouts that respond to different device types:

```swift
// VStack on iPhone, HStack on iPad
Stack(currentPlatform == .phone ? .vertical : .horizontal, spacing: 25) {
    // Content
    CameraView()
    
    // VStack on iPad, HStack on iPhone
    Stack(currentPlatform == .phone ? .horizontal : .vertical) {
        Button("Capture") { /* capture action */ }
        Button("Switch") { /* switch camera */ }
    }
}
```

### Custom Toggle Style

SwiftyLaunch includes a checkbox-style toggle implementation:

```swift
Toggle("Terms and Conditions", isOn: $agreedToTerms)
    .toggleStyle(CheckToggleStyle())
```

This provides a more visually distinctive alternative to the standard toggle, which is useful for agreements, feature selections, and other boolean options.

### Button Styles

SwiftyLaunch provides two main button styles for common UI patterns:

1. **CTA Button Style**: For primary action buttons that should grab the user's attention
2. **Secondary Button Style**: For less prominent actions that should blend more with the background

#### CTA Button Style

For primary actions, use the `.cta()` button style:

```swift
Button("Sign Up") { /* action */ }
    .buttonStyle(.cta())
```

This fills the button with your app's accent color and spans the full width of its container.

> **Note**: It's best to use only one CTA button per screen to maintain its impact.

For destructive actions, combine with the `.destructive` role:

```swift
Button("Delete", role: .destructive) { /* action */ }
    .buttonStyle(.cta())
```

#### Secondary Button Style

For secondary actions, use the `.secondary()` button style:

```swift
Button("Cancel") { /* action */ }
    .buttonStyle(.secondary())
```

This creates a button with your app's secondary color (typically a light gray) with accent-colored text.

You can also use this with the `.destructive` role:

```swift
Button("Remove", role: .destructive) { /* action */ }
    .buttonStyle(.secondary())
```

### Hero View Component

HeroView is a component that contains a big, accent-colored SF Symbol icon, a title, and a subtitle:

```swift
public struct HeroView: View {
    public init(
        sfSymbolName: String,
        title: LocalizedStringKey,
        subtitle: LocalizedStringKey? = nil,
        size: HeroViewSize = .large,
        bounceOnAppear: Bool = false
    ) {
        // Implementation
    }
}
```

Usage example:

```swift
HeroView(
    symbol: "theatermask.and.paintbrush.fill",
    title: "Welcome to SwiftyLaunch!",
    subtitle: "The best way to build your app idea."
)
```

### Profile Image Component

ProfileImage displays an image in a circular frame, and shows a placeholder if the URL fails to load:

```swift
public struct ProfileImage: View {
    public init(url: URL?, width: CGFloat) {
        self.url = url
        self.width = width
    }
}
```

Usage example:

```swift
ProfileImage(url: URL(string: "https://picsum.photos/id/237/200/300")!, width: 100)
```

### TextField Style

For a consistent appearance of text fields, use the `CommonTextField()` style:

```swift
TextField("Test TextField", text: $textValue)
    .textFieldStyle(CommonTextField())
```

You can also create disabled text fields:

```swift
TextField("Test TextField", text: $textValue)
    .textFieldStyle(CommonTextField(disabled: true))
```

You can customize the default height, radius, and font of the text fields by modifying these constants in the `CommonTextField.swift` file:

```swift
let commonTextFieldRadius = 10.0
let commonTextFieldHeight = 50.0
let commonTextFieldFontStyle: Font = .system(.body, weight: .regular)
```

## View Modifiers

### Sensitive View Modifier

When displaying sensitive information, you can hide the content when the app is in the background:

```swift
extension View {
    public func sensitiveView(protectWithBiometrics: Bool = false) -> some View {
        modifier(SensitiveViewModifier(protectWithBiometrics: protectWithBiometrics))
    }
}
```

Simply apply it to any view containing sensitive content:

```swift
VStack {
    Text("This is sensitive information")
    // Other sensitive content
}
.sensitiveView()
```

With biometric protection:

```swift
VStack {
    Text("This is sensitive information")
    // Other sensitive content
}
.sensitiveView(protectWithBiometrics: true)
```

When using the biometric protection option, the view will be blurred and display a lock icon. When the view becomes active, it will prompt for biometric authentication (Face ID or Touch ID) and will only reveal the content upon successful authentication.

### Squircle Modifier

Create iOS app icon style rounded squares (squircles) with this modifier:

```swift
extension View {
    public func squircle(width: CGFloat) -> some View {
        modifier(Squircle(width: width))
    }
}
```

Usage example:

```swift
Image("AppIcon-Preview")
    .resizable()
    .squircle(width: 150)
```

### Accent Background Modifier

Apply the app's accent color as a background with varying intensity:

```swift
extension View {
    public func accentBackground(strong: Bool = false) -> some View {
        // Implementation
    }
}
```

Apply to any view:

```swift
VStack {
    Text("Highlighted Content")
}
.accentBackground()

// For stronger accent
Text("Important")
    .accentBackground(strong: true)
```

## Biometric Authentication

SharedKit provides built-in support for biometric authentication (Face ID and Touch ID) through the `BiometricAuth` class.

> **Note**: You must add the `NSFaceIDUsageDescription` key to your app's Info.plist file with a description that explains why your app needs to use Face ID.

### Basic Usage

To check if the user can be authenticated using biometrics:

```swift
import SharedKit

func printSecretEmoji() async {
    if (await BiometricAuth.authenticate()) {
        print("😎")
    }
}
```

### Executing Code on Successful Authentication

You can also use the `executeIfSuccessfulAuth()` method to run code only if authentication succeeds:

```swift
import SharedKit

func printSecretEmoji() async {
    await BiometricAuth.executeIfSuccessfulAuth {
        print("😎") // called on a successful authentication
    } otherwise: {
        print("🤨") // called on a failed authentication
    }
}
```

### Protecting Views with Biometric Authentication

As shown in the Sensitive View Modifier section, you can protect entire views with biometric authentication:

```swift
import SwiftUI
import SharedKit

struct SecretContentView: View {
    var body: some View {
        VStack {
            Text("Some Sensitive Information")
            // Other content that requires authentication
        }
        .sensitiveView(protectWithBiometrics: true)
    }
}
```

## Device Permissions

### Request Types

The `RequestType` enum defines the different capabilities the user can be asked to grant access to:

```swift
public enum RequestType: Identifiable {
    case photosAccess
    case cameraAccess
    case microphoneAccess
    case locationAccess
    case contactsAccess
    case calendarAccess
    case remindersAccess
}
```

### Asking for Permissions

To request permission for a capability:

```swift
askUserFor(.microphoneAccess, executeIfGotAccess: {
    // Handle successful permission grant
}, onDismiss: {
    // Handle permission denial
})
```

### Requiring Permissions for a View

You can require permission before displaying a view:

```swift
CameraView()
    .requireCapabilityPermission(
        of: .cameraAccess,
        onSuccess: {
            // Got camera access
        }
    )
```

### Opening App Settings

When permission has been denied, direct users to the system settings:

```swift
Button("Open Settings") {
    openAppInSettings()
}
```

## App Styling

SwiftyLaunch apps use accent colors extensively. You can change the accent color in `Assets.xcassets` by modifying the `AccentColor` asset.

### Button Styles

Two predefined button styles are available:

```swift
// Primary action button
Button("Confirm") { }
    .buttonStyle(.cta())

// Secondary action button
Button("Cancel") { }
    .buttonStyle(.secondary())
```

## Best Practices for Using SharedKit Utilities

1. **Environment Detection**:
   - Use `isPreview` when you need different data or behavior in preview versus runtime
   - Use `currentPlatform` to create adaptive layouts that work well on different device types

2. **Configuration**:
   - Keep sensitive information in property lists and access them with `getPlistEntry`
   - Handle potential errors with proper try-catch blocks

3. **User Feedback**:
   - Use haptics sparingly to avoid overwhelming the user
   - Choose the appropriate haptic type based on the action (notification vs. impact)
   - Keep in-app notifications concise and use the appropriate type (success, error, warning, info)

4. **UI Components**:
   - Use `VideoPlayerAlphaView` instead of GIFs for better performance
   - Consider memory usage when using videos, especially on older devices

5. **Permissions**:
   - Be clear about why your app needs each permission
   - Provide alternatives when permissions are denied
   - Use the built-in permission flow to ensure consistent UX

## Integration Patterns

SharedKit utilities are designed to integrate with other SwiftyLaunch modules:

1. **AnalyticsKit**: Capture analytics events when showing notifications or important UI states
2. **NotifKit**: Route push notifications to in-app notifications for a consistent user experience
3. **AuthKit**: Use environment detection to bypass authentication in previews
4. **InAppPurchaseKit**: Display success/error notifications after purchase attempts

By leveraging these utilities consistently across your app, you'll create a cohesive user experience that feels polished and responsive.
