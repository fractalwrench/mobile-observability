# BugSnag Integration Guide

BugSnag error monitoring and stability tracking for iOS, Android, and React Native.

## Quick Start

### iOS (Swift)

```bash
# SPM
.package(url: "https://github.com/bugsnag/bugsnag-cocoa", from: "6.0.0")
```

```swift
import Bugsnag

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    let config = BugsnagConfiguration("YOUR_API_KEY")
    config.releaseStage = AppConfig.environment
    config.appVersion = Bundle.main.appVersion
    config.enabledErrorTypes.ooms = true
    config.enabledErrorTypes.thermalKills = true
    config.addOnSend { event in
        // Sanitize PII
        return true
    }
    Bugsnag.start(with: config)
    return true
}
```

### Android (Kotlin)

```kotlin
// build.gradle.kts
plugins { id("com.bugsnag.android.gradle") version "8.0.0" }
dependencies { implementation("com.bugsnag:bugsnag-android:6.0.0") }
```

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        val config = Configuration.load(this).apply {
            apiKey = "YOUR_API_KEY"
            releaseStage = BuildConfig.BUILD_TYPE
            enabledErrorTypes.anrs = true
            enabledErrorTypes.ndkCrashes = true
        }
        Bugsnag.start(this, config)
    }
}
```

### React Native

```bash
npm install @bugsnag/react-native
```

```typescript
import Bugsnag from '@bugsnag/react-native';

Bugsnag.start({
  apiKey: 'YOUR_API_KEY',
  releaseStage: __DEV__ ? 'development' : 'production',
});

export default Bugsnag.getPlugin('react')?.createErrorBoundary(React)(App);
```

---

## Key Features

### Error Reporting

```swift
// iOS
do {
    try riskyOperation()
} catch {
    Bugsnag.notifyError(error) { event in
        event.addMetadata(["product_id": productId], section: "checkout")
        event.severity = .warning
        return true
    }
}
```

```kotlin
// Android
try {
    riskyOperation()
} catch (e: Exception) {
    Bugsnag.notify(e) { event ->
        event.addMetadata("checkout", mapOf("product_id" to productId))
        event.severity = Severity.WARNING
        true
    }
}
```

### Breadcrumbs

```swift
// iOS
Bugsnag.leaveBreadcrumb("Added to cart", metadata: ["product_id": productId], type: .user)
```

```kotlin
// Android
Bugsnag.leaveBreadcrumb("Added to cart", mapOf("product_id" to productId), BreadcrumbType.USER)
```

### User Tracking

```swift
// iOS
Bugsnag.setUser("user_123", withEmail: "user@example.com", andName: "John")
```

```kotlin
// Android
Bugsnag.setUser("user_123", "user@example.com", "John")
```

### Stability Score

BugSnag calculates stability score automatically:
- **Crash-free sessions %**
- **Crash-free users %**
- Tracked per release for regression detection

### Feature Flags

```swift
// iOS
Bugsnag.addFeatureFlag(name: "checkout_v2", variant: "enabled")
```

```kotlin
// Android
Bugsnag.addFeatureFlag("checkout_v2", "enabled")
```

---

## Best Practices

| Practice | Implementation |
|----------|----------------|
| **Set release stage** | `config.releaseStage = "production"` |
| **Track user** | Enable stability-per-user metrics |
| **Add metadata** | Context for faster debugging |
| **Use feature flags** | Correlate errors with experiments |
| **Upload source maps** | `bugsnag-cli upload` in CI |

---

## Performance Monitoring

BugSnag captures performance via custom spans:

```swift
// iOS - Custom spans
let span = Bugsnag.startSpan(name: "checkout.process_payment")
defer { span?.end() }
// ... payment logic
```

```kotlin
// Android - Custom spans
val span = Bugsnag.startSpan("checkout.process_payment")
try {
    // payment logic
} finally {
    span?.end()
}
```

### App Launch

```swift
// iOS - Measured automatically with enabledBehaviors
config.enabledBehaviors.appHangs = true
```

---

## Session Replay

BugSnag does not include session replay. Consider pairing with:
- FullStory
- LogRocket
- Smartlook

---

## Alert Configuration

Configure via BugSnag Dashboard:

1. **Project Settings → Alerts**
2. Set thresholds:
   - New error: Immediate
   - Error spike: >10 occurrences in 5 min
   - Stability degradation: <99% crash-free

### Slack Integration

```
Project Settings → Integrations → Slack
- #mobile-alerts for P0
- #mobile-monitoring for daily digest
```

---

## Dashboard Queries

### Find High-Impact Errors

```
Error Status: Open
Sort By: User Count (Descending)
Release: [current]
```

### Release Comparison

```
Compare → Select two releases → View stability delta
```

### Error Grouping

BugSnag groups automatically by:
1. Exception class + message
2. Stack trace fingerprint
3. Custom grouping hash (if configured)

---

## Symbolication

### iOS

```bash
# Build phase script
bugsnag-cli upload dsym \
  --api-key $BUGSNAG_API_KEY \
  --project-root "$PROJECT_DIR" \
  "${DWARF_DSYM_FOLDER_PATH}"
```

### Android

```kotlin
// build.gradle.kts
bugsnag {
    apiKey.set("YOUR_API_KEY")
    uploadNdkMappings.set(true)
    uploadProguardMapping.set(true)
}
```

### React Native

```bash
# In CI after build
bugsnag-cli upload react-native-sourcemaps \
  --api-key $BUGSNAG_API_KEY \
  --bundle ./build/main.jsbundle \
  --source-map ./build/main.jsbundle.map
```

---

## Anti-Patterns

| Don't | Why | Do Instead |
|-------|-----|------------|
| Log full user objects | PII risk | Log user ID only |
| Catch and ignore | Misses errors | Catch and notify |
| Skip feature flags | Can't correlate | Always add flags |

---

## Links

- [BugSnag iOS Docs](https://docs.bugsnag.com/platforms/ios/)
- [BugSnag Android Docs](https://docs.bugsnag.com/platforms/android/)
- [BugSnag React Native Docs](https://docs.bugsnag.com/platforms/react-native/)
