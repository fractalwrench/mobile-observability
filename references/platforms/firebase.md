# Firebase Integration Guide

Firebase observability setup and best practices for iOS, Android, and Flutter mobile apps.

## Quick Start

### iOS (Swift)

```bash
# Swift Package Manager
dependencies: [
    .package(url: "https://github.com/firebase/firebase-ios-sdk.git", from: "10.0.0")
]
```

```swift
// AppDelegate.swift
import Firebase
import FirebaseCrashlytics
import FirebaseAnalytics
import FirebasePerformance

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Initialize Firebase
    FirebaseApp.configure()

    // Crashlytics setup
    Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(true)

    // Performance Monitoring (automatic)
    Performance.sharedInstance()

    // Analytics (automatic collection)
    Analytics.setAnalyticsCollectionEnabled(true)
    Analytics.setUserProperty(AppConfig.environment, forName: "environment")

    return true
}
```

### Android (Kotlin)

```kotlin
// build.gradle.kts (project)
plugins {
    id("com.google.gms.google-services") version "4.4.0" apply false
    id("com.google.firebase.crashlytics") version "2.9.9" apply false
    id("com.google.firebase.firebase-perf") version "1.4.2" apply false
}

// build.gradle.kts (app)
plugins {
    id("com.google.gms.google-services")
    id("com.google.firebase.crashlytics")
    id("com.google.firebase.firebase-perf")
}

dependencies {
    implementation(platform("com.google.firebase:firebase-bom:32.7.0"))
    implementation("com.google.firebase:firebase-crashlytics-ktx")
    implementation("com.google.firebase:firebase-analytics-ktx")
    implementation("com.google.firebase:firebase-perf-ktx")
    implementation("com.google.firebase:firebase-config-ktx")
}
```

```kotlin
// Application.kt
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Initialize Firebase (automatic via google-services.json)
        Firebase.crashlytics.setCrashlyticsCollectionEnabled(true)

        // Set user properties
        Firebase.analytics.setUserProperty("environment", BuildConfig.BUILD_TYPE)
    }
}
```

### Flutter

```yaml
# pubspec.yaml
dependencies:
  firebase_core: ^2.24.0
  firebase_crashlytics: ^3.4.8
  firebase_analytics: ^10.7.4
  firebase_performance: ^0.9.3
  firebase_remote_config: ^4.3.8
```

```dart
// main.dart
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_crashlytics/firebase_crashlytics.dart';
import 'package:firebase_analytics/firebase_analytics.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp();

  // Crashlytics setup
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };

  runApp(MyApp());
}
```

---

## Key Features

### Crashlytics - Crash Reporting

#### Custom Keys (Context)

```swift
// iOS - Add context before crash
Crashlytics.crashlytics().setCustomValue("checkout", forKey: "feature")
Crashlytics.crashlytics().setCustomValue(cartItems.count, forKey: "cart_size")
Crashlytics.crashlytics().setCustomValue(user.isPremium, forKey: "is_premium")
```

```kotlin
// Android - Add context
Firebase.crashlytics.setCustomKey("feature", "checkout")
Firebase.crashlytics.setCustomKey("cart_size", cartItems.size)
Firebase.crashlytics.setCustomKey("is_premium", user.isPremium)
```

```dart
// Flutter - Add context
FirebaseCrashlytics.instance.setCustomKey("feature", "checkout");
FirebaseCrashlytics.instance.setCustomKey("cart_size", cartItems.length);
```

#### Custom Logs (Breadcrumbs)

```swift
// iOS - Log breadcrumbs
Crashlytics.crashlytics().log("User viewed product: \(productId)")
Crashlytics.crashlytics().log("Added item to cart")
Crashlytics.crashlytics().log("Initiated checkout")
```

```kotlin
// Android - Log breadcrumbs
Firebase.crashlytics.log("User viewed product: $productId")
Firebase.crashlytics.log("Added item to cart")
Firebase.crashlytics.log("Initiated checkout")
```

#### User Identification

```swift
// iOS - Set user ID
Crashlytics.crashlytics().setUserID("user_123")

// Clear on logout
Crashlytics.crashlytics().setUserID(nil)
```

```kotlin
// Android - Set user ID
Firebase.crashlytics.setUserId("user_123")

// Clear on logout
Firebase.crashlytics.setUserId(null)
```

#### Non-Fatal Exceptions

```swift
// iOS - Report recoverable errors
do {
    try riskyOperation()
} catch {
    Crashlytics.crashlytics().record(error: error)
}

// Force a fatal crash for testing
Crashlytics.crashlytics().forceCrashException()
```

```kotlin
// Android - Report recoverable errors
try {
    riskyOperation()
} catch (e: Exception) {
    Firebase.crashlytics.recordException(e)
}

// Force a fatal crash for testing
Firebase.crashlytics.forceCrash()
```

