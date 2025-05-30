---
title: "Xcode requirement"
description: "This article describes the Xcode requirement for building .NET for iOS, tvOS, Mac Catalyst or macOS apps. It discusses problems that can occur and a solution for these problems."
ms.date: 05/13/2025
---

# Xcode requirement

Each version of the .NET for iOS, tvOS, Mac Catalyst or macOS workloads requires a specific version of Xcode.

We state in our [release notes](https://github.com/dotnet/macios/releases) the exact Xcode version for each release.

> [!NOTE]
> In a few limited scenarios using a different version of Xcode may work, but this is not supported, even if there are no build errors or warnings.

## New workloads

A new version of a workload might require a different version of Xcode. This
typically happens whenever Apple releases a new version of Xcode: soon
afterwards we release new versions of the workloads, supporting the new
version of Xcode. Updating the workloads will thus end up requiring a new
version of Xcode.

This often manifests with a build error like this:

> This version of Microsoft.iOS requires the iOS 18.4 SDK (shipped with Xcode 16.3). The current version of Xcode is 16.2. Either install Xcode 16.3, or use a different version of Microsoft.iOS. See https://aka.ms/xcode-requirement for more information.

or:

> This version of .NET for iOS (18.4.9288) requires Xcode 16.3. The current version of Xcode is 16.2. Either install Xcode 16.3, or use a different version of .NET for iOS. See https://aka.ms/xcode-requirement for more information.

The simplest solution is typically to upgrade to the version of Xcode the
error message mentions. 

It's also possible to install an [older](#install-older-version-of-a-workload)
version of the corresponding workload to avoid having to upgrade Xcode.

In some cases the newer version of Xcode also requires updating to a newer
major version of macOS (this generally occurs around April every year). If the
new macOS version isn't supported on the developer's current hardware, the
only option is to use an [older](#install-older-version-of-a-workload) version
of the workload (or get new hardware).

## New Xcode

There is a window of time between Apple releasing a new version of Xcode and
us releasing support for this new Xcode version. Sometimes macOS will
auto-update the installed version of Xcode, which may cause problems during
this time frame.

The simplest solution is to [install multiple versions of Xcode](#installing-multiple-versions-of-xcode), and selecting the version of Xcode that corresponds with the Xcode requirement for the installed workload(s).

> [!NOTE]
> We recommend disabling automatic updates in the App Store on macOS in order to avoid this scenario.

## Installing multiple versions of Xcode

It's possible to have multiple versions of Xcode installed simultaneously.

This can be accomplished with the following steps:

* Go to the [Apple Developer Downloads](https://developer.apple.com/download/all/) site.
* Sign in with your Apple ID.
* Search for the desired versions of Xcode.
* Download the `.xip` file(s).
* Extract the file(s) by double-clicking them.
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

Here's an example for our release with support for [Xcode 16.3](https://github.com/dotnet/macios/releases/tag/dotnet-9.0.1xx-xcode16.3-9288):

```shell
$ dotnet workload install ios --version 9.0.203
```

## FAQ

### Is it safe to upgrade my Xcode?

It's possible to check if we've released support for a particular version of
Xcode by looking at our list of [releases](https://github.com/dotnet/macios/releases).

If we've released support for a particular version of Xcode, it's safe to
upgrade to that version of Xcode (this includes MAUI developers as well).
There may be other documents elsewhere stating that some older version of
Xcode is the supported version; those documents typically lag behind our
releases to some degree.

The opposite is also true: if we _haven't_ released support for a given
version of Xcode, it's likely that upgrading Xcode will cause problems. For
developers who want a newer version of Xcode, the best solution in this case
is to [install multiple versions of Xcode](#installing-multiple-versions-of-xcode).
