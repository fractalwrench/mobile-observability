# bitdrift Capture Integration Guide

bitdrift Capture is a dynamic observability platform for mobile. Lightweight SDK with real-time control plane for selective data upload.

## Requirements

| Platform | Minimum Version |
|----------|-----------------|
| iOS | 15+ |
| Android | API 23+ |

---

## Quick Start

### iOS (Swift)

```bash
# Swift Package Manager
.package(url: "https://github.com/bitdriftlabs/capture-ios", from: "0.21.0")

# CocoaPods
pod 'BitdriftCapture', '~> 0.21.0'
```

> **Note:** Incompatible with `use_frameworks! :linkage => :static` in Podfile.

```swift
import Capture

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    Logger.start(
        withAPIKey: "YOUR_API_KEY",
        sessionStrategy: .activityBased()
    )

    return true
}
```

### Android (Kotlin)

```kotlin
// build.gradle.kts
plugins {
    id("io.bitdrift.capture-plugin") version "0.21.0"
}

dependencies {
    implementation("io.bitdrift:capture:0.21.0")
}
```

```kotlin
import io.bitdrift.capture.Capture.Logger
import io.bitdrift.capture.Capture.SessionStrategy

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        Logger.start(
            apiKey = "YOUR_API_KEY",
            sessionStrategy = SessionStrategy.ActivityBased,
            context = this
        )
    }
}
```

### React Native

```bash
npm install @bitdrift/react-native
```

```typescript
import { init, SessionStrategy } from '@bitdrift/react-native';

// Initialize as early as possible
init('YOUR_API_KEY', SessionStrategy.ActivityBased);
```

---

## Key Features

### Logging

```swift
// iOS - Level-specific logging
Logger.logInfo("User started checkout", fields: ["cart_size": "\(cart.items.count)"])
Logger.logWarning("Retry attempt", fields: ["attempt": "2"])
Logger.logError("Payment failed", fields: ["error_code": error.code], error: error)

// Trace and debug for verbose logging
Logger.logTrace("Network request started")
Logger.logDebug("Cache hit for product \(productId)")
```

```kotlin
// Android - Level-specific logging
Logger.logInfo { "User started checkout" }
Logger.logWarning(fields = mapOf("attempt" to "2")) { "Retry attempt" }
Logger.logError(throwable = error, fields = mapOf("error_code" to error.code)) { "Payment failed" }
```

### Screen Tracking

```swift
// iOS
Logger.logScreenView(screenName: "CheckoutScreen")
```

```kotlin
// Android
Logger.logScreenView("CheckoutScreen")
```

### App Launch TTI

```swift
// iOS - Call when app is truly interactive
Logger.logAppLaunchTTI(duration)
```

```kotlin
// Android
Logger.logAppLaunchTTI(durationMs)
```

### Spans

```swift
// iOS - Track operations with timing
let span = Logger.startSpan(
    name: "checkout.process_payment",
    level: .info,
    fields: ["amount": "\(order.total)"]
)

// ... payment processing

span?.end()
```

```kotlin
// Android - Inline span tracking
Logger.trackSpan("checkout.process_payment") {
    // payment processing
    processPayment()
}

// Or manual span management
val span = Logger.startSpan("checkout.process_payment")
try {
    processPayment()
} finally {
    span?.end()
}
```

### Network Logging

```swift
// iOS - Log HTTP requests/responses
Logger.log(level: .debug, HTTPRequestInfo(
    method: "POST",
    url: url,
    headers: sanitizedHeaders
))

Logger.log(level: .debug, HTTPResponseInfo(
    statusCode: response.statusCode,
    duration: requestDuration
))
```

```kotlin
// Android - Automatic OkHttp instrumentation via plugin
// In build.gradle.kts:
bitdriftCapture {
    automaticOkHttpInstrumentation.set(true)
}
```

### Session Management

```swift
// iOS
let sessionId = Logger.sessionID
let sessionUrl = Logger.sessionURL  // Link to bitdrift dashboard

// Start fresh session (e.g., on logout)
Logger.startNewSession()
```

