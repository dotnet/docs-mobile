---
title: Get started with .NET for iOS
description: Create, build, and run a native UIKit iOS app with C#, .NET 10, and the iOS simulator on a Mac.
ms.date: 10/30/2026
ms.topic: tutorial
titleSuffix: .NET for iOS
searchScope:
  - .NET for iOS
---

# Get started with .NET for iOS

.NET for iOS lets you build native iPhone and iPad apps in C# using Apple's UIKit and other iOS frameworks. This tutorial creates a native UIKit app and runs it in the iOS simulator. You don't need an Apple Developer Program membership for this simulator tutorial.

.NET for iOS is also the platform SDK used by .NET MAUI, but this tutorial doesn't use MAUI or its cross-platform controls. For a shared UI across mobile and desktop platforms, see [What is .NET MAUI?](/dotnet/maui/what-is-maui?view=net-maui-10.0).

## Prerequisites

[!INCLUDE [Apple development prerequisites](../includes/get-started/prerequisites.md)]

## Create the app

Create a new working directory outside any existing project:

```shell
mkdir HelloIos
cd HelloIos
```

[!INCLUDE [Select the .NET SDK](../includes/get-started/select-sdk.md)]

Install the native iOS workload, then create the app:

```shell
dotnet workload install ios
dotnet new ios -n HelloIos -o .
```

The template creates `HelloIos.csproj`, an `Info.plist` app manifest, and C# application and view code. The app targets `net10.0-ios`.

## Build and run

[!INCLUDE [Choose a simulator](../includes/get-started/simulator.md)]

Choose an **iPhone or iPad simulator**, not a physical device or an Apple TV simulator. On an Apple silicon Mac, build and run with:

```shell
dotnet build -f net10.0-ios -r iossimulator-arm64
dotnet build -t:Run -f net10.0-ios -r iossimulator-arm64 '-p:_DeviceName=:v2:udid=<UDID>'
```

On an Intel Mac, use a .NET 10 workload/Xcode combination that supports Intel simulators and replace `iossimulator-arm64` with `iossimulator-x64` in both commands. Leave the quotes around the run command's property argument.

The simulator opens the app's initial view. Current .NET 10 templates display **Hello, iOS!** from the view code in `SceneDelegate.cs`; older workload templates can have a different starter interface. This is a native UIKit app, not the MAUI button-counter app. Close the app in the simulator when finished.

If the build reports an Xcode or SDK mismatch, resolve the [Xcode requirement](../troubleshooting/xcode-requirement.md) before changing the project. If the simulator isn't found, check its platform and UDID in the available-device list.

## Next steps

- Explore the [UIKit API reference](/dotnet/api/uikit?view=net-ios-26.0-10.0) and the [build properties](../building-apps/build-properties.md).
- Use [Xcode synchronization](/dotnet/maui/macios/xcsync) to work with Apple resources in Xcode.
- Port an existing app with [native project migration](/dotnet/maui/migration/native-projects) and [Apple project migration](/dotnet/maui/migration/apple-projects). For native library bindings, see [iOS binding project migration](/dotnet/maui/migration/ios-binding-projects).
- Running on a physical iPhone or iPad requires signing and provisioning. See [CodesignKey](../building-apps/build-properties.md#codesignkey), [CodesignProvision](../building-apps/build-properties.md#codesignprovision), and [iOS command-line publishing](/dotnet/maui/ios/deployment/publish-cli). The publishing article uses a MAUI project; use your native project and `net10.0-ios` target framework rather than adding MAUI settings.
