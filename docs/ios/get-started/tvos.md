---
title: Get started with .NET for tvOS
description: Create, build, and run a native UIKit tvOS app with C#, .NET 10, and the Apple TV simulator on a Mac.
ms.date: 10/30/2026
ms.topic: tutorial
titleSuffix: .NET for tvOS
searchScope:
  - .NET for tvOS
---

# Get started with .NET for tvOS

.NET for tvOS lets you build native Apple TV apps in C# using Apple's UIKit and other tvOS frameworks. This tutorial creates a native tvOS app and runs it in the Apple TV simulator. You don't need an Apple Developer Program membership for this simulator tutorial.

The tvOS workload is a native platform SDK. tvOS isn't a supported .NET MAUI target, and this tutorial doesn't use MAUI.

## Prerequisites

[!INCLUDE [Apple development prerequisites](../includes/get-started/prerequisites.md)]

## Create the app

Create a new working directory outside any existing project:

```shell
mkdir HelloTv
cd HelloTv
```

[!INCLUDE [Select the .NET SDK](../includes/get-started/select-sdk.md)]

Install the native tvOS workload, then create the app:

```shell
dotnet workload install tvos
dotnet new tvos -n HelloTv -o .
```

The template creates `HelloTv.csproj`, an `Info.plist` app manifest, `Main.storyboard`, and C# application and view-controller code. The app targets `net10.0-tvos`.

## Build and run

[!INCLUDE [Choose a simulator](../includes/get-started/simulator.md)]

Choose an **Apple TV simulator**, not an iPhone or iPad simulator. On an Apple silicon Mac, build and run with:

```shell
dotnet build -f net10.0-tvos -r tvossimulator-arm64
dotnet build -t:Run -f net10.0-tvos -r tvossimulator-arm64 '-p:_DeviceName=:v2:udid=<UDID>'
```

On an Intel Mac, use a .NET 10 workload/Xcode combination that supports Intel simulators and replace `tvossimulator-arm64` with `tvossimulator-x64` in both commands. Leave the quotes around the run command's property argument.

The Apple TV simulator launches the app's initial view. The template supplies an empty view defined in `Main.storyboard`, with C# behavior in `ViewController.cs`. A blank view is expected: this is a native starter app, not a media player or a MAUI button-counter app. Return to the simulator's Home screen when finished.

If the build reports an Xcode or SDK mismatch, resolve the [Xcode requirement](../troubleshooting/xcode-requirement.md) before changing the project. If the simulator isn't found, check its platform and UDID in the available-device list.

## Next steps

- Explore the [UIKit API reference](/dotnet/api/uikit?view=net-tvos-26.0-10.0). Check each API's platform availability before using it on tvOS.
- Use [Xcode synchronization](/dotnet/maui/macios/xcsync) to work with Apple resources in Xcode.
- Port an existing app with [native project migration](/dotnet/maui/migration/native-projects) and [Apple project migration](/dotnet/maui/migration/apple-projects).
- Running on a physical Apple TV requires signing and provisioning. See [CodesignKey](../building-apps/build-properties.md#codesignkey), [CodesignProvision](../building-apps/build-properties.md#codesignprovision), and [BuildIpa](../building-apps/build-properties.md#buildipa) for the corresponding native build settings.
