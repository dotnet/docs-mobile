---
title: "Xcode requirement"
description: "This article describes the Xcode requirement for building .NET for iOS, tvOS, Mac Catalyst or macOS apps. It discusses problems that can occur and a solution for these problems."
ms.date: 05/13/2025
---

# Xcode requirement

Each version of the .NET for iOS, tvOS, Mac Catalyst or macOS workloads requires a specific version of Xcode. In a few limited scenarios using a different version of Xcode may work, but this is not supported.

We state in our [release notes](https://github.com/dotnet/macios/releases) the exact Xcode version for each release.

## Workload update

A new version of a workload might require a different version of Xcode. This
typically happens whenever Apple releases a new version of Xcode: soon afterwards
we release new versions of the workloads, supporting the new version of Xcode.

There are numerous problems that can occur here:

## Ugraded the workload, but not Xcode

Upgrading the workload, but not Xcode, will lead to a build error like this:

> This version of Microsoft.iOS requires the iOS 18.4 SDK (shipped with Xcode 16.3). The current version of Xcode is 16.2. Either install Xcode 16.3, or use a different version of Microsoft.iOS. See https://aka.ms/xcode-requirement for more information.

The simplest solution is typically to upgrade to the version of Xcode the
error message mentions, while it's also possible to install an
[older](#install-older-version-of-a-workload) version of the corresponding
workload to avoid having to upgrade Xcode.

In some cases the newer version of Xcode also requires updating to a newer
major version of macOS (this generally occurs around April every year). If the
new macOS version isn't supported on the developer's current hardware, the
only option is to use an [older](#install-older-version-of-a-workload) version
of the workload (or get new hardware).

## Upgraded Xcode, but not the workload

There is a window of time between Apple releasing a new version of Xcode and
us releasing support for this new Xcode version. Sometimes macOS will
auto-update the installed version of Xcode, which may cause problems during
this time frame.

The simplest solution is to [install multiple versions of Xcode](#installing-multiple-versions-of-Xcode), and selecting the version of Xcode that corresponds with the Xcode requirement for the installed workload(s).

## Installing multiple versions of Xcode

* Go to the [Apple Developer Downloads](https://developer.apple.com/download/all/) site.
* Sign in with your Apple ID.
* Search for the desired versions of Xcode.
* Download the `.xip` files.
* Extract the files by double-clicking them.
* Rename the `Xcode.app` in the Downloads folder to something more descriptive (for instance `Xcode_15.app`).
* Move the extracted `*.app` to the `/Applications/` directory.

> [!NOTE]
> We've seen strange problems if the Xcode app is renamed after it's been opened at least once, therefore we recommend only renaming the app right after downloading and extracting it.

Once the desired versions of Xcode are installed, developers can choose between them either from Xcode (menu Xcode -> Settings -> Locations -> Command Line Tools), or by using the `xcode-select` tool from the command line:

```shell
$ sudo xcode-select --switch /Applications/Xcode_15.app
```

> [!IMPORTANT]
> The file `~/Library/Preferences/Xamarin/Settings.plist` can also be used to choose a specific version of Xcode, and this file will take precedence over the setting specified in either Xcode or on the command-line using `xcode-select`. To avoid confusion, we recommend just simply deleting this file.

## Install older version of a workload

A specific version of a workload is installed using a [workload set](/dotnet/core/tools/dotnet-workload-sets).

The exact workload set version is not predictable ahead of time, but we state the workload set version for a specific workload version with every release: https://github.com/dotnet/macios/releases.

An example for our release with support for [Xcode 16.3][https://github.com/dotnet/macios/releases/tag/dotnet-9.0.1xx-xcode16.3-9288]:

```shell
$ dotnet workload install ios --version 9.0.203
```
