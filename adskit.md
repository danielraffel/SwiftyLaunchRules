## AdsKit

AdsKit integrates Google AdMob to enable monetization through native-looking ad banners. When implementing features with AdsKit, follow these guidelines to ensure proper ad integration that respects user experience.

### Setup and Configuration

1. **AdMob Account Setup**:
   - Create and configure a Google AdMob account before implementing ads.
   - Set up your app in the AdMob dashboard and create a Native Advanced ad unit.
   - Use the AdMob App ID and Ad Unit ID provided by Google in your app configuration.

2. **Info.plist Configuration**:
   - Configure your Info.plist with the required `GADApplicationIdentifier` key set to your AdMob App ID.
   - Add all required `SKAdNetworkItems` entries to comply with iOS privacy requirements.
   - Do not modify these entries as they are required for proper ad network functioning.

3. **Test Mode Configuration**:
   - During development, use test ad unit IDs to prevent accidental ad impressions:
   ```swift
   // Test ad unit ID for Native Advanced
   "ca-app-pub-3940256099942544/3986624511"
   ```
   - Add test device IDs to your configuration to ensure proper testing:
   ```swift
   GADMobileAds.sharedInstance().requestConfiguration.testDeviceIdentifiers = ["YOUR_TEST_DEVICE_ID"]
   ```

### Ad Implementation

1. **Using the AdBanner Component**:
   - Implement ads using the provided `AdBanner` component, which handles the proper display of native ads:
   ```swift
   AdBanner(adUnitID: "YOUR_AD_UNIT_ID")
   ```
   - Never create custom implementations that bypass the `AdBanner` component.
   - The banner takes up the whole width of the parent view and has a height of 75 points.

2. **Ad Banner Placement**:
   - Place ad banners in non-intrusive locations that don't interfere with core app functionality.
   - Don't place ads near UI controls to prevent accidental clicks.
   - Consider using ad banners at natural transition points in your app flow.
   - It is recommended to place banners at the top or bottom of your view to avoid overlapping with other content.
   - Avoid placing ad banners inside a `ScrollView`; keep them outside of scrollable content.

3. **Ad Refresh Frequency**:
   - AdsKit automatically sets a 30-minute minimum interval between ad refreshes.
   - Don't attempt to override this timing to ensure compliance with AdMob policies.
   - Allow the built-in view model to handle the ad refresh cycle.

4. **Ad Content Styling**:
   - The native ad banner styling is handled by AdsKit to maintain a consistent look.
   - Use the default styling provided by `NativeAdBannerView` which is designed to integrate well with your app's UI.
   - Don't apply custom styling directly to ad components.

5. **Ad Unit ID Management**:
   - Always use the "Native Advanced" ad format when creating ad units in Google AdMob.
   - You can use a single Ad Unit ID for all banner ads in your app.
   - The recommended approach is to create unique Ad Unit IDs for each banner placement to better track performance.

### Integration with Other Modules

1. **With AnalyticsKit**:
   - When AnalyticsKit is included, ad interactions are automatically tracked.
   - Two primary events are tracked by default: ad views and ad taps.
   - Don't implement custom ad tracking logic if AnalyticsKit is present.

2. **With InAppPurchaseKit**:
   - When InAppPurchaseKit is included, the `AdBanner` automatically checks for premium subscriptions.
   - Ads will not be displayed to users with an active premium subscription.
   - No additional code is required for this integration as it's handled automatically.
   ```swift
   // Example: Ads are automatically hidden for premium users
   AdBanner(adUnitID: "YOUR_AD_UNIT_ID")
   ```

### Error Handling and Troubleshooting

1. **Missing SKAdNetwork Identifiers**:
   - If you encounter errors about missing SKAdNetwork identifiers, verify that all required entries are in your Info.plist.
   - Clean your project and rebuild if identifiers were added after initial builds.

2. **Test Device Configuration**:
   - When you receive a "To get test ads..." message, add the device ID to your test configuration:
   ```swift
   GADMobileAds.sharedInstance().requestConfiguration.testDeviceIdentifiers = ["DEVICE_ID_FROM_CONSOLE"]
   ```

3. **Account Approval Issues**:
   - If you encounter "Account not approved yet" messages, continue using test ads until approval.
   - Don't release your app to the App Store before your AdMob account is approved.

### Compliance and User Experience

1. **Privacy Requirements**:
   - Include appropriate disclosure in your app's privacy policy about ad tracking.
   - Request App Tracking Transparency permission when required by your implementation.
   - Ensure ads comply with App Store Review Guidelines and AdMob policies.

2. **Ad Loading Performance**:
   - Don't load multiple ad banners simultaneously as this may impact app performance.
   - Let the built-in ad refresh mechanism handle ad loading timing.

3. **Ad Density Guidelines**:
   - Follow Google's ad density guidelines: no more than one ad unit visible on the screen at a time.
   - Don't place ads too close together or in a way that dominates the user interface.

4. **After App Store Release**:
   - Link your App Store listing to your AdMob account after your app is published.
   - Monitor ad performance and user feedback to ensure ads aren't negatively impacting user experience.

### Implementation Examples

1. **Basic Ad Banner Implementation**:
   ```swift
   import SwiftUI
   import AdsKit

   struct ContentView: View {
       var body: some View {
           VStack {
               // Top placement
               AdBanner(adUnitID: "ca-app-pub-3940256099942544/3986624511")
               
               // Main content
               ScrollView {
                   // Your scrollable content here
               }
               
               // Bottom placement
               AdBanner(adUnitID: "ca-app-pub-3940256099942544/3986624511")
           }
       }
   }
   ```

2. **Ad Banner with Premium User Check**:
   ```swift
   import SwiftUI
   import AdsKit
   import InAppPurchaseKit

   struct ContentView: View {
       @EnvironmentObject var purchaseManager: PurchaseManager
       
       var body: some View {
           VStack {
               // Main content
               Text("Your app content")
               Spacer()
               
               // AdsKit automatically checks for premium status when 
               // InAppPurchaseKit is present
               AdBanner(adUnitID: "ca-app-pub-3940256099942544/3986624511")
           }
       }
   }
   ```

3. **Tab View with Ad Banners**:
   ```swift
   import SwiftUI
   import AdsKit

   struct TabViewWithAds: View {
       var body: some View {
           VStack {
               // Ad at top of screen above TabView
               AdBanner(adUnitID: "ca-app-pub-3940256099942544/3986624511")
               
               // Tab content
               TabView {
                   Text("First Tab")
                       .tabItem {
                           Label("First", systemImage: "1.circle")
                       }
                   
                   Text("Second Tab")
                       .tabItem {
                           Label("Second", systemImage: "2.circle")
                       }
               }
           }
       }
   }
   ```
