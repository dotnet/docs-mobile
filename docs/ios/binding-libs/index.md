---
title: "Use and distribute native libraries in .NET for iOS, Mac Catalyst, macOS, and tvOS"
description: "Learn how to link C, C++, Rust, Objective-C, and Xcode native libraries and XCFrameworks into .NET for iOS, Mac Catalyst, macOS, and tvOS projects and NuGet packages."
ms.date: 10/29/2026
no-loc: [ "xcframework" ]
---

# Use and distribute native libraries in .NET for iOS, Mac Catalyst, macOS, and tvOS

.NET for iOS, Mac Catalyst, macOS, and tvOS applications and libraries can consume native code packaged as static libraries (`*.a`), dynamic libraries (`*.dylib`), frameworks (`*.framework`), multi-platform XCFrameworks (`*.xcframework`), or Xcode source projects (`*.xcodeproj`).

This article explains how to choose the right integration approach, link native binaries directly in an application or class library project, and package native binaries into NuGet packages for reuse across projects.

## Choose an integration approach

The right way to reference a native library depends on its language ABI and how you plan to distribute it:

| Scenario                                                                                                           | Recommended approach                                                                                                                                                                   | Binding project required? |
|--------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------|
| Prebuilt C, C++, or Rust binary (`*.a`, `*.dylib`, `*.framework`, `*.xcframework`) called with P/Invoke (`[LibraryImport]` / `[DllImport]`) | Add a `<NativeReference>` item in the app or library project, or ship in a NuGet package under `runtimes/<rid>/native/`.                                                               | No                        |
| Objective-C library or framework (`*.framework`, `*.xcframework`, `*.a`)                                           | Create a binding project (`<IsBindingProject>true</IsBindingProject>`) with `ObjcBindingApiDefinition`, `ObjcBindingCoreSource`, and `<NativeReference>`. See [Bind and distribute XCFrameworks](xcframeworks.md). | Yes                       |
| Swift framework or XCFramework with `@objc` APIs                                                                   | Expose an `@objc`-annotated wrapper surface and bind it with a binding project and `<NativeReference>`, or use [Native Library Interop](/dotnet/communitytoolkit/maui/native-library-interop). | Yes                       |
| Xcode source project (`*.xcodeproj`)                                                                               | Reference the project with `<XcodeProject>` (.NET 9+), which builds an XCFramework and registers it as a `<NativeReference>` automatically.                                            | Only for Objective-C APIs |

### C, C++, and Rust libraries (P/Invoke)

If a native library exposes a C ABI and you call it using `[LibraryImport]` or `[DllImport]`, you do **not** need an Objective-C binding project unless you specifically want to package the binary as a binding resource package. Reference the native binary directly with `<NativeReference>` in your application project, or distribute it in a NuGet package under `runtimes/<rid>/native/`.

### Objective-C libraries and XCFrameworks

If a native library exposes Objective-C classes, protocols, or categories, use a binding project (`<IsBindingProject>true</IsBindingProject>`). The binding generator (`bgen`) compiles your `ObjcBindingApiDefinition` and `ObjcBindingCoreSource` items into strongly typed C# wrappers and packages the native binaries into a sidecar binding resource package (`.resources` or `.resources.zip`). For a complete walkthrough, see [Bind and distribute XCFrameworks](xcframeworks.md).

### Xcode source projects (`*.xcodeproj`)

