# OSLogClient
Utility class that polls OSLogStore and sends any valid logs to subscribed log drivers.

## Why? (Problem Statement)

`OSLog` is the recommended logging approach by Apple and the core Swift team. A comprehensive and reliable logging system is essential in software development. However, projects often need to send logs to third-party vendors or other services. This presents a challenge: how to use `OSLog` while also ensuring seamless and flexible integration with external logging solutions.

`OSLogClient` aims to bridge this gap by acting as an intermediary, channeling the strengths and convenience of `OSLog` into custom logging mechanisms. It does this by polling the underlying `OSLogStore`, assessing the logs, and then forwarding the post-processed log messages to registered `LogDriver` instances. As a result, a registered `LogDriver` can receive log messages with all `OSLog`-based privacy, security, and formatting intact. The driver will also receive metadata such as date-time, log level, logger subsystem, and logger category.

## Benefits of OSLog

With the increasing emphasis on user data privacy and security, Apple's `OSLog` has become an invaluable tool for developers. `OSLog` provides many advantages:

##### Privacy:
OSLog allows you to format and redact sensitive data, ensuring user data isn't unintentionally exposed.

##### Performance:
OSLog was designed for efficiency, it minimizes the performance impact on your apps.

##### Diagnostics:
OSLog Integrates seamlessly with the system's diagnostic framework, making troubleshooting easier. You can use the native Console app to filter and monitor logs far easier.

##### Recommended Approach:
Apple and the core Swift team recommend using `OSLog` over other logging mechanisms due to its in-built capabilities.

By integrating `OSLog` with our library, you are enabled to harness the strengths of OSLog while ensuring a flexible logging infrastructure that can be extended as per your project needs.

## Basic Usage

Using the `OSLogClient` is straightforward. Below is a simple guide to get you started:

```swift
// Import the library (OSLog is also included in the import)
import OSLogClient

// Initialize the OSLogClient
try OSLogClient.initialize(pollingInterval: .short)

// Register your custom log driver
let myDriver = MyLogDriver(id: "myLogDriver")
OSLogClient.registerDriver(myDriver)

// Start polling
OSLogClient.startPolling()
```

With just these three steps, `OSLogClient` begins monitoring logs from `OSLog` and forwards them to your registered log drivers, leaving you to use `OSLog.Logger` instances as normal:

```swift
let logger = Logger(subsystem: "com.company.AppName", category: "ui")

logger.info("Password '\(password, privacy: .private)' did not pass validation")
```

when your driver gets the log message, it will be the processed message that ensures any privacy and formatting has been applied. For example, when not attached to a debugger, the above would invoke with:

- `"Password '<private>' did not pass validation"`

## Subclassing LogDriver:

While the base LogDriver class provides the necessary foundation for handling OS logs, you can easily subclass it for custom processing, such as writing logs to a text file:

```swift
class FileLogDriver: LogDriver {
    let logFilePath: String
    
    init(id: String, subsystem: String, logSources: [LogSource] = []) {
        self.logFilePath = logFilePath
        super.init(id: id, logSources: logSources)
    }
    
    override func processLog(level: LogLevel, category: String, date: Date, message: String) {
        let logMessage = "[\(date)] [\(level)] [\(category)] \(message)\n"
        if let data = logMessage.data(using: .utf8) {
            try? data.append(to: fileURL)
        }
    }
}

```

## Filtering Logs with LogSource Filters

Instead of only assessing log level, date, and category in the `processLog` method, you can fine-tune which logs should be processed by a `LogDriver` instance by specifying valid `LogSource` enum cases.

