# InAppPurchaseKit Integration Guidelines

## Overview
InAppPurchaseKit integrates with RevenueCat to provide in-app purchase and subscription functionality with minimal code, handling purchase verification, receipt validation, and subscription management automatically.

## Key Features and Components

### Product Configuration
- Configure products in App Store Connect before implementation
- Define products in RevenueCat dashboard and organize into offerings
- Use a consistent product identification strategy across systems

### Initialization and State Management
- InAppPurchaseKit is automatically initialized during app startup
- Products are fetched during initialization and cached for performance
- User identification is synchronized with AuthKit when a user signs in
- Access purchase state through `InAppPurchases.subscriptionState` or the convenience property `InAppPurchases.isPremium`

### Purchase Management
- Initiate purchases with `InAppPurchases.purchase()` with proper error handling
- Implement `InAppPurchases.restorePurchases()` to allow users to recover previous purchases
- Access subscription management with `InAppPurchases.showSubscriptionManagement()`

## Implementation Guidelines

### Basic Subscription State Checking
```swift
// Check if user has premium access
if inAppPurchases.subscriptionState == .subscribed {
    // Perform premium-only action
}

// Or use the convenience property
if inAppPurchases.isPremium {
    // Perform premium-only action
}
```

### Protecting Actions with Premium Check
```swift
// Recommended approach: Show paywall if user doesn't have premium
inAppPurchases.executeIfGotPremium {
    // This code only runs if user has premium access
    performPremiumAction()
}

// With custom behavior
inAppPurchases.executeIfGotPremium(otherwise: .showInAppNotification) {
    performPremiumAction()
}
```

### Protecting Views with Premium Check
```swift
struct PremiumFeatureView: View {
    @EnvironmentObject var inAppPurchases: InAppPurchases
    
    var body: some View {
        // This view will only be shown if user has premium access
        // Otherwise, a paywall will be displayed
        Text("Premium Content")
            .requirePremium(iap: inAppPurchases, onCancel: {
                // Handle cancellation (e.g., navigate back)
            })
    }
}
```

### Initiating a Purchase
```swift
Button("Subscribe Now") {
    Task {
        do {
            let result = try await InAppPurchases.purchase(productID: "premium_monthly")
            if result {
                // Purchase successful
                showSuccessMessage()
            }
        } catch {
            // Handle purchase error
            handlePurchaseError(error)
        }
    }
}
```

### Restoring Purchases
```swift
Button("Restore Purchases") {
    Task {
        do {
            try await InAppPurchases.restorePurchases()
        } catch {
            // Handle restore error
            handleRestoreError(error)
        }
    }
}
```

### Manually Showing the Paywall
```swift
Button("View Premium Options") {
    InAppPurchases.showPaywallSheet()
}
```

## VIPER Architecture Integration

When using InAppPurchaseKit with the VIPER architecture:

### View
```swift
struct PremiumView: View {
    @ObserveInjection var inject
    let presenter: PremiumPresenterProtocol
    @EnvironmentObject var inAppPurchases: InAppPurchases
    
    var body: some View {
        VStack {
            // Premium content
            if inAppPurchases.isPremium {
                PremiumContentView(presenter: presenter)
            } else {
                PaywallView()
            }
        }
        .enableInjection()
    }
}
```

### Interactor
```swift
protocol PremiumInteractorProtocol {
    func purchasePremium(productID: String) async throws -> Bool
    func restorePurchases() async throws
}

final class PremiumInteractor: PremiumInteractorProtocol {
    private let inAppPurchases: InAppPurchases
    
    init(inAppPurchases: InAppPurchases) {
        self.inAppPurchases = inAppPurchases
    }
    
    func purchasePremium(productID: String) async throws -> Bool {
        return try await inAppPurchases.purchase(productID: productID)
    }
    
    func restorePurchases() async throws {
        try await inAppPurchases.restorePurchases()
    }
}
```

### Presenter
```swift
protocol PremiumPresenterProtocol: ObservableObject {
    var isPremium: Bool { get }
    func purchasePremium() async
    func restorePurchases() async
}

final class PremiumPresenter: PremiumPresenterProtocol {
    @Published private(set) var isPremium: Bool = false
    private let interactor: PremiumInteractorProtocol
    private let inAppPurchases: InAppPurchases
    
    init(interactor: PremiumInteractorProtocol, inAppPurchases: InAppPurchases) {
        self.interactor = interactor
        self.inAppPurchases = inAppPurchases
        self.isPremium = inAppPurchases.isPremium
    }
    
    func purchasePremium() async {
        do {
            _ = try await interactor.purchasePremium(productID: "premium_monthly")
            DispatchQueue.main.async {
                self.isPremium = self.inAppPurchases.isPremium
            }
        } catch {
            // Handle error
        }
    }
    
    func restorePurchases() async {
        do {
            try await interactor.restorePurchases()
            DispatchQueue.main.async {
                self.isPremium = self.inAppPurchases.isPremium
            }
        } catch {
            // Handle error
        }
    }
}
```

### Router
```swift
protocol PremiumRouterProtocol {
    func createPremiumModule() -> PremiumView
    func showPaywall()
}

final class PremiumRouter: PremiumRouterProtocol {
    private let inAppPurchases: InAppPurchases
    
    init(inAppPurchases: InAppPurchases) {
        self.inAppPurchases = inAppPurchases
    }
    
    func createPremiumModule() -> PremiumView {
        let interactor = PremiumInteractor(inAppPurchases: inAppPurchases)
        let presenter = PremiumPresenter(interactor: interactor, inAppPurchases: inAppPurchases)
        return PremiumView(presenter: presenter)
    }
    
    func showPaywall() {
        InAppPurchases.showPaywallSheet()
    }
}
```

## Paywall Implementation

The paywall is a customizable UI component that is shown in the following scenarios:

- When a user tries to access a view protected with `.requirePremium()` view modifier
- When a user tries to perform an action protected with `executeIfGotPremium()`
- When the user manually navigates to premium settings or calls `InAppPurchases.showPaywallSheet()`

### Paywall Structure
The paywall consists of two main parts:
1. **Upper Part (Carousel)**: A SwiftUI carousel that showcases the benefits of the premium plan
2. **Lower Part (Paywall Footer)**: Implemented using RevenueCat's Paywall SDK, customizable through the RevenueCat dashboard

### Customizing the Paywall
- The upper carousel can be modified directly in code in `InAppPurchaseView.swift`
- The footer is configured in the RevenueCat dashboard, allowing you to update the paywall without submitting a new app version

## Integration with Other Modules

- **AuthKit**: User authentication enables purchases to persist across devices
- **AnalyticsKit**: Purchase events automatically tracked
- **BackendKit**: Verify subscription status server-side with RevenueCat webhooks
- **Settings**: Premium status management UI is automatically included in app settings

## Testing and Compliance

### Testing
- Use RevenueCat's sandbox environment for testing
- Use StoreKit configuration files for local testing
- Implement proper test coverage for purchase logic

### App Store Compliance
- Restore purchases functionality is included as required by App Store guidelines
- Terms of service and privacy policy links are displayed
- Proper subscription management UI is provided
- StoreKit 2 is used for improved security and reliability

## Error Handling

Handle these common error scenarios in your purchase flows:
- Network connectivity issues during purchase
- Payment processing failures
- User cancellation during purchase flow
- Receipt validation failures

## Offline Support

The module includes these offline capabilities:
- Cache entitlement status for offline access
- Queue purchase requests when offline
- Synchronize state when connection is restored