### Analytics - Event Tracking

#### Standard Events

```swift
// iOS - Log recommended events
Analytics.logEvent(AnalyticsEventSelectContent, parameters: [
    AnalyticsParameterContentType: "product",
    AnalyticsParameterItemID: "SKU_12345"
])

Analytics.logEvent(AnalyticsEventPurchase, parameters: [
    AnalyticsParameterCurrency: "USD",
    AnalyticsParameterValue: 29.99,
    AnalyticsParameterTransactionID: "T_12345"
])

Analytics.logEvent(AnalyticsEventAddToCart, parameters: [
    AnalyticsParameterItemID: "SKU_12345",
    AnalyticsParameterItemName: "Blue Widget",
    AnalyticsParameterQuantity: 1
])
```

```kotlin
// Android - Log recommended events
Firebase.analytics.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT) {
    param(FirebaseAnalytics.Param.CONTENT_TYPE, "product")
    param(FirebaseAnalytics.Param.ITEM_ID, "SKU_12345")
}

Firebase.analytics.logEvent(FirebaseAnalytics.Event.PURCHASE) {
    param(FirebaseAnalytics.Param.CURRENCY, "USD")
    param(FirebaseAnalytics.Param.VALUE, 29.99)
    param(FirebaseAnalytics.Param.TRANSACTION_ID, "T_12345")
}
```

#### Custom Events

```swift
// iOS - Custom events (up to 500 types)
Analytics.logEvent("level_completed", parameters: [
    "level_name": "volcano_escape",
    "character": "knight",
    "time_seconds": 142
])

Analytics.logEvent("feature_discovered", parameters: [
    "feature_name": "dark_mode",
    "discovery_method": "settings"
])
```

```kotlin
// Android - Custom events
Firebase.analytics.logEvent("level_completed") {
    param("level_name", "volcano_escape")
    param("character", "knight")
    param("time_seconds", 142)
}
```

#### Screen Tracking

```swift
// iOS - Manual screen tracking
Analytics.logEvent(AnalyticsEventScreenView, parameters: [
    AnalyticsParameterScreenName: "ProductDetail",
    AnalyticsParameterScreenClass: "ProductDetailViewController"
])
```

```kotlin
// Android - Manual screen tracking
Firebase.analytics.logEvent(FirebaseAnalytics.Event.SCREEN_VIEW) {
    param(FirebaseAnalytics.Param.SCREEN_NAME, "ProductDetail")
    param(FirebaseAnalytics.Param.SCREEN_CLASS, "ProductDetailActivity")
}
```

#### User Properties

```swift
// iOS - Set user properties (up to 25)
Analytics.setUserProperty("premium", forName: "user_tier")
Analytics.setUserProperty("dark", forName: "theme_preference")
Analytics.setUserID("user_123")

// Clear on logout
Analytics.setUserID(nil)
```

```kotlin
// Android - Set user properties
Firebase.analytics.setUserProperty("user_tier", "premium")
Firebase.analytics.setUserProperty("theme_preference", "dark")
Firebase.analytics.setUserId("user_123")
```

### Performance Monitoring

#### Custom Traces

```swift
// iOS - Track custom operations
let trace = Performance.startTrace(name: "load_product_detail")
trace?.setValue("SKU_12345", forAttribute: "product_id")

// Perform work
let product = try await api.fetchProduct(id: productId)

trace?.incrementMetric("api_calls", by: 1)
trace?.stop()
```

```kotlin
// Android - Track custom operations
val trace = Firebase.performance.newTrace("load_product_detail")
trace.putAttribute("product_id", "SKU_12345")
trace.start()

// Perform work
val product = api.fetchProduct(productId)

trace.incrementMetric("api_calls", 1)
trace.stop()
```

#### Custom Metrics

```swift
// iOS - Add metrics to traces
trace?.setValue(product.imageUrls.count, forMetric: "image_count")
trace?.incrementMetric("cache_hits", by: 1)
```

```kotlin
// Android - Add metrics to traces
trace.putMetric("image_count", product.imageUrls.size.toLong())
trace.incrementMetric("cache_hits", 1)
```

#### Network Monitoring

```swift
// iOS - Manual network monitoring
guard let url = URL(string: "https://api.example.com/products") else { return }
let metric = HTTPMetric(url: url, httpMethod: .get)
metric?.start()

// Perform request
let (data, response) = try await URLSession.shared.data(from: url)

if let httpResponse = response as? HTTPURLResponse {
    metric?.responseCode = httpResponse.statusCode
    metric?.responseContentType = httpResponse.value(forHTTPHeaderField: "Content-Type")
}
metric?.responsePayloadSize = Int64(data.count)
metric?.stop()
```