```kotlin
// Android
val sessionId = Logger.sessionId
val sessionUrl = Logger.sessionUrl

Logger.startNewSession()
```

### Device Identification

```swift
// iOS - Persistent device ID
let deviceId = Logger.deviceID
```

```kotlin
// Android
val deviceId = Logger.deviceId
```

### Global Fields

```swift
// iOS - Attach to all subsequent logs
Logger.addField(withKey: "user_tier", value: "premium")
Logger.addField(withKey: "app_version", value: Bundle.main.appVersion)

// Remove when no longer relevant
Logger.removeField(withKey: "user_tier")
```

```kotlin
// Android
Logger.addField("user_tier", "premium")
Logger.removeField("user_tier")
```

### Feature Flags

```swift
// iOS - Track feature flag exposure
Logger.setFeatureFlagExposure(withName: "checkout_v2", variant: "enabled")
```

```kotlin
// Android
Logger.setFeatureFlagExposure("checkout_v2", "enabled")
```

### Sleep Mode

```swift
// iOS - Reduce SDK activity (e.g., when app backgrounded)
Logger.setSleepMode(.full)
Logger.setSleepMode(.normal)
```

```kotlin
// Android
Logger.setSleepMode(SleepMode.FULL)
Logger.setSleepMode(SleepMode.NORMAL)
```

### Real-Time Streaming

```swift
// iOS - Generate code for live log streaming
Logger.createTemporaryDeviceCode { result in
    switch result {
    case .success(let code):
        print("Stream code: \(code)")
    case .failure(let error):
        print("Failed: \(error)")
    }
}
```

---

## Session Strategies

| Strategy | Behavior |
|----------|----------|
| `activityBased` | New session after app backgrounded for threshold |
| `fixed` | Session lasts fixed duration regardless of activity |

---

## Dashboard Features

### Timeline / Session Replay
- Visual timeline of user sessions
- All logs, screens, network calls in context
- Jump to specific moments

### Workflows
- Deploy monitoring rules without app release
- Adjust data collection levels remotely
- Target specific users or cohorts

### Instant Insights
- Auto-generated health dashboards
- Crash rates, error trends, performance metrics

### Alerts
- Configure alerts on any workflow or metric
- Slack, PagerDuty integrations

---

## Best Practices

| Practice | Implementation |
|----------|----------------|
| **Log app launch TTI** | Call `logAppLaunchTTI` when truly interactive |
| **Track all screens** | Use `logScreenView` on every screen appear |
| **Add global context** | Set user tier, app version via `addField` |
| **Use spans for operations** | Wrap async work in spans for timing |
| **Use appropriate levels** | Trace/Debug for verbose, Info for events, Error for failures |

---

## Anti-Patterns

| Don't | Why | Do Instead |
|-------|-----|------------|
| Log PII in fields | Compliance risk | Use anonymized IDs |
| Log high-cardinality values | Storage explosion | Use bucketed values |
| Skip screen tracking | Breaks timeline visualization | Track every screen |
| Ignore session URL | Lose debugging context | Log sessionURL on errors |

---

## Symbolication

bitdrift handles symbolication via the Gradle plugin for Android:

```kotlin
// build.gradle.kts
bitdriftCapture {
    uploadProguardMappings.set(true)
}
```

For iOS, dSYM upload is configured via the bitdrift dashboard or CLI.

---

## Links

- [bitdrift Docs](https://docs.bitdrift.io)
- [SDK Quickstart](https://docs.bitdrift.io/sdk/quickstart.html)
- [GitHub - capture-sdk](https://github.com/bitdriftlabs/capture-sdk)
- [GitHub - capture-ios](https://github.com/bitdriftlabs/capture-ios)
- [iOS Releases](https://docs.bitdrift.io/sdk/releases-ios.html)
- [Android Releases](https://docs.bitdrift.io/sdk/releases-android.html)
