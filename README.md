# React Native WaaS SDK

Mikolaj Roszak www.mikolaj.com
Wykonawca: Mikołaj Roszak
Adres odbiorcy: Przedwiośnie, 79/12, 73-110, Stargard, PL
Tel. +48 500 487 977
Skype: mikolajroszak_1
Zoom: Mikołaj Roszak
email: ul.echo12@gmail.com
REGON: 383183972
NIP: 854-156-08-32
REGON: 383183972
Kapitał zakładowy 5 000 000.00 zł. Wpłacony w całości.
Dane konta Revolut
IBAN: LT41 3250 0894 7676 6825
BIC: REVOLT21
Konto (tylko przelewy krajowe): 2029 1000 0600 0000 0003 1339 92

This is the repository for the mobile React Native SDK for Wallet-as-a-Service APIs.
It exposes a subset of the WaaS APIs to the mobile developer and, in particular, is
required for the completion of MPC operations such as Seed generation and Transaction signing.

### Prerequisites:

- [node 18+](https://nodejs.org/en/download/)
- [yarn classic 1.22+](https://classic.yarnpkg.com/en/docs/install)

For iOS development:
- [Xcode 14.0+](https://developer.apple.com/xcode/)
  - iOS15.2+ simulator (iPhone 14 recommended)
- [CocoaPods](https://guides.cocoapods.org/using/getting-started.html)
- [make](https://www.gnu.org/software/make/)

For Android development:
- [Android Studio](https://developer.android.com/studio)
  - x86_64 Android emulator running Android 30+ (Pixel 5 running S recommended)
  - [Android NDK](https://developer.android.com/ndk)
- [Java 8](https://www.java.com/en/download)
  - [Java JDK 17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html) (JDK 19 will not work with React Native)

## Installation

### React Native

With `npm`:

```
npm install --save @coinbase/waas-sdk-react-native
```

With `yarn`:

```
yarn add @coinbase/waas-sdk-react-native
```

### Android

In your Android application's `settings.gradle` file, make sure to add the following:

```gradle

include ":android-native", ":android-native:go-internal-sdk", ":android-native:mpc-sdk" 

project(':android-native').projectDir = new File(rootProject.projectDir, '../node_modules/@coinbase/waas-sdk-react-native/android-native')
project(':android-native:mpc-sdk').projectDir = new File(rootProject.projectDir, '../node_modules/@coinbase/waas-sdk-react-native/android-native/mpc-sdk')
project(':android-native:go-internal-sdk').projectDir = new File(rootProject.projectDir, '../node_modules/@coinbase/waas-sdk-react-native/android-native/go-internal-sdk')
```

## Usage

See [index.tsx](./src/index.tsx) for the list of supported APIs.

## Example App

This repository provides an example app that demonstrates how the APIs should be used.

> NOTE: An example Cloud API Key json file is at `example/src/.coinbase_cloud_api_key.json`
> To run the example app, populate, or replace, this file with the Cloud API Key file provided to you
> by Coinbase.

### Running in Proxy-Mode (recommended)

To run the example app using `proxy-mode`:

1. Ensure the proxy server is correctly set up and active at the designated endpoint. The default endpoint is `localhost:8091`.
2. If your proxy server endpoint differs from the default, update the `proxyUrl` in the `config.json` file accordingly.
3. On app startup, choose `Proxy Mode` from the Mode Selection screen.

### Running in Direct-Mode (not recommended)

To run the example app using `direct-mode`:

1. Populate the `.coinbase_cloud_api_key.json` file with your personal API credentials.
2. On app startup, select `Direct Mode` from the Mode Selection screen.

### iOS
Ensure you have XCode open and run the following from the root directory of the repository:

```bash
yarn bootstrap # Install packages for the root and /example directories
yarn example start # Start the Metro server
yarn example ios --simulator "iPhone 14" # Build and start the app on iOS simulator
```

> *NOTE:* To build an app that depends on the WaaS SDK, you'll also need a compatible version of OpenSSL.
> You can build the OpenSSL framework by running the following on your Mac from the root of this repository:
> 
> `yarn ssl-ios`
> 
> You can alternatively depend on an open-compiled version of OpenSSL, like [OpenSSL-Universal](https://cocoapods.org/pods/OpenSSL-Universal), by adding the following to your app's Podfile:
> 
> `pod "OpenSSL-Universal"`

### Android
Ensure you have the following [Android environment variables](https://developer.android.com/studio/command-line/variables) set correctly:

-  `ANDROID_HOME`
-  `ANDROID_SDK_ROOT="${ANDROID_HOME}"`
-  `ANDROID_NDK_HOME="${ANDROID_HOME}/ndk/<insert ndk version>"`
-  `ANDROID_NDK_ROOT="${ANDROID_NDK_HOME}"`

And then export the following to your `PATH`:

`export PATH="${ANDROID_HOME}/emulator:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/tools:${ANDROID_HOME}/tools/bin:${ANDROID_HOME}/platform-tools:${PATH}"`


Run the following from the root directory of the repository:

```bash
yarn install # Install packages for the root directory
emulator -avd Pixel_5_API_31 # Use any x86_64 emulator with min SDK version: 30.
yarn example start # Start the Metro server
yarn example android # Build and start the app on Android emulator
```

By following the above steps, you should be able to run the example app either in proxy-mode or direct-mode based on your preference and setup.

## Recommended Architecture

Broadly speaking, there are two possible approaches to using the WaaS SDK:

1. Use the WaaS backends directly for all calls.
2. Use the WaaS backends directly only for MPC operations; proxy all other calls through an intermediate server.

Of these two approaches, we recommend approach #2, as outlined in the following diagram:

![Recommended Set-up](./assets/diagram.png)

The motivation for placing a proxy server in between your application and the WaaS backends are as
follows:

1. Your proxy server can log API calls and collect metrics.
2. Your proxy server can filter results as it sees fit (e.g. policy enforcement).
3. Your proxy server can perform end user authentication.
4. Your proxy server can store the Coinbase API Key / Secret, rather than it being exposed to the client.
5. Your proxy server can throttle traffic.

In short, having a proxy server that you control in between your application and the WaaS backends will
afford you significantly more control than using the WaaS backends directly in most cases.

The methods from the WaaS SDK which are _required_ to be used for participation in MPC are:
1. `initMPCSdk`
2. `bootstrapDevice`
3. `getRegistrationData`
4. `computeMPCOperation`

## Proxy-Mode vs. Direct-Mode

Users can switch between two distinct operating modes: `proxy-mode` and `direct-mode`.

### Proxy-Mode (recommended)

**Proxy-mode** allows the application to connect to the Coinbase WaaS API through a proxy server. The primary features of this mode include:

- Initiate the SDK without needing the Coinbase Cloud API key details in the app itself. Communication will be authenticated during Proxy-Server <> WaaS API.
- The SDK, by default, points to a proxy server endpoint at `localhost:8091`. Update this `proxyUrl` in the `config.json` file.
- Cloud credentials are stored in the proxy server, eliminating the need to have them on the application side.

### Direct-Mode (not recommended)

**Direct-mode** enables the app to connect directly to the Coinbase WaaS API, bypassing the need for a proxy server. Key aspects of this mode are:

- The app communicates directly with the Coinbase WaaS API, with no involvement of a proxy server.
- Users must provide the API Key and private key for communication, retrieved from the `.coinbase_cloud_api_key.json` file.

# Native Waas SDK

We expose a Java 8+, `java.util.concurrent.Future`-based SDK for use with Java/Kotlin. An example
app is included in `android-native-example/` for more information.

## Requirements

- Java 8+
- Gradle 7.*
  - If using central gradle repositories, you may need to update your `settings.gradle` to not fail on project repos.
    - i.e (`repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)`)

## Installation
To begin, place the `android-native` directory relative to your project.

In your `settings.gradle`, include the following:

```
include ':android-native', ':android-native:mpc-sdk', ':android-native:go-internal-sdk'
project(':android-native').projectDir = new File(rootProject.projectDir, '../android-native')
project(':android-native:mpc-sdk').projectDir = new File(rootProject.projectDir, '../android-native/mpc-sdk')
project(':android-native:go-internal-sdk').projectDir = new File(rootProject.projectDir, '../android-native/go-internal-sdk')
```

Remember to specify the correct relative-location of `android-native`.

In your `build.gradle`, you should now take dependencies on

```
implementation project(":android-native")
implementation project(':android-native:mpc-sdk')
implementation project(':android-native:go-internal-sdk')
```

## Demo App
A demo app of the native SDK is included in `android-native/`. Opening this directory with Android Studio should be 
sufficient to build and run the app.

## Considerations

- The SDK should import cleanly into Kotlin as-is -- the sample app includes a demonstration of utilizing Waas's Futures
with Kotlin task-closures. Please reach out with any questions.
