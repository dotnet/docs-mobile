---
title: .NET for Android Build Items
description: .NET for Android Build Items
ms.date: 04/11/2024
---

# Build Items

Build items control how a .NET for Android application
or library project is built.

They're specified within the project file, for example **MyApp.csproj**, within
an [MSBuild PropertyGroup](/visualstudio/msbuild/itemgroup-element-msbuild).

> [!NOTE]
> In .NET for Android there is technically no distinction between an application and a bindings project, so build items will work in both. In practice it is highly recommended to create separate application and bindings projects. Build items that are primarily used in bindings projects are documented in the [MSBuild bindings project items](../binding-libs/msbuild-reference/build-items.md) reference guide.

## AndroidAsset

Supports [Android Assets](https://developer.android.com/guide/topics/resources/providing-resources#OriginalFiles),
files that would be included in the `assets` folder in a Java Android project.

Starting with .NET 9 the `@(AndroidAsset)` build action also supports additional metadata for generating [Asset Packs](https://developer.android.com/guide/playcore/asset-delivery). The `%(AndroidAsset.AssetPack)` metadata can be used to automatically generate an asset pack of that name. This feature is only supported when the [`$(AndroidPackageFormat)`](build-properties.md#androidpackageformat) is set to `.aab`. The following example will place `movie2.mp4` and `movie3.mp4` in separate asset packs.

```xml
<ItemGroup>
   <AndroidAsset Update="Asset/movie.mp4" />
   <AndroidAsset Update="Asset/movie2.mp4" AssetPack="assets1" />
   <AndroidAsset Update="Asset/movie3.mp4" AssetPack="assets2" />
</ItemGroup>
```

This feature can be used to include large files in your application which would normally exceed the max
package size limits of Google Play.

If you have a large number of assets it might be more efficient to make use of the `base` asset pack.
In this scenario you update ALL assets to be in a single asset pack then use the `AssetPack="base"` metadata
to declare which specific assets end up in the base aab file. With this you can use wildcards to move most
assets into the asset pack.

```xml
<ItemGroup>
   <AndroidAsset Update="Assets/*" AssetPack="assets1" />
   <AndroidAsset Update="Assets/movie.mp4" AssetPack="base" />
   <AndroidAsset Update="Assets/some.png" AssetPack="base" />
</ItemGroup>
```

In this example, `movie.mp4` and `some.png` will end up in the `base` aab file, while all the other assets
will end up in the `assets1` asset pack.

The additional metadata is only supported on .NET for Android 9 and above.

## AndroidAotProfile

Used to provide an AOT profile, for use with profile-guided AOT.

It can be also used from Visual Studio by setting the `AndroidAotProfile`
build action to a file containing an AOT profile.

## AndroidAppBundleMetaDataFile

Specifies a file that will be included as metadata in the Android App Bundle.
The format of the flag value is `<bundle-path>:<physical-file>` where
`bundle-path` denotes the file location inside the App Bundle's metadata
directory, and `physical-file` is an existing file containing the raw data
to be stored.

```xml
<ItemGroup>
  <AndroidAppBundleMetaDataFile
    Include="com.android.tools.build.obfuscation/proguard.map:$(OutputPath)mapping.txt"
  />
</ItemGroup>
```

See [bundletool](https://developer.android.com/studio/build/building-cmdline#build_your_app_bundle_using_bundletool) documentation for more details.

## AndroidBoundLayout

Indicates that the layout file is to have [code-behind generated](../features/layout-code-behind/index.md)
for it in case when the [`$(AndroidGenerateLayoutBindings)`](build-properties.md#androidgeneratelayoutbindings)
property is set to `false`. In all other aspects
it is identical to [`AndroidResource`](#androidresource).

This action can be used **only** with layout files:

```xml
<AndroidBoundLayout Include="Resources\layout\Main.axml" />
```

## AndroidEnvironment

Files with a Build action of `AndroidEnvironment` are used
to [initialize environment variables and system properties during process startup](/xamarin/android/deploy-test/environment).
The `AndroidEnvironment` Build action may be applied to
multiple files, and they will be evaluated in no particular order (so don't
specify the same environment variable or system property in multiple
files).

## AndroidLintConfig

The Build action 'AndroidLintConfig' should be used in conjunction with the
[`$(AndroidLintEnabled)`](/xamarin/android/deploy-test/building-apps/build-properties#androidlintenabled)
property. Files with this build action will be merged together and passed to the
android `lint` tooling. They should be XML files containing information on
tests to enable and disable.

See the [lint documentation](https://developer.android.com/studio/write/lint)
for more details.

## AndroidManifestOverlay

The `AndroidManifestOverlay` build action can be used to provide
`AndroidManifest.xml` files to the [Manifest Merger](https://developer.android.com/studio/build/manifest-merge) tool.
Files with this build action will be passed to the Manifest Merger along with
the main `AndroidManifest.xml` file and manifest files from
references. These will then be merged into the final manifest.

You can use this build action to provide changes and settings to
your app depending on your build configuration. For example, if you need to
have a specific permission only while debugging, you can use the overlay to
inject that permission when debugging. For example, given the following
overlay file contents:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
  <uses-permission android:name="android.permission.CAMERA" />
</manifest>
```

You can use the following to add a manifest overlay for a debug build:

```xml
<ItemGroup>
  <AndroidManifestOverlay Include="DebugPermissions.xml" Condition=" '$(Configuration)' == 'Debug' " />
</ItemGroup>
```

## AndroidInstallModules

Specifies the modules that get installed by **bundletool** command when
installing app bundles.

## AndroidNativeLibrary

[Native libraries](/xamarin/android/platform/native-libraries)
are added to the build by setting their Build action to
`AndroidNativeLibrary`.

Note that since Android supports multiple Application Binary Interfaces
(ABIs), the build system must know the ABI the native library is
built for. There are two ways the ABI can be specified:

1. Path "sniffing".
2. Using the  `%(Abi)` item metadata.

With path sniffing, the parent directory name of the native library is
used to specify the ABI that the library targets. Thus, if you add
`lib/armeabi-v7a/libfoo.so` to the build, then the ABI will be "sniffed" as
`armeabi-v7a`.

### Item Attribute Name

**Abi** &ndash; Specifies the ABI of the native library.

```xml
<ItemGroup>
  <AndroidNativeLibrary Include="path/to/libfoo.so">
    <Abi>armeabi-v7a</Abi>
  </AndroidNativeLibrary>
</ItemGroup>
```

## AndroidPackagingOptionsExclude

A set of file glob compatible items which will allow for items to be
excluded from the final package. The default values are as follows

```xml
<ItemGroup>
	<AndroidPackagingOptionsExclude Include="DebugProbesKt.bin" />
	<AndroidPackagingOptionsExclude Include="$([MSBuild]::Escape('*.kotlin_*')" />
</ItemGroup>
```

Items can use file blob characters for wildcards such as `*` and `?`.
However these Items MUST be URL encoded or use
[`$([MSBuild]::Escape(''))`](/visualstudio/msbuild/how-to-escape-special-characters-in-msbuild).
This is so MSBuild does not try to interpret them as actual file wildcards.

For example

```xml
<ItemGroup>
	<AndroidPackagingOptionsExclude Include="%2A.foo_%2A" />
  <AndroidPackagingOptionsExclude Include="$([MSBuild]::Escape('*.foo')" />
</ItemGroup>
```

NOTE: `*`, `?` and `.` will be replaced in the `BuildApk` task with the
appropriate file globs.

If the default file glob is too restrictive you can remove it by adding the
following to your csproj

```xml
<ItemGroup>
	<AndroidPackagingOptionsExclude Remove="$([MSBuild]::Escape('*.kotlin_*')" />
</ItemGroup>
```

Added in .NET 7.

## AndroidPackagingOptionsInclude

A set of file glob compatible items which will allow for items to be
included from the final package. The default values are as follows

```xml
<ItemGroup>
	<AndroidPackagingOptionsInclude Include="$([MSBuild]::Escape('*.kotlin_builtins')" />
</ItemGroup>
```

Items can use file blob characters for wildcards such as `*` and `?`.
However these Items MUST use URL encoding or '$([MSBuild]::Escape(''))'.
This is so MSBuild does not try to interpret them as actual file wildcards.
For example

```xml
<ItemGroup>
	<AndroidPackagingOptionsInclude Include="%2A.foo_%2A" />
  <AndroidPackagingOptionsInclude Include="$([MSBuild]::Escape('*.foo')" />
</ItemGroup>
```

NOTE: `*`, `?` and `.` will be replaced in the `BuildApk` task with the
appropriate file globs.

Added in .NET 9.

## AndroidResource

All files with an *AndroidResource* build action are compiled into
Android resources during the build process and made accessible via `$(AndroidResgenFile)`.

```xml
<ItemGroup>
  <AndroidResource Include="Resources\values\strings.xml" />
</ItemGroup>
```

More advanced users might perhaps wish to have different resources used in
different configurations but with the same effective path. This can be achieved
by having multiple resource directories and having files with the same relative
paths within these different directories, and using MSBuild conditions to
conditionally include different files in different configurations. For
example:

```xml
<ItemGroup Condition=" '$(Configuration)' != 'Debug' ">
  <AndroidResource Include="Resources\values\strings.xml" />
</ItemGroup>
<ItemGroup  Condition=" '$(Configuration)' == 'Debug' ">
  <AndroidResource Include="Resources-Debug\values\strings.xml"/>
</ItemGroup>
<PropertyGroup>
  <MonoAndroidResourcePrefix>Resources;Resources-Debug</MonoAndroidResourcePrefix>
</PropertyGroup>
```

**LogicalName** &ndash; Specifies the resource path explicitly. Allows
&ldquo;aliasing&rdquo; files so that they will be available as multiple
distinct resource names.

```xml
<ItemGroup Condition="'$(Configuration)'!='Debug'">
  <AndroidResource Include="Resources/values/strings.xml"/>
</ItemGroup>
<ItemGroup Condition="'$(Configuration)'=='Debug'">
  <AndroidResource Include="Resources-Debug/values/strings.xml">
    <LogicalName>values/strings.xml</LogicalName>
  </AndroidResource>
</ItemGroup>
```

## Content

The normal `Content` Build action is not supported (as we
haven't figured out how to support it without a possibly costly first-run
step).

Attempting to use the `@(Content)` Build action will result in a
[XA0101](../messages/xa0101.md) warning.

## EmbeddedNativeLibrary

In a .NET for Android class library or Java binding project, the
**EmbeddedNativeLibrary** build action bundles a native library such
as `lib/armeabi-v7a/libfoo.so` into the library. When a
.NET for Android application consumes the library, the `libfoo.so` file
will be included in the final Android application.

You can use the
[**AndroidNativeLibrary**](#androidnativelibrary) build action as an
alternative.

## LinkDescription

Files with a *LinkDescription* build action are used to
[control linker behavior](/xamarin/cross-platform/deploy-test/linker).

## ProguardConfiguration

Files with a *ProguardConfiguration* build action contain options which
are used to control `proguard` behavior. For more information about
this build action, see
[ProGuard](/xamarin/android/deploy-test/release-prep/proguard).

These files are ignored unless the
[`$(EnableProguard)`](/xamarin/android/deploy-test/building-apps/build-properties#enableproguard)
MSBuild property is `True`.
