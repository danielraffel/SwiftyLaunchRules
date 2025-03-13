## BackendKit

BackendKit integrates Firebase Cloud Functions into your app, providing a robust TypeScript-based backend solution. When implementing features with BackendKit, follow these guidelines to ensure proper setup and integration.

### Setup and Configuration

1. **Firebase Configuration**:
   - Set up Firebase Cloud Functions with the proper billing account (Blaze Plan).
   - Configure the `.firebaserc` file with the correct Firebase project ID.
   - Ensure all API keys are properly stored in the `config.ts` file.

2. **API Key Management**:
   - Store all API keys in the `config.ts` file in your backend code.
   - Never hardcode API keys directly in function implementations.
   - Required keys (based on selected modules):
     - RevenueCat API key (with InAppPurchaseKit)
     - OneSignal App ID and REST API Key (with NotifKit)
     - PostHog API key and host (with AnalyticsKit)
     - OpenAI API key (with AIKit)

3. **Backend Deployment**:
   - Use `firebase deploy --only functions` to deploy your backend functions.
   - Run deployment commands from the `functions` folder.
   - Verify function deployment in the Firebase Console.

### Backend Function Structure

1. **Cloud Function Definition**:
   - Define cloud functions using the `onCall` wrapper for client-callable functions.
   - Export all functions from the main `index.ts` file.

   ```typescript
   export const functionName = onCall(async (request) => {
     // Authentication check (if needed)
     if (!request.auth) {
       throw new Error("Unauthorized");
     }
     
     // Function implementation
     try {
       // Function logic
       return result;
     } catch (error) {
       // Error handling
       throw new Error("Function error");
     }
   });
   ```

2. **Authentication and Security**:
   - Always validate user authentication when security is required:
   ```typescript
   if (!request.auth) {
     throw new Error("Unauthorized");
   }
   ```
   - Use Firebase Auth for user identification and security.
   - Implement role-based access control for sensitive operations.

3. **Error Handling**:
   - Use try/catch blocks for proper error management.
   - Return standardized error responses with meaningful messages.
   - Log errors with AnalyticsKit (if available) for monitoring.

   ```typescript
   try {
     // Function logic
   } catch (error) {
     // Log error with AnalyticsKit
     Analytics.captureEvent({
       data: {
         eventType: "error",
         id: "function_error",
         longDescription: `Error in functionName: ${error.message}`,
         source: "backend"
       }
     });
     throw new HttpsError("internal", "Specific error message");
   }
   ```

### Client-Side Integration

1. **Function Calls**:
   - Call backend functions through the extension of the `DB` class.
   - Implement proper error handling and loading states.

   ```swift
   func callBackendFunction() async throws -> Result {
     do {
       let parameters: [String: Any] = ["key": "value"]
       let data = try await functions.httpsCallable("functionName").call(parameters)
       // Parse response data
       return result
     } catch {
       // Handle error
       throw error
     }
   }
   ```

2. **Data Serialization**:
   - Use proper type conversion between Swift and TypeScript.
   - Create Swift models/structs corresponding to backend data structures.
   - Implement proper decoding/encoding of JSON data.

   ```swift
   struct BackendResponse: Decodable {
     let field1: String
     let field2: Int
     
     init(from dict: [String: Any]) throws {
       guard let field1 = dict["field1"] as? String else {
         throw NSError(domain: "ParsingError", code: 1)
       }
       // Continue parsing other fields
       guard let field2 = dict["field2"] as? Int else {
         throw NSError(domain: "ParsingError", code: 2)
       }
       
       self.field1 = field1
       self.field2 = field2
     }
   }
   ```

3. **Error Handling in Client**:
   - Implement structured error handling for Firebase function calls.
   - Check for specific error codes to provide appropriate user feedback.

   ```swift
   do {
       let data = try await Functions.functions().httpsCallable("functionName").call()
       // Process successful response
   } catch {
       if let error = error as NSError? {
           if error.domain == FunctionsErrorDomain {
               let code = FunctionsErrorCode(rawValue: error.code)!
               switch code {
                   case .invalidArgument: 
                       // Handle invalid argument error
                   case .unauthenticated:
                       // Handle authentication error
                   default: 
                       // Handle other server errors
               }
               let message = error.localizedDescription
               // Display error message to user
           }
       }
   }
   ```

### Module Integrations

1. **With AuthKit**:
   - Use Firebase Authentication to secure backend functions.
   - Access user information via `request.auth.uid`.
   - Validate user sessions and permissions.

