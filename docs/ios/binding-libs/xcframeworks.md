---
title: "Bind and distribute XCFrameworks in .NET for iOS, Mac Catalyst, macOS, and tvOS"
description: "Learn how to bind single or multiple Apple XCFrameworks in an SDK-style binding project, understand binding resource packages and NoBindingEmbedding, and distribute bindings via NuGet."
ms.date: 10/29/2026
no-loc: [ "xcframework" ]
---

# Bind and distribute XCFrameworks

An `.xcframework` bundle is Apple's standard format for packaging a native framework or static library across multiple platforms (iOS, Mac Catalyst, macOS, tvOS) and environments (device and simulator) in a single directory structure.

This article covers how to:

* Create an SDK-style binding project for an `.xcframework`.
* Generate C# API definitions with Objective Sharpie.
* Integrate multiple `.xcframework` bundles (including transitive native dependencies) in a binding project.
* Understand binding resource packages (`.resources` / `.resources.zip`) and `NoBindingEmbedding`.
* Distribute a binding library as a NuGet package or assembly reference, and troubleshoot common linker and packaging errors.

## Create a binding project for an XCFramework

### 1. Create an SDK-style binding project

Create a new binding project using the .NET CLI (`dotnet new iosbinding`, `dotnet new maccatalystbinding`, `dotnet new macosbinding`, or `dotnet new tvosbinding`), or configure an SDK-style `.csproj` with `<IsBindingProject>true</IsBindingProject>`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net10.0-ios;net10.0-maccatalyst</TargetFrameworks>
    <Nullable>enable</Nullable>
    <ImplicitUsings>true</ImplicitUsings>
    <IsBindingProject>true</IsBindingProject>
  </PropertyGroup>

  <ItemGroup>
    <ObjcBindingApiDefinition Include="ApiDefinition.cs" />
    <ObjcBindingCoreSource Include="StructsAndEnums.cs" />
  </ItemGroup>

  <ItemGroup>
    <NativeReference Include="NativeFrameworks/MySdk.xcframework">
      <Kind>Framework</Kind>
      <SmartLink>true</SmartLink>
      <ForceLoad>true</ForceLoad>
      <Frameworks>Foundation UIKit</Frameworks>
    </NativeReference>
  </ItemGroup>
</Project>
```

### 2. Point `<NativeReference>` to the root `.xcframework`

Always set `<NativeReference Include="..." />` to the top-level `.xcframework` directory (or a compressed `.xcframework.zip` archive)—**never** to a platform-specific slice subdirectory such as `MySdk.xcframework/ios-arm64/MySdk.framework`.

During the build, the .NET SDK inspects the XCFramework's `Info.plist` and automatically selects the slice that matches the current target platform (`ios`, `maccatalyst`, `macos`, `tvos`), target environment (device vs. simulator), and architecture (`arm64`, `x64`).

### 3. Configure `Kind` and linker metadata

Set the `<NativeReference>` metadata to match the contents of the `.xcframework` slices:

| Metadata         | When to use                                                                                                                                                              |
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Kind`           | Set to `Framework` if each slice inside the `.xcframework` contains a `.framework` bundle (dynamic or static framework), `Static` if each slice contains a bare `.a` static archive, or `Dynamic` for a bare `.dylib` (or omit `Kind` to let the SDK infer it from the resolved slice). |
| `SmartLink`      | Defaults to `true`. Lets the SDK determine whether `-force_load` is needed.                                                                                              |
| `ForceLoad`      | Set to `true` when static libraries or static frameworks define Objective-C categories or classes that the native linker might otherwise strip as unused.                |
| `Frameworks`     | Space-separated list of Apple system frameworks that the XCFramework links against (for example, `Foundation UIKit CoreGraphics Security`).                              |
| `WeakFrameworks` | Space-separated list of optional Apple system frameworks linked with `-weak_framework`.                                                                                  |
| `LinkerFlags`    | Additional flags passed to the native linker, such as `-ObjC`, `-lsqlite3`, or `-lz`.                                                                                    |
| `IsCxx`          | Set to `true` if the native library is implemented in C++ and requires `libc++`.                                                                                         |

### 4. Generate `ApiDefinition.cs` with Objective Sharpie

