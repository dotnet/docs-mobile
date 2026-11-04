---
title: Get started with .NET for Mac Catalyst
description: Create, build, and run a native UIKit Mac Catalyst app with C# and .NET 10 on a Mac.
ms.date: 10/30/2026
ms.topic: tutorial
titleSuffix: .NET for Mac Catalyst
searchScope:
  - .NET for Mac Catalyst
---

# Get started with .NET for Mac Catalyst

.NET for Mac Catalyst lets you build Mac apps in C# using Apple's UIKit and Mac Catalyst APIs. This tutorial creates a native UIKit app and runs it directly on your Mac; it doesn't use an iOS simulator.

.NET MAUI also uses Mac Catalyst for its Mac support, but this tutorial uses the native SDK directly, without MAUI controls. For an AppKit-based Mac app instead, see [Get started with .NET for macOS](macos.md).

## Prerequisites

[!INCLUDE [Apple development prerequisites](../includes/get-started/prerequisites.md)]

You don't need an Apple Developer Program membership to run this local development app. Distribution has separate signing and provisioning requirements.

## Create the app

Create a new working directory outside any existing project:

```shell
mkdir HelloMacCatalyst
cd HelloMacCatalyst
```

[!INCLUDE [Select the .NET SDK](../includes/get-started/select-sdk.md)]

Install the native Mac Catalyst workload, then create the app:

```shell
dotnet workload install maccatalyst
dotnet new maccatalyst -n HelloMacCatalyst -o .
```

The template creates `HelloMacCatalyst.csproj`, an `Info.plist` app manifest, and C# application and view code. The app targets `net10.0-maccatalyst`.

## Build and run

On an Apple silicon Mac, build and run with:

```shell
dotnet build -f net10.0-maccatalyst -r maccatalyst-arm64
dotnet run -f net10.0-maccatalyst -r maccatalyst-arm64
```

On an Intel Mac, replace `maccatalyst-arm64` with `maccatalyst-x64` in both commands.

The app opens a native Mac window containing a UIKit view. Current .NET 10 templates display **Hello, Mac Catalyst!** from the view code in `SceneDelegate.cs`; older workload templates can have a different starter interface. No MAUI button counter is expected. Quit the app from its application menu or press **Command+Q** when finished.

If the build reports an Xcode or SDK mismatch, resolve the [Xcode requirement](../troubleshooting/xcode-requirement.md) before changing the project.

## Next steps

- Explore the [UIKit API reference](/dotnet/api/uikit?view=net-maccatalyst-26.0-10.0). Check API availability for Mac Catalyst rather than assuming that every iOS API is available on the Mac.
- Use [Xcode synchronization](/dotnet/maui/macios/xcsync) to work with Apple resources in Xcode.
- Port an existing app with [native project migration](/dotnet/maui/migration/native-projects) and [Apple project migration](/dotnet/maui/migration/apple-projects).
- Before distributing the app, review [CodesignKey](../building-apps/build-properties.md#codesignkey), [CodesignProvision](../building-apps/build-properties.md#codesignprovision), and [publishing outside the Mac App Store](/dotnet/maui/mac-catalyst/deployment/publish-outside-app-store). The publishing article uses a MAUI project; use your native project and `net10.0-maccatalyst` target framework rather than adding MAUI settings.