2. **With AnalyticsKit**:
   - Track backend events using the backend implementation:
   ```typescript
   Analytics.captureEvent({
     data: {
       eventType: "info",
       id: "event_name",
       longDescription: "Event description",
       source: "backend"
     },
     fromUserID: request.auth?.uid  // Optional
   });
   ```
   
3. **With NotifKit**:
   - Send notifications to users via OneSignal:
   ```typescript
   const response = await Notifications.sendTo({
     userID: "user-id",
     title: "Notification Title",
     body: "Notification content",
     data: { additionalData: "value" }
   });
   ```

4. **With InAppPurchaseKit**:
   - Validate purchases server-side:
   ```typescript
   const hasPremium = await InAppPurchases.doesUserHavePremium(userID);
   ```
   - Manage subscription status changes.

5. **With AIKit**:
   - Process AI requests securely on the backend:
   ```typescript
   const aiResponse = await AIKit.processPrompt(prompt, options);
   ```
   - Keep sensitive AI operations server-side to protect API keys.

### Performance and Scaling

1. **Function Optimization**:
   - Write efficient code to minimize execution time and resources.
   - Use batch operations for Firestore and other database interactions.
   - Implement appropriate caching strategies.

2. **Cold Start Mitigation**:
   - Keep initialization code outside function handlers.
   - Use global variables for reusable resources (like database connections).
   - Structure code to optimize for cold starts.

3. **Error Monitoring**:
   - Implement proper logging for all backend functions.
   - Set up alerts for critical function failures.
   - Use AnalyticsKit to track function performance and errors.

### Testing and Deployment

1. **Local Testing**:
   - Test functions locally before deployment:
   ```bash
   firebase emulators:start
   ```
   - Use Firebase emulators for development.

2. **CI/CD Integration**:
   - Set up continuous deployment for backend functions.
   - Implement testing before deployment.
   - Use separate environments for development, staging, and production.

3. **Versioning**:
   - Implement versioning for backend APIs.
   - Ensure backward compatibility when updating functions.
   - Use feature flags for gradual rollouts.

### Security Best Practices

1. **Data Validation**:
   - Validate all input data before processing.
   - Implement strict type checking.
   - Never trust client-side data without validation.

2. **Least Privilege Principle**:
   - Use the minimal required permissions for service accounts.
   - Restrict database access to only necessary paths.
   - Implement Firestore security rules to complement backend security.

3. **Sensitive Data Handling**:
   - Never log sensitive user data.
   - Encrypt sensitive data at rest and in transit.
   - Follow data protection regulations (GDPR, CCPA, etc.).

### Common Patterns and Examples

1. **Basic Example**: Creating a Simple Cloud Function
   ```typescript
   // In index.ts
   export const helloWorldFunction = onCall(async (request) => {
     return "Hello World!";
   });
   ```

2. **Passing Parameters**: Handling Function Parameters
   ```typescript
   // In index.ts
   export const functionWithParams = onCall(async (request) => {
     const name = request.data?.name as string | null;
     if (!name) {
       throw new HttpsError("invalid-argument", "No Name Provided");
     }
     return `Hello, ${name}!`;
   });
   ```

3. **Calling from Swift**: Client-Side Implementation
   ```swift
   // Calling a backend function
   let contentDict: [String: Any] = ["name": "John"]
   let data = try await Functions.functions().httpsCallable("functionWithParams").call(contentDict)
   
   // Handling the response
   guard let response = data.data as? String else {
     // Error handling
     return
   }
   
   // Use the response
   print(response) // Outputs: "Hello, John!"
   ```

4. **Typed Responses**: Creating Typed Data Models
   ```swift
   struct HelloWorldResponse: Codable {
     let message: String
     let count: Int
     let someBooleanValue: Bool
     
     init(from rawData: [String: Any]) throws {
       guard JSONSerialization.isValidJSONObject(rawData),
           let data = try? JSONSerialization.data(withJSONObject: rawData),
           let decodedResults = try? JSONDecoder().decode(Self.self, from: data)
       else {
           throw NSError(domain: "Invalid Init Object", code: 0, userInfo: nil)
       }
       self = decodedResults
     }
   }
   ```

### Running BackendKit Locally

1. **Local Development Setup**:
   - Install the Firebase CLI tools.
   - Configure your local environment with required variables.
   - Start the Firebase emulators.

2. **Testing with Emulators**:
   - Point your app to the local emulator instead of production.
   - Use the Firebase emulator UI to monitor function execution.
   - Verify function behavior before deployment.

3. **Debugging Tips**:
   - Use `console.log` for debugging in functions.
   - Monitor the emulator logs for errors and function execution.
   - Test different input scenarios to ensure robustness.
