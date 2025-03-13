# SwiftyLaunchRules

The **SwiftyLaunchRules** repository is a collection of markdown files designed to be useful when working with [SwiftyLaunch](http://swiftylaun.ch) and developing with LLMs or AI-powered coding tools like Cursor. It provides a structured set of rules and guidelines currently supporting v1.5.

At the core of this repository is swiftrules.md, which serves as a high-level introduction to SwiftyLaunch. It provides an overview of the project’s structure, guiding principles, and details about the App Module, which is included in all projects.

Beyond this introduction, the repository includes discrete markdown files for each module, allowing AI-assisted development tools to efficiently retrieve relevant information. These files cover essential topics such as integration, error handling, environment-specific code, analytics, deep linking, and more, making them a helpful resource for AI-powered coding assistants.

**Note:** These rules were inspired by [SwiftRules](https://github.com/danielraffel/SwiftRules), which, in turn, was heavily inspired by [Swift Cursor Rules](https://www.rayfernando.ai/swift-cursor-rules) and includes some of its text verbatim.

## Prerequisites and Assumptions

These rules assume you are using the following extensions in Cursor, VSCode, or similar editors:

- **[Sweetpad](https://sweetpad.hyzyla.dev)** – Enables command-line iOS app builds with Xcode.
- **[InjectionIII](https://github.com/johnno1962/InjectionIII)** – Supports hot reloading for SwiftUI views.

These best practices ensure your code remains clean, scalable, and maintainable by utilizing the latest SwiftUI features and adhering to the [VIPER architecture](https://medium.com/@pinarkocak/understanding-viper-pattern-619fa9a0b1f1).

For a walkthrough on setting up Sweetpad and InjectionIII, check out [this video](https://www.youtube.com/watch?v=s7BVmsZSmWQ).

## What is SwiftyLaunch?

SwiftyLaunch is an iOS app generator designed to streamline the development process by creating a new Xcode project with foundational code pre-written. By taking care of repetitive setup work, SwiftyLaunch allows developers to focus on what makes their app unique.

## SwiftyLaunch Modules

SwiftyLaunch offers a collection of powerful, optional modules that developers can select based on their app's specific needs. Below is an overview of the modules along with links to their dedicated rule files:

1. **[App Module](swiftrules.md)** (Included in all projects)  
   - App initialization and module integration  
   - Customizable launch screen  
   - Developer view with feature toggles  
   - Native settings screen  
   - Privacy manifest generation  
   - Onboarding and "What's New" screens

2. **[DatabaseKit](databasekit.md)** (Optional)  
   - Firebase Firestore wrapper  
   - Data management examples  
   - Automatic event tracking  
   - BackendKit integration

3. **[AuthKit](authkit-module.md)** (Optional)  
   - Firebase Authentication integration  
   - Email and Apple Sign-in  
   - Authentication-based view and action protection  
   - Account settings integration  
   - Analytics tracking

4. **[BackendKit](backendkit.md)** (Optional)  
   - Firebase Cloud Functions  
   - Pre-built server-side functions  
   - Client-side backend interaction utilities

5. **[AIKit](aikit.md)** (Optional)  
   - AI-powered mini-app templates  
   - Computer vision  
   - Voice translation  
   - AI chatbot  
   - Secure API key handling

6. **[NotifKit](notifkit-module.md)** (Optional)  
   - OneSignal SDK integration  
   - Push notification management  
   - In-app notification routing  
   - User ID sharing with AuthKit

7. **[AnalyticsKit](analyticskit-rules.md)** (Optional)  
   - PostHog SDK integration  
   - Automatic event tracking  
   - View tracking and session recording  
   - Crash detection and reporting  
   - Learn more in the [SwiftyLaunch Analytics Integration](swiftylaunch-analytics-integration.md) file

8. **[InAppPurchaseKit](inapppurchasekit-module.md)** (Optional)  
   - RevenueCat SDK integration  
   - Customizable paywall  
   - Subscription management  
   - Action and view monetization

9. **[AdsKit](adskit.md)** (Optional)  
   - Google AdMob integration  
   - Native ad banners  
   - Analytics tracking  
   - Premium plan ad removal

10. **[SharedKit](complete-sharedkit-guide.md)** (Included in all projects)  
    - Cross-module utilities  
    - Permission request helpers  
    - In-app review and authentication  
    - Haptic feedback  
    - UI components and design styles

---

## Repository Structure

```
SwiftyLaunchRules/
├── README.md
├── swiftrules.md              # App Module rules
├── databasekit.md             # DatabaseKit rules
├── authkit-module.md          # AuthKit rules
├── backendkit.md              # BackendKit rules
├── aikit.md                   # AIKit rules
├── notifkit-module.md         # NotifKit rules
├── analyticskit-rules.md      # AnalyticsKit rules
├── swiftylaunch-analytics-integration.md   # Detailed analytics integration guidelines
├── inapppurchasekit-module.md  # InAppPurchaseKit rules
├── adskit.md                  # AdsKit rules
├── complete-sharedkit-guide.md# SharedKit rules
└── developer-guidelines.md    # General developer guidelines
```