While `<NativeReference>` in the `.csproj` points to the root `.xcframework`, [Objective Sharpie](/dotnet/maui/ios/objective-sharpie/platform/native-frameworks) inspects headers from a single platform `.framework` slice inside the `.xcframework` to scaffold `ApiDefinition.cs` and `StructsAndEnums.cs`:

```bash
sharpie bind \
  -f ./NativeFrameworks/MySdk.xcframework/ios-arm64/MySdk.framework \
  -sdk iphoneos \
  -namespace MySdkBinding \
  -o Binding
```

After running Objective Sharpie, review the generated `ApiDefinition.cs` and `StructsAndEnums.cs`, remove any `[Verify]` attributes after confirming the signatures, and include the files in `ObjcBindingApiDefinition` and `ObjcBindingCoreSource`.

## Integrate multiple XCFrameworks in a binding project

Many third-party Apple SDKs ship as a primary `.xcframework` accompanied by one or more supporting `.xcframework` bundles (for example, core utilities, networking, or analytics modules).

### Add multiple `<NativeReference>` items

You can include multiple `<NativeReference>` items in the same binding project:

```xml
<ItemGroup>
  <!-- Primary SDK XCFramework -->
  <NativeReference Include="NativeFrameworks/PrimarySdk.xcframework">
    <Kind>Framework</Kind>
    <SmartLink>true</SmartLink>
    <ForceLoad>true</ForceLoad>
    <Frameworks>Foundation UIKit Security</Frameworks>
  </NativeReference>

  <!-- Supporting dependency XCFrameworks required by PrimarySdk -->
  <NativeReference Include="NativeFrameworks/PrimarySdkCore.xcframework">
    <Kind>Framework</Kind>
    <SmartLink>true</SmartLink>
    <ForceLoad>true</ForceLoad>
  </NativeReference>

  <NativeReference Include="NativeFrameworks/PrimarySdkNetworking.xcframework">
    <Kind>Framework</Kind>
    <SmartLink>true</SmartLink>
  </NativeReference>
</ItemGroup>
```

### Native linking (`<NativeReference>`) vs. C# API binding (`ApiDefinition.cs`)

In .NET for iOS, Mac Catalyst, macOS, and tvOS, **packaging/linking a native binary** and **exposing a C# API for it** are independent steps:

* **Internal / transitive dependency XCFrameworks**: If `PrimarySdk.xcframework` depends on `PrimarySdkCore.xcframework` internally, and your C# code does not directly call types from `PrimarySdkCore` (and its types do not appear in the public signatures of `PrimarySdk` APIs that you bind), you only need to add `<NativeReference Include="NativeFrameworks/PrimarySdkCore.xcframework" ... />`. You do **not** need to generate or write `ObjcBindingApiDefinition` entries for `PrimarySdkCore`.
* **Publicly exposed dependency types**: If a bound method or property in `PrimarySdk` accepts or returns a type defined in `PrimarySdkCore`, either bind that type in `ApiDefinition.cs` (in the same binding project or a referenced binding project) or replace the parameter/return type with `NSObject` / `IntPtr` if callers don't need strongly typed access to it.

### Single binding project vs. separate binding projects

| Scenario                                                                                                 | Recommended structure                                                                                                                                          |
|----------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Supporting XCFrameworks are exclusively used by one SDK and always ship together                         | Keep all `<NativeReference>` items in a **single binding project**. All XCFrameworks are packed together into the same binding resource package and NuGet.     |
| A shared dependency XCFramework is used by multiple independent binding libraries in the same app        | Place the shared XCFramework in its **own binding project / NuGet package** and reference that package from the dependent binding projects to avoid duplicate native symbols or conflicting framework copies. |

## Binding resource packages (`.resources`) and `NoBindingEmbedding`

When you build a binding project, the build produces two outputs in `bin/<Configuration>/<TargetFramework>/`:

1. **`<AssemblyName>.dll`**: the managed binding assembly containing the C# types generated from `ApiDefinition.cs` and `StructsAndEnums.cs`.
2. **`<AssemblyName>.resources.zip` (or `<AssemblyName>.resources/` directory)**: a sidecar *binding resource package* containing every `<NativeReference>` (such as `.xcframework` bundles) along with a `manifest` file that records the `<NativeReference>` linker metadata (`Kind`, `Frameworks`, `ForceLoad`, `LinkerFlags`, and so on).