Starting in .NET 9, you can build an Xcode framework project directly during the .NET build using the [`<XcodeProject>`](../building-apps/build-items.md#xcodeproj) item group:

```xml
<ItemGroup>
  <XcodeProject Include="../NativeSource/MyLibrary.xcodeproj"
                SchemeName="MyLibrary"
                Configuration="Release"
                Kind="Framework"
                SmartLink="true" />
</ItemGroup>
```

The build compiles the Xcode scheme into an `.xcframework` and automatically adds it to `@(NativeReference)` unless `CreateNativeReference="false"` is specified. Any `<NativeReference>` metadata set on `<XcodeProject>` (such as `Kind`, `Frameworks`, or `SmartLink`) is forwarded to the generated `<NativeReference>`.

## Consume native libraries directly in a project

To link a native binary directly into a .NET for iOS, Mac Catalyst, macOS, or tvOS project, add a [`<NativeReference>`](../building-apps/build-items.md#nativereference) item to your `.csproj` file.

### Static library (`*.a`) or static XCFramework

```xml
<ItemGroup>
  <!-- Single-architecture or universal static library -->
  <NativeReference Include="native/libfoo.a"
                   Kind="Static"
                   ForceLoad="true" />

  <!-- Multi-platform XCFramework containing static libraries -->
  <NativeReference Include="native/libbar.xcframework"
                   Kind="Static"
                   SmartLink="true"
                   ForceLoad="true"
                   IsCxx="true" />
</ItemGroup>
```

Static libraries are linked directly into the application's main executable. When calling C functions from a static library with `[LibraryImport]` or `[DllImport]`, use `"__Internal"` as the library name:

```csharp
using System.Runtime.InteropServices;

internal static partial class NativeMethods
{
    [LibraryImport("__Internal", EntryPoint = "foo_compute")]
    internal static partial int FooCompute(int value);
}
```

### Dynamic library (`*.dylib`) or framework (`*.framework` / `*.xcframework`)

```xml
<ItemGroup>
  <!-- Dynamic framework inside an XCFramework -->
  <NativeReference Include="native/MyCalc.xcframework"
                   Kind="Framework"
                   SmartLink="true"
                   Frameworks="Foundation CoreGraphics" />
</ItemGroup>
```

Dynamic frameworks are copied into the app bundle's `Frameworks` directory and code-signed with the app. When calling exported C functions from a dynamic framework with P/Invoke, pass `@rpath/<FrameworkName>.framework/<FrameworkName>`:

```csharp
using System.Runtime.InteropServices;

internal static partial class NativeMethods
{
    [LibraryImport("@rpath/MyCalc.framework/MyCalc", EntryPoint = "mycalc_add")]
    internal static partial int Add(int a, int b);
}
```

> [!IMPORTANT]
> Always point `<NativeReference>` to the root `.xcframework` directory (or `.xcframework.zip` archive), never to a platform-specific slice subfolder such as `ios-arm64/MyCalc.framework`. The .NET SDK automatically selects the slice matching the active target platform, simulator or device environment, and CPU architecture.

### `<NativeReference>` metadata reference

The following metadata can be configured on `<NativeReference>` items:

| Metadata                       | Description                                                                                                                                                                                    |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Kind`                         | `Static` for static libraries (`*.a`), `Dynamic` for dynamic libraries (`*.dylib`), or `Framework` for `.framework` bundles. For an `.xcframework`, set `Kind` to match the payload inside its slices (`Framework`, `Static`, or `Dynamic`), or omit `Kind` to let the SDK infer it from the resolved slice. |
| `SmartLink`                    | Boolean. Controls whether the .NET SDK determines whether `ForceLoad` is required. Defaults to `true`.                                                                                         |
| `ForceLoad`                    | Boolean. Passes `-force_load` to the native linker so unused archive members (such as Objective-C categories) aren't stripped.                                                                 |
| `Frameworks`                   | Space-separated list of Apple system frameworks required by the native library (for example, `Foundation UIKit Metal`).                                                                        |
| `WeakFrameworks`               | Space-separated list of Apple system frameworks to weak-link (`-weak_framework`).                                                                                                              |
| `LinkerFlags`                  | Additional flags passed to the native linker (for example, `-lsqlite3 -lz`).                                                                                                                   |
| `IsCxx`                        | Boolean. Links the C++ standard library (`libc++`) when set to `true`.                                                                                                                         |
| `NeedsGccExceptionHandling`    | Boolean. Links the GCC exception-handling library (`-lgcc_eh`) when set to `true`.                                                                                                             |
| `LinkWithSwiftSystemLibraries` | Boolean. Adds Xcode's Swift system library search paths to the native linker when the native library uses Swift.                                                                               |

## Ship native libraries in a NuGet package

There are two supported ways to distribute native Apple libraries inside a NuGet package:

1. **Standard NuGet `runtimes/<rid>/native/` layout**: best for C, C++, or Rust libraries consumed via P/Invoke in multi-targeted class libraries.
2. **Binding resource packages (`.resources` / `.resources.zip`)**: best for Objective-C bindings or native libraries that require custom `<NativeReference>` linker metadata.

### Path A: Standard NuGet `runtimes/<rid>/native/` layout

In .NET 6 and later, when an application references a NuGet package via `<PackageReference>`, the .NET SDK resolves native assets from `runtimes/<rid>/native/` inside the package for the active Runtime Identifier (RID) and feeds them to the Apple platform build automatically. Native files supported in `runtimes/<rid>/native/` include `*.a`, `*.dylib`, `*.framework`, and `*.xcframework`.

Common Apple RIDs include:

| Platform     | Runtime Identifiers (`<rid>`)                            |
|--------------|----------------------------------------------------------|
| iOS          | `ios-arm64`, `iossimulator-arm64`, `iossimulator-x64`    |
| Mac Catalyst | `maccatalyst-arm64`, `maccatalyst-x64`                   |
| macOS        | `osx-arm64`, `osx-x64`                                   |
| tvOS         | `tvos-arm64`, `tvossimulator-arm64`, `tvossimulator-x64` |

> [!NOTE]
> Starting in .NET 11, only `arm64` device and simulator RIDs are supported for iOS and tvOS (`ios-arm64`, `iossimulator-arm64`, `tvos-arm64`, `tvossimulator-arm64`).

To pack native binaries into `runtimes/<rid>/native/`, add `None` or `Content` items with `Pack="true"` and `PackagePath="runtimes/<rid>/native/"` in your library `.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net10.0;net10.0-ios;net10.0-maccatalyst;net10.0-macos</TargetFrameworks>
    <PackageId>Contoso.NativeCrypto</PackageId>
    <Version>1.0.0</Version>
  </PropertyGroup>

  <ItemGroup>
    <None Include="native/ios-arm64/libcrypto.a"
          Pack="true"
          PackagePath="runtimes/ios-arm64/native/" />
    <None Include="native/iossimulator-arm64/libcrypto.a"
          Pack="true"
          PackagePath="runtimes/iossimulator-arm64/native/" />
    <None Include="native/maccatalyst-arm64/libcrypto.dylib"
          Pack="true"
          PackagePath="runtimes/maccatalyst-arm64/native/" />
    <None Include="native/osx-arm64/libcrypto.dylib"
          Pack="true"
          PackagePath="runtimes/osx-arm64/native/" />
  </ItemGroup>
</Project>
```

Keep two important caveats in mind when using `runtimes/<rid>/native/`:

1. **Only NuGet `<PackageReference>` restores `runtimes/<rid>/native/` assets.** Creating a `runtimes/<rid>/native/` folder directly inside an application project, or referencing a class library via `<ProjectReference>`, does **not** automatically link those native assets. For local project references or in-app native binaries, use `<NativeReference>` instead.
2. **Special linker metadata is not inferred from `runtimes/<rid>/native/`.** If the native library requires linker metadata such as `ForceLoad`, `Frameworks`, `WeakFrameworks`, `IsCxx`, or `LinkerFlags`, either ship it via a binding project's binding resource package or include a `buildTransitive/<PackageId>.targets` file in the NuGet package that adds the `<NativeReference>` metadata explicitly:

```xml
<!-- buildTransitive/Contoso.NativeCrypto.targets -->
<Project>
  <ItemGroup Condition="'$(TargetPlatformIdentifier)' == 'ios' or '$(TargetPlatformIdentifier)' == 'maccatalyst'">
    <NativeReference Include="$(MSBuildThisFileDirectory)../native/ContosoCrypto.xcframework"
                     Kind="Framework"
                     SmartLink="true"
                     Frameworks="Security Foundation" />
  </ItemGroup>
</Project>
```

Legacy `PackNativeLibrary` items from earlier SDKs are not needed in .NET 6+.

### Path B: Objective-C binding NuGet packages (`.resources` / `.resources.zip` sidecar)

When you build an Objective-C binding project (`<IsBindingProject>true</IsBindingProject>`), the SDK packages all `<NativeReference>` items and their linker metadata into a binding resource package (`<AssemblyName>.resources` directory or `<AssemblyName>.resources.zip` archive) next to `<AssemblyName>.dll`.

Running `dotnet pack` automatically packs both the binding assembly (`<AssemblyName>.dll`) and its `<AssemblyName>.resources` directory or `<AssemblyName>.resources.zip` archive under `lib/<TargetFramework>/` in the resulting `.nupkg`.

For full details on binding XCFrameworks, configuring `NoBindingEmbedding` and `CompressBindingResourcePackage`, and distributing binding packages, see [Bind and distribute XCFrameworks](xcframeworks.md).

## See also

* [Bind and distribute XCFrameworks](xcframeworks.md)
* [Build items for iOS, Mac Catalyst, macOS, and tvOS (`NativeReference`, `XcodeProject`)](../building-apps/build-items.md#nativereference)
* [Build properties for iOS, Mac Catalyst, macOS, and tvOS (`NoBindingEmbedding`, `CompressBindingResourcePackage`)](../building-apps/build-properties.md#nobindingembedding)