If log filters are specified (i.e., the list isn't empty), they're used to evaluate incoming log entries, ensuring there's a matching filter.

Currently, two source options are supported:

- `.subsystem(String)`: Includes logs where the `subsystem` matches the provided string.
- `.subsystemAndCategories(subsystem: String, categories: [String])`: Includes logs where the `subsystem` matches the provided string **and** the log `category` is in the `categories` array.

For instance, to configure a log driver to only receive `ui` and `api` log entries:

```swift
let apiLogger = Logger(subsystem: "com.company.AppName", category: "api")
let uiLogger = Logger(subsystem: "com.company.AppName", category: "ui")
let storageLogger = Logger(subsystem: "com.vendor.AppName", category: "storage")

myLogDriver.addLogSources([
    .subsystemAndCategories(
        subsystem: "com.company.AppName",
        categories: ["ui", "api"]
    ),
])
```

With this setup, logger instances work as usual, but the driver will only capture logs validated by at least one log source:

```swift
// Driver will capture these logs:
apiLogger.info("api info message")
uiLogger.info("button was tapped")

// Driver **won't** capture this log:
storageLogger.error("database error message")
```

This approach facilitates managing loggers with varied categories across distinct driver instances as needed.

## PollingInterval:

The `PollingInterval` supports four enumerations:

- `short` - 10 second intervals
- `medium` - 30 second intervals
- `long` - 60 second intervals
- `custom(TimeInterval)` - where `TimeInterval` is the duration in seconds you want to poll at

**Note:** There is a hard-enforced minimum of 1 second for the `custom` interval option.

## Installation

Currently, OSLogClient supports Swift Package Manager (SPM).

To add OSLogClient to your project, add the following line to your dependencies in your Package.swift file:

```swift
.package(url: "https://github.com/CheekyGhost-Labs/OSLogClient", from: "0.1.0")
```

Then, add OSLogClient as a dependency for your target:

```swift
.target(
    name: "YourTarget",
    dependencies: [
        // other dependencies
        .product(name: "OSLogClient", package: "OSLogClient")
    ]
),
```

## License

OSLogClient is released under the MIT License. See the LICENSE file for more information.

## Contributing

Contributions to OSLogClient are welcomed! If you have a bug to report, feel free to help out by opening a new issue or submitting a pull request.

OSLogClient follows pretty closely to a standard git flow process. For the most part, pull requests should be made against the `develop` branch to coordinate any releases. This also provides a means to test from the `develop` branch in the wild to further test pending releases. Once a release is ready it will be merged into `main`, tagged, and have a release branch cut.

#### To get started:

1. **Fork the repository**: Start by creating a fork of the project to your own GitHub account.

2. **Clone the forked repository**: After forking, clone your forked repository to your local machine so you can make changes.

```shell
git clone https://github.com/CheekyGhost-Labs/OSLogClient.git
```

3. **Create a new branch**: Before making changes, create a new branch for your feature or bug fix. Use a descriptive name that reflects the purpose of your changes.

```shell
git checkout -b your-feature-branch
```

4. **Follow the Swift Language Guide**: Ensure that your code adheres to the [Swift Language Guide](https://swift.org/documentation/api-design-guidelines/) for styling and syntax conventions.

5. **Make your changes**: Implement your feature or bug fix, following the project's code style and best practices. Don't forget to add tests and update documentation as needed.

6. **Commit your changes**: Commit your changes with a descriptive and concise commit message. Use the imperative mood, and explain what your commit does, rather than what you did.

```shell

# Feature
git commit -m "Feature: Adding convenience method of awesomeness"


# Bug
git commit -m "Bug: Fixing issue where awesome thing was not including awesome"
```

7. **Pull the latest changes from the upstream**: Before submitting your changes, make sure to pull the latest changes from the upstream repository and merge them into your branch. This helps to avoid any potential merge conflicts.

```shell
git pull origin develop
```

8. **Push your changes**: Push your changes to your forked repository on GitHub.

```shell
git push origin your-feature-branch
```

9. **Submit a pull request**: Finally, create a pull request from your forked repository to the original repository, targeting the `develop` branch. Fill in the pull request template with the necessary details, and wait for the project maintainers to review your contribution.

### Unit Testing

Please ensure you add unit tests for any changes. The aim is not `100%` coverage, but rather meaningful test coverage that ensures your changes are behaving as expected without negatively effecting existing behavior.

Please note that the project maintainers may ask you to make changes to your contribution or provide additional information. Be open to feedback and willing to make adjustments as needed. Once your pull request is approved and merged, your changes will become part of the project!

## Additional Resources:
For a deeper dive into Apple's OSLog, please refer to the following documentation:

- [Apple's OSLog documentation](https://developer.apple.com/documentation/os/oslog)
- [Avanderlee: Unifying logging with OSLog](https://www.avanderlee.com/debugging/oslog-unified-logging/)