```text
bin/Release/net10.0-ios/
├── MySdkBinding.dll
└── MySdkBinding.resources.zip
    ├── manifest
    ├── MySdk.xcframework/
    └── MySdkCore.xcframework/
```

### `NoBindingEmbedding` behavior and .NET 12 removal of `false`

The [`NoBindingEmbedding`](../building-apps/build-properties.md#nobindingembedding) MSBuild property controls whether native references are placed in the sidecar binding resource package (`true`) or embedded inside `<AssemblyName>.dll` as managed resources (`false`):

* **`NoBindingEmbedding` defaults to `true`.** You do not need to set `<NoBindingEmbedding>true</NoBindingEmbedding>` in your project file.
* **XCFrameworks require `NoBindingEmbedding=true`.** XCFrameworks cannot be embedded inside the managed assembly on any .NET version.

> [!WARNING]
> Setting `<NoBindingEmbedding>false</NoBindingEmbedding>` is deprecated and is **no longer supported starting with .NET 12**, where setting it to `false` produces a build error. If your project file sets `<NoBindingEmbedding>false</NoBindingEmbedding>`, remove that property so the default (`true`) is used.

## Distribute a binding library

### Why copying only `<AssemblyName>.dll` fails

Because native XCFrameworks and their linker manifest live in the `<AssemblyName>.resources.zip` (or `<AssemblyName>.resources`) sidecar rather than inside `<AssemblyName>.dll`, **copying only `<AssemblyName>.dll` to another folder or repository is incomplete**. If a consuming application references a standalone `.dll` without its `.resources.zip` or `.resources` sidecar alongside it, the native XCFramework is not linked or bundled into the app, resulting in build-time native linker errors (`Undefined symbols for architecture ...`) or runtime `ObjCRuntime.Class` lookup failures.

### Recommended: Create a NuGet package with `dotnet pack`

The recommended way to distribute a binding project is to pack it into a NuGet package using `dotnet pack`:

```dotnetcli
dotnet pack -c Release
```

In SDK-style binding projects, `dotnet pack` **automatically** packages both `<AssemblyName>.dll` and `<AssemblyName>.resources.zip` (or the `<AssemblyName>.resources` directory) under `lib/<TargetFramework>/` inside the `.nupkg`:

```text
MySdkBinding.1.0.0.nupkg
└── lib/
    ├── net10.0-ios26.0/
    │   ├── MySdkBinding.dll
    │   └── MySdkBinding.resources.zip
    └── net10.0-maccatalyst26.0/
        ├── MySdkBinding.dll
        └── MySdkBinding.resources.zip
```

No custom `.nuspec` file or manual MSBuild copy targets from legacy Xamarin projects are needed. You can configure standard NuGet metadata properties (`PackageId`, `Version`, `Authors`, `Description`) directly in the binding `.csproj`.

### Configure `CompressBindingResourcePackage`

The [`CompressBindingResourcePackage`](../building-apps/build-properties.md#compressbindingresourcepackage) property controls whether the sidecar binding resource package is emitted as a `.resources.zip` archive or an uncompressed `.resources` directory:

* `auto` (default): automatically chooses whether to zip the resource package (for example, when symlinks are present in frameworks).
* `true`: always creates `<AssemblyName>.resources.zip`. Recommended when distributing XCFrameworks that contain symbolic links or deep paths that could exceed Windows `MAX_PATH` limits.
* `false`: creates an uncompressed `<AssemblyName>.resources` directory.

### Direct assembly reference (`<Reference>`)

If you must reference a prebuilt binding `.dll` directly with `<Reference Include="path/to/MySdkBinding.dll" />` instead of `<ProjectReference>` or `<PackageReference>`, always copy `MySdkBinding.resources.zip` (or the `MySdkBinding.resources` directory) into the **exact same directory** as `MySdkBinding.dll`. The Apple SDK build automatically discovers `<AssemblyName>.resources.zip` or `<AssemblyName>.resources` next to any referenced assembly.

## Consume the binding in an application

Reference the binding library from your .NET for iOS, Mac Catalyst, macOS, or tvOS application (or .NET MAUI app) using either `<ProjectReference>` or `<PackageReference>`:

```xml
<!-- Consuming via ProjectReference -->
<ItemGroup Condition="'$(TargetPlatformIdentifier)' == 'ios'">
  <ProjectReference Include="..\MySdkBinding\MySdkBinding.csproj" />
</ItemGroup>

<!-- Or consuming via PackageReference -->
<ItemGroup Condition="'$(TargetPlatformIdentifier)' == 'ios'">
  <PackageReference Include="MySdkBinding" Version="1.0.0" />
</ItemGroup>
```

> [!IMPORTANT]
> Do **not** add `<NativeReference>` for the same `.xcframework` in the consuming application project when referencing the binding project or NuGet package. The binding's `.resources` / `.resources.zip` sidecar already supplies the native XCFramework and its linker metadata to the application build. Duplicating `<NativeReference>` in the app project can cause duplicate symbol or bundle copy conflicts.

## Troubleshoot XCFramework bindings

| Symptom or error message                                                                                          | Likely cause                                                                                                                                     | Resolution                                                                                                                                                                                                                     |
|-------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Undefined symbols for architecture arm64: "_OBJC_CLASS_$_SomeClass", referenced from: registrar.o`               | A bound Objective-C class is missing at native link time because its XCFramework wasn't linked, `Kind` is wrong, or a dependency XCFramework is missing. | Verify `<NativeReference>` points to the `.xcframework` containing `SomeClass`, confirm `Kind` (`Framework` vs. `Static`) matches the slice payload, add `ForceLoad="true"` for static libraries, and add any required dependency XCFrameworks as `<NativeReference>` items. |
| `Undefined symbols for architecture ...` referencing Apple system symbols (for example `_SecItemCopyMatching` or `_sqlite3_*`) | The XCFramework depends on Apple system frameworks or system libraries that aren't declared in metadata.                                          | Add the required system frameworks to `<Frameworks>Security ...</Frameworks>` or libraries to `<LinkerFlags>-lsqlite3</LinkerFlags>` on the `<NativeReference>`.                                                               |
| Build error stating no matching slice was found in `MySdk.xcframework` for `iossimulator-arm64` or `ios-arm64`     | The `.xcframework` was built without a slice for the target platform, environment (simulator vs. device), or CPU architecture (`arm64`).         | Inspect `MySdk.xcframework/Info.plist` (`AvailableLibraries`) to see which slices exist. Rebuild the `.xcframework` in Xcode with `xcodebuild -create-xcframework` including both device (`iphoneos`) and simulator (`iphonesimulator` with `arm64`) archives. |
| Binding compiles, but referencing `MySdkBinding.dll` in an app fails to link or crashes with `Class.GetHandle` / missing framework at launch | `MySdkBinding.dll` was distributed or copied without its `MySdkBinding.resources.zip` / `MySdkBinding.resources` sidecar, or `<NoBindingEmbedding>false</NoBindingEmbedding>` was set. | Distribute the binding via `dotnet pack` (NuGet `<PackageReference>`), use `<ProjectReference>`, or keep `MySdkBinding.resources.zip` in the same folder as `MySdkBinding.dll`. Ensure `NoBindingEmbedding` is not set to `false`. |
| Corrupted frameworks or missing headers/binaries when unpacking on Windows                                        | Framework bundles inside the `.xcframework` use symbolic links that were lost when committed to Git or copied uncompressed on Windows.           | Supply the XCFramework as a `.xcframework.zip` archive in `<NativeReference Include="MySdk.xcframework.zip" />` and keep `<CompressBindingResourcePackage>true</CompressBindingResourcePackage>` (or `auto`).                  |
| Warning `MT7200` about framework extension capitalization                                                         | A `.framework` directory uses uppercase characters such as `.Framework` or `.FRAMEWORK`.                                                         | Rename the framework directory and update references to use the lowercase `.framework` extension.                                                                                                                              |

## See also

* [Use and distribute native libraries in .NET for iOS, Mac Catalyst, macOS, and tvOS](index.md)
* [Build items for iOS, Mac Catalyst, macOS, and tvOS (`NativeReference`, `ObjcBindingApiDefinition`, `ObjcBindingCoreSource`)](../building-apps/build-items.md#nativereference)
* [Build properties for iOS, Mac Catalyst, macOS, and tvOS (`NoBindingEmbedding`, `CompressBindingResourcePackage`)](../building-apps/build-properties.md#nobindingembedding)