```kotlin
// Android - Manual network monitoring
val url = "https://api.example.com/products"
val metric = Firebase.performance.newHttpMetric(url, FirebasePerformance.HttpMethod.GET)
metric.start()

// Perform request
val response = api.fetchProducts()

metric.setHttpResponseCode(response.code)
metric.setResponseContentType(response.headers["Content-Type"])
metric.setResponsePayloadSize(response.body.bytes().size.toLong())
metric.stop()
```

#### Screen Rendering

```swift
// iOS - Automatic screen rendering metrics
// Collected automatically for:
// - Screen frozen frames
// - Slow rendering
// - Frame rate

// View in console: Performance > Screen rendering
```

### Remote Config - Feature Flags

#### Setup & Fetch

```swift
// iOS - Initialize Remote Config
let remoteConfig = RemoteConfig.remoteConfig()

// Set defaults
let defaults: [String: NSObject] = [
    "feature_new_checkout": false as NSObject,
    "max_cart_items": 20 as NSObject,
    "theme_color": "#007AFF" as NSObject
]
remoteConfig.setDefaults(defaults)

// Configure fetch
let settings = RemoteConfigSettings()
settings.minimumFetchInterval = 3600  // 1 hour in production
remoteConfig.configSettings = settings

// Fetch and activate
remoteConfig.fetch { status, error in
    if status == .success {
        remoteConfig.activate { changed, error in
            // Config updated
        }
    }
}
```

```kotlin
// Android - Initialize Remote Config
val remoteConfig = Firebase.remoteConfig

// Set defaults
remoteConfig.setDefaultsAsync(mapOf(
    "feature_new_checkout" to false,
    "max_cart_items" to 20,
    "theme_color" to "#007AFF"
))

// Configure fetch
val configSettings = remoteConfigSettings {
    minimumFetchIntervalInSeconds = 3600  // 1 hour in production
}
remoteConfig.setConfigSettingsAsync(configSettings)

// Fetch and activate
remoteConfig.fetchAndActivate()
    .addOnCompleteListener { task ->
        if (task.isSuccessful) {
            // Config updated
        }
    }
```

#### Access Values

```swift
// iOS - Read config values
let showNewCheckout = remoteConfig.configValue(forKey: "feature_new_checkout").boolValue
let maxItems = remoteConfig.configValue(forKey: "max_cart_items").numberValue.intValue
let themeColor = remoteConfig.configValue(forKey: "theme_color").stringValue
```

```kotlin
// Android - Read config values
val showNewCheckout = remoteConfig.getBoolean("feature_new_checkout")
val maxItems = remoteConfig.getLong("max_cart_items").toInt()
val themeColor = remoteConfig.getString("theme_color")
```

#### Real-Time Updates

```swift
// iOS - Listen for real-time updates
remoteConfig.addOnConfigUpdateListener { configUpdate, error in
    guard let configUpdate = configUpdate, error == nil else {
        return
    }

    remoteConfig.activate { changed, error in
        // Handle updated keys
        print("Updated keys: \(configUpdate.updatedKeys)")
    }
}
```

```kotlin
// Android - Listen for real-time updates
remoteConfig.addOnConfigUpdateListener { configUpdate ->
    remoteConfig.activate().addOnCompleteListener {
        // Handle updated keys
        println("Updated keys: ${configUpdate.updatedKeys}")
    }
}
```

---

## Advanced Configuration

### Crashlytics - Debug Symbols

```bash
# iOS - Upload dSYMs (automatic via Xcode build phase)
# Add Firebase Crashlytics run script:
"${PODS_ROOT}/FirebaseCrashlytics/run"

# Android - ProGuard/R8 mapping (automatic via Gradle plugin)
# Verify mapping uploaded:
firebase crashlytics:symbols:upload --app=APP_ID build/outputs/mapping/release/mapping.txt
```

### Analytics - BigQuery Export

```bash
# Enable BigQuery export in Firebase Console:
# Project settings > Integrations > BigQuery > Link

# Query events in BigQuery:
SELECT
  event_name,
  COUNT(*) as event_count,
  user_pseudo_id
FROM `project.analytics_XXXXXXXX.events_*`
WHERE _TABLE_SUFFIX BETWEEN '20240101' AND '20240131'
  AND event_name = 'purchase'
GROUP BY event_name, user_pseudo_id
ORDER BY event_count DESC
```

### Performance - Sampling

```swift
// iOS - Configure performance sampling
// Note: Sampling controlled via Firebase Console
// Performance Monitoring > Data collection > Sampling rate

// For custom traces, use attributes to filter:
trace?.setValue("critical", forAttribute: "priority")
```

```kotlin
// Android - Configure performance sampling
// Note: Sampling controlled via Firebase Console
// Can disable for specific builds:
val firebasePerformance = Firebase.performance
firebasePerformance.isPerformanceCollectionEnabled = !BuildConfig.DEBUG
```

