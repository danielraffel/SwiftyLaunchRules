# AnalyticsKit Integration Guidelines

AnalyticsKit provides easy analytics integration for your app through PostHog. When implementing features with AnalyticsKit, follow these guidelines to ensure proper tracking and integration.

## Setup and Configuration

1. **PostHog Configuration**: 
   - Ensure the PostHog API key and host are added to `PostHog-Info.plist` during the initial setup.
   - Do not modify the default PostHog configuration without explicitly asking the user.

2. **Analytics Initialization**:
   - Analytics is initialized automatically via `Analytics.initPostHog()` in `App.swift`.
   - Do not add redundant initialization calls in other parts of the app.

## Capturing Events

1. **Event Capture Format**:
   - Use the `Analytics.capture()` static function for recording events.
   - Follow the parameter structure: eventType, id, longDescription, source, fromView, and relevancy.

   ```swift
   Analytics.capture(
       .success,             // EventType: .info, .error, .success
       id: "event_name",     // Snake case string identifier
       longDescription: "Detailed event description", // Optional
       source: .general,     // EventSource: .general, .auth, .db, etc.
       fromView: "ViewName", // Optional, name of the originating view
       relevancy: .medium    // Optional, defaults to .medium, .error defaults to .high
   )
   ```

2. **Event ID Naming Conventions**:
   - Use descriptive snake_case for event IDs.
   - Prefix with action or result (e.g., `user_logged_in`, `payment_completed`).

3. **Event Types**:
   - Use `.info` for general tracking and informational events.
   - Use `.error` for capturing failures and error conditions.
   - Use `.success` for successful user actions or system operations.

4. **Alternative Event Capture Method**:
   - For more granular control, you can use the `Analytics.captureEvent()` function:
   
   ```swift
   Analytics.captureEvent(
       "success_picture_liked", // Format: (event_type)_(event_name)
       properties: [
           "source": "general",
           "relevancy": "medium",
           "long_description": "User liked picture with ID \(pictureID)",
           "$screen_name": "HomeFeedView" 
       ]
   )
   ```

## View Tracking and User Interactions

1. **Capturing View Activity**:
   - Add the `.captureViewActivity()` modifier to views that should be tracked when they appear.
   - Apply this modifier to root-level container views, not to individual elements.

   ```swift
   struct ContentView: View {
       var body: some View {
           NavigationView {
               // View content
           }
           .captureViewActivity(as: "ContentView")
       }
   }
   ```

2. **Tracking Button and Control Interactions**:
   - Use the `.captureTaps()` modifier to track user interactions with buttons and interactive elements.
   - Apply appropriate event naming for clarity.

   ```swift
   Button("Sign In") {
       // Action code
   }
   .captureTaps("sign_in_button", fromView: "AuthView")
   ```

3. **Manual Tap Capture**:
   - For views that don't support the `.captureTaps()` modifier, use the manual capture method:
   
   ```swift
   Analytics.captureTap(
       "like_button", 
       fromView: "HomeFeedView",
       relevancy: .medium
   )
   ```

## Crash Detection and Error Reporting

1. **Error Tracking**:
   - Use the `.error` event type when capturing errors or exceptions.
   - Include sufficient context in longDescription to help diagnose issues.

   ```swift
   func fetchUserData() async {
       do {
           // Fetch data
       } catch {
           Analytics.capture(
               .error,
               id: "user_data_fetch_failed",
               longDescription: "Failed to fetch user data: \(error.localizedDescription)",
               source: .db
           )
       }
   }
   ```

2. **Crash Reports**:
   - When using AuthKit and DBKit together, crash detection is automatically configured.
   - Do not add manual crash detection logic, as it's handled by the integration with Firebase Crashlytics.
   - The system will automatically track crashes and upload crash reports to Firebase.

## User Identification (with AuthKit)

1. **User Association**:
   - When AuthKit is present, users are automatically associated with their Firebase User ID.
   - Do not manually call `Analytics.associateUserWithID()` or `Analytics.removeUserIDAssociation()` as it's handled by the framework.

2. **Anonymous Events**:
   - For users who aren't signed in, a random anonymous ID is automatically used.
   - Design appropriate fallbacks for features that depend on user identity.

## Session Recording

1. **Configuration**:
   - Session recording is enabled by default in SwiftyLaunch apps.
   - To disable it, set `sessionReplay` to `false` in the PostHog configuration.

2. **Privacy Considerations**:
   - Ensure that sensitive UI elements are masked appropriately.
   - Session recordings should be used in compliance with privacy regulations.
   - Use PostHog masking features to protect user data:
   
   ```swift
   // In PostHog configuration
   config.sessionReplayConfig.maskAllImages = true
   config.sessionReplayConfig.maskAllTextInputs = true
   ```

## Backend Integration (with BackendKit)

1. **Consistent Event Structure**:
   - Use the same event structure on backend code when using BackendKit:

   ```typescript
   Analytics.captureEvent({
     data: {
       eventType: "info",  // "info", "error", "success"
       id: "event_name",   // Snake case identifier
       longDescription: "Detailed description",  // Optional
       source: "general",  // "general", "auth", "db", etc.
       relevancy: "medium" // "low", "medium", "high", Optional
     },
     fromUserID: userId    // Optional, Firebase user ID
   });
   ```

2. **Event Synchronization**:
   - Use consistent event IDs and sources between frontend and backend for related operations.
   - This helps create a complete picture of user flows across client and server.

## Performance Considerations

1. **Event Volume**:
   - Track meaningful events, but avoid excessive tracking that could impact app performance.
   - Be selective about what events need tracking for specific views.

2. **Data Privacy**:
   - Never include sensitive or personal user data in event descriptions or properties.
   - Follow data minimization principles in analytics implementations.

## Testing Analytics

1. **Verify Event Capture**:
   - Manually verify events appear in the PostHog dashboard during development.
   - Include analytics validation in your testing plans.

2. **Debug Mode**:
   - Consider adding a debug flag for development environments to log analytics events locally.
   - This can help diagnose issues before they reach production.

## Common Pitfalls

1. **Using Reserved Properties**:
   - Avoid using PostHog reserved properties (prefixed with `$`) unless you specifically intend to override them.
   - Examples include `$screen_name`, `$user_id`, and `$device_id`.

2. **Missing View Activity**:
   - Ensure all important views have the `.captureViewActivity()` modifier applied.
   - Without this, user navigation patterns will be incomplete in analytics.

3. **Inconsistent Event Naming**:
   - Maintain a consistent naming scheme across your application.
   - Document your event names in a central location for team reference.
