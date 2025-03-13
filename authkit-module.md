# AuthKit Integration Guidelines

## Overview
AuthKit wraps Firebase Authentication SDK (or Supabase Auth for Supabase projects) to provide complete authentication functionality with minimal boilerplate code. It handles user authentication state tracking, multiple authentication methods, and secure account management.

## Key Features and Components

### Authentication States
- Access and track user authentication state through the `db.authState` property:
  - `.signedOut`: User is not authenticated
  - `.signedIn`: User is fully authenticated
  - `.signedInUnverified`: User is signed in but hasn't verified their email address yet
- Retrieve current user details via the `db.currentUser` property
- Observe changes to authentication state as the properties are `@Published`

### Authentication Methods
The module supports multiple authentication methods:
- **Email/Password**: Complete flow including account creation, verification, and password reset
- **Sign in with Apple**: Full implementation with proper configuration
- All methods are accessible through a unified interface

### User Authentication Actions
- Display sign-in UI with `db.showSignInSheet()`
- Protect functionality with `db.executeIfSignedIn()`
- Lock views with the `.requireLogin()` view modifier
- Show user-specific content with `.visibleOnlyToUserWithID()`

## Implementation Guidelines

### Basic Authentication State Checking
```swift
// Check if user is authenticated
if db.authState == .signedIn {
    // Perform action for authenticated users
}
```

### Protecting Actions with Authentication
```swift
// Recommended approach: Show sign-in UI if user is not authenticated
db.executeIfSignedIn {
    // This code only runs if user is authenticated
    performSecureAction()
}

// With custom behavior
db.executeIfSignedIn(otherwise: .showSignInSheet) {
    performSecureAction()
}

// For user-specific actions
db.executeIfSignedIn(withUserID: specificUserID) {
    // This code only runs if the current user's ID matches specificUserID
}
```

### Protecting Views with Authentication
```swift
struct ProtectedView: View {
    @EnvironmentObject var db: DB
    
    var body: some View {
        // This view will only be shown if user is authenticated
        Text("Protected content")
            .requireLogin(db: db, onCancel: {
                // Handle cancellation (e.g., navigate back)
            })
    }
}

// For user-specific content
struct UserSpecificView: View {
    @EnvironmentObject var db: DB
    
    var body: some View {
        Text("User-specific content")
            .visibleOnlyToUserWithID(specificUserID, db: db, onCancel: {
                // Handle cancellation
            })
    }
}
```

### Authentication Flow Integration
Follow these patterns for a complete authentication experience:

#### Sign In Process
```swift
// Manual sign-in
try await db.signIn(email: email, password: password)

// Show sign-in sheet
db.showSignInSheet()
```

#### Sign Up Process
```swift
// Create new account
try await db.signUp(email: email, password: password)
```

#### Sign Out Process
```swift
// Sign out user
try db.signOut()
```

### User Account Management
Implement these features in your account settings:

#### Account Settings View
```swift
struct AccountSettingsView: View {
    @EnvironmentObject var db: DB
    
    var body: some View {
        List {
            // Account information
            
            Section {
                // Change name
                Button("Change Name") {
                    // Implement name change dialog
                }
                
                // Change password (for email auth)
                if db.currentUserProvider == .email {
                    Button("Change Password") {
                        // Implement password change flow
                    }
                }
                
                // Delete account
                Button("Delete Account", role: .destructive) {
                    // Implement confirmation + deletion flow
                }
                
                // Sign out
                Button("Sign Out", role: .destructive) {
                    try? db.signOut()
                }
            }
        }
        .requireLogin(db: db, onCancel: { /* Navigation handler */ })
    }
}
```

## VIPER Architecture Integration

When using AuthKit with the VIPER architecture:

### View
```swift
struct AuthenticatedView: View {
    @ObserveInjection var inject
    let presenter: YourPresenterProtocol
    @EnvironmentObject var db: DB
    
    var body: some View {
        VStack {
            // Authentication-protected content
        }
        .requireLogin(db: db, onCancel: {
            // Handle navigation
        })
        .enableInjection()
    }
}
```

### Interactor
```swift
protocol AuthenticatedInteractorProtocol {
    func fetchUserData() async throws -> UserData
}

final class AuthenticatedInteractor: AuthenticatedInteractorProtocol {
    private let db: DB
    
    init(db: DB) {
        self.db = db
    }
    
    func fetchUserData() async throws -> UserData {
        // Use db.currentUser to get user information
        guard let user = db.currentUser else {
            throw AuthError.notAuthenticated
        }
        
        // Fetch user-specific data
        return UserData(userId: user.uid, name: user.displayName ?? "User")
    }
}
```

### Presenter
```swift
protocol AuthenticatedPresenterProtocol: ObservableObject {
    var userData: UserData? { get }
    func loadUserData() async
    func signOut()
}

final class AuthenticatedPresenter: AuthenticatedPresenterProtocol {
    @Published private(set) var userData: UserData?
    private let interactor: AuthenticatedInteractorProtocol
    private let db: DB
    
    init(interactor: AuthenticatedInteractorProtocol, db: DB) {
        self.interactor = interactor
        self.db = db
    }
    
    func loadUserData() async {
        // Check authentication state first
        db.executeIfSignedIn {
            Task {
                do {
                    userData = try await interactor.fetchUserData()
                } catch {
                    // Handle error
                }
            }
        }
    }
    
    func signOut() {
        try? db.signOut()
    }
}
```

### Router
```swift
protocol AuthenticationRouterProtocol {
    func createAuthenticatedModule() -> AuthenticatedView
    func showSignIn()
}

final class AuthenticationRouter: AuthenticationRouterProtocol {
    private let db: DB
    
    init(db: DB) {
        self.db = db
    }
    
    func createAuthenticatedModule() -> AuthenticatedView {
        let interactor = AuthenticatedInteractor(db: db)
        let presenter = AuthenticatedPresenter(interactor: interactor, db: db)
        return AuthenticatedView(presenter: presenter)
    }
    
    func showSignIn() {
        db.showSignInSheet()
    }
}
```

## Security Considerations

- **Important**: View and action protection methods (`.requireLogin()`, `.visibleOnlyToUserWithID()`, and `executeIfSignedIn()`) are convenience features but **should not be relied upon as the sole security measures**. Always validate permissions on the server side.

- For sensitive operations (password change, account deletion), use re-authentication:
  ```swift
  showReAuthSheet(db: db, reAuthSheetRef: $reAuthSheetRef) { result in
      switch result {
          case .success:
              // Perform protected operation
          case .canceled:
              // Handle cancellation
          case .forgotPassword:
              // Handle password reset flow
      }
  }
  ```

- For email authentication, enforce email verification to prevent account misuse

## Integration with Other Modules

- **DatabaseKit**: User authentication automatically enforces document ownership and security rules
- **BackendKit**: Access authentication details via `request.auth?.uid`
- **AnalyticsKit**: Authentication events are automatically tracked
- **InAppPurchaseKit**: Firebase user ID shared with RevenueCat for subscription management
- **NotifKit**: Firebase user ID shared with notification provider for user targeting

## Testing Considerations

- Mock the DB object for testing authentication-dependent features
- Test both authenticated and unauthenticated user flows
- Verify proper display of authentication UI and protected content
- Test re-authentication for sensitive operations