### Remote Config - Conditional Overrides

```swift
// iOS - Use conditions in Firebase Console:
// - App version: >= 2.0.0
// - Platform: iOS
// - User in audience: "premium_users"
// - Custom signal: user_tier == "premium"

// Access in code (same as defaults):
let featureEnabled = remoteConfig.configValue(forKey: "premium_feature").boolValue
```

---

## Integration Patterns

### Crash Context with Analytics

```swift
// iOS - Link crash context to user journey
Analytics.logEvent("checkout_initiated", parameters: [
    "cart_value": 149.99,
    "item_count": 3
])

Crashlytics.crashlytics().setCustomValue("checkout_initiated", forKey: "last_action")
Crashlytics.crashlytics().setCustomValue(149.99, forKey: "cart_value")

// Crashes will show in Analytics as 'app_exception' events
```

### Performance with Feature Flags

```swift
// iOS - Track performance by feature variant
let remoteConfig = RemoteConfig.remoteConfig()
let useNewAlgorithm = remoteConfig.configValue(forKey: "use_new_algorithm").boolValue

let trace = Performance.startTrace(name: "process_data")
trace?.setValue(useNewAlgorithm ? "v2" : "v1", forAttribute: "algorithm_version")

// Process data
processData(useNewAlgorithm: useNewAlgorithm)

trace?.stop()
```

### Analytics with Remote Config

```kotlin
// Android - Track feature flag exposure
val showNewUI = Firebase.remoteConfig.getBoolean("feature_new_ui")

Firebase.analytics.logEvent("feature_flag_exposure") {
    param("flag_name", "feature_new_ui")
    param("flag_value", showNewUI)
}
```

---

## Best Practices

| Practice | Implementation |
|----------|----------------|
| **Environment separation** | Use separate Firebase projects for dev/staging/prod |
| **User privacy** | Never log PII in events, logs, or custom keys |
| **Event naming** | Use lowercase_with_underscores (50 char max) |
| **Custom keys** | Limit to critical context (64 keys max) |
| **Remote Config defaults** | Always set in-app defaults matching console |
| **Performance traces** | Limit to critical user flows (< 300 active) |
| **Analytics properties** | Limit user properties (25 max) |
| **Debug builds** | Disable collection in debug: `setCrashlyticsCollectionEnabled(false)` |

---

## Testing & Validation

### Crashlytics Testing

```swift
// iOS - Force test crash
fatalError("Test crash")

// Or use Firebase test method
Crashlytics.crashlytics().forceCrashException()

// Verify in console within 5 minutes
```

```kotlin
// Android - Force test crash
throw RuntimeException("Test crash")

// Or use Firebase test method
Firebase.crashlytics.forceCrash()
```

### Analytics Testing

```bash
# Enable debug mode on device
# iOS - Run with argument: -FIRDebugEnabled
# Android - adb shell setprop debug.firebase.analytics.app <package_name>

# View events in Firebase Console: DebugView
# Events appear in real-time during debug sessions
```

### Performance Testing

```swift
// iOS - View traces in Xcode console
// Enable verbose logging:
// -FIRDebugEnabled in scheme arguments

// Traces appear in console within minutes
```

---

## Event Limits & Quotas

| Limit | Value |
|-------|-------|
| Custom event types | 500 |
| Event name length | 40 characters |
| Event parameter keys | 25 per event |
| Event parameter value length | 100 characters |
| User properties | 25 |
| Custom keys (Crashlytics) | 64 |
| Active traces (Performance) | 300 |
| Trace attributes | 5 per trace |

---

## Useful Console Queries

### Crashlytics

```
# Find crashes by user
user_id:user_123

# Crashes in specific feature
custom_key.feature:checkout

# Recent crashes (last 7 days)
state:open timestamp:>7d

# Crashes on specific version
version:1.2.0
```

### Analytics

```
# User retention by cohort
Retention > First Open > Cohort analysis

# Conversion funnel
Events > Create funnel > Add sequence

# User lifetime value
Audiences > Create audience > User lifetime value

# Event parameter filtering
Events > [event_name] > Filter by parameter
```

---

## Links

- [Firebase iOS SDK](https://firebase.google.com/docs/ios/setup)
- [Firebase Android SDK](https://firebase.google.com/docs/android/setup)
- [Firebase Flutter](https://firebase.google.com/docs/flutter/setup)
- [Crashlytics Documentation](https://firebase.google.com/docs/crashlytics)
- [Analytics Documentation](https://firebase.google.com/docs/analytics)
- [Performance Monitoring](https://firebase.google.com/docs/perf-mon)
- [Remote Config Documentation](https://firebase.google.com/docs/remote-config)
- [Firebase Console](https://console.firebase.google.com/)
