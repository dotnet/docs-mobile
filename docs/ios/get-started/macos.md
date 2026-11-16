---
title: Get started with .NET for macOS
description: Create, build, and run a native AppKit macOS app with C# and .NET 10 on a Mac.
ms.date: 10/30/2026
ms.topic: tutorial
titleSuffix: .NET for macOS
searchScope:
  - .NET for macOS
---

# Get started with .NET for macOS

.NET for macOS lets you build native Mac apps in C# using Apple's AppKit and other macOS frameworks. This tutorial creates an AppKit app and runs it directly on your Mac; it doesn't use a simulator.

This is different from [.NET for Mac Catalyst](maccatalyst.md), which uses UIKit on macOS. .NET MAUI's supported Mac target is Mac Catalyst. This tutorial uses the native AppKit SDK, not the experimental MAUI AppKit backend.

## Prerequisites

[!INCLUDE [Apple development prerequisites](../includes/get-started/prerequisites.md)]

You don't need an Apple Developer Program membership to run this local development app. Distribution has separate signing and packaging requirements.

## Create the app

Create a new working directory outside any existing project:

```shell
mkdir HelloMac
cd HelloMac
```

[!INCLUDE [Select the .NET SDK](../includes/get-started/select-sdk.md)]

Install the native macOS workload, then create the app:

```shell
dotnet workload install macos
dotnet new macos -n HelloMac -o .
```

The template creates `HelloMac.csproj`, an `Info.plist` app manifest, `Main.storyboard`, and C# application and view-controller code. The app targets `net10.0-macos`.

## Build and run

On an Apple silicon Mac, build and run with:

```shell
dotnet build -f net10.0-macos -r osx-arm64
dotnet run -f net10.0-macos -r osx-arm64
```

On an Intel Mac, replace `osx-arm64` with `osx-x64` in both commands.

The app opens a native Mac window. The AppKit template supplies an empty starter window; no MAUI controls or button counter are expected. The window layout is defined in `Main.storyboard`, with C# behavior in `ViewController.cs`. Quit the app from its application menu or press **Command+Q** when finished.

If the build reports an Xcode or SDK mismatch, resolve the [Xcode requirement](../troubleshooting/xcode-requirement.md) before changing the project.

## Next steps

- Explore the [AppKit API reference](/dotnet/api/appkit?view=net-macos-26.0-10.0) and the [build properties](../building-apps/build-properties.md).
- Use [Xcode synchronization](/dotnet/maui/macios/xcsync) to edit the storyboard in Xcode.
- Port an existing app with [native project migration](/dotnet/maui/migration/native-projects) and [Apple project migration](/dotnet/maui/migration/apple-projects).
- Before distributing the app, review signing with [CodesignKey](../building-apps/build-properties.md#codesignkey) and installer packaging with [CreatePackage](../building-apps/build-properties.md#createpackage). A locally runnable development app isn't automatically ready for the Mac App Store or distribution outside it.
