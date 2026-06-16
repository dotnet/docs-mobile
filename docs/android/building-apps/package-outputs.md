---
title: Discover Android package outputs
description: Learn how to discover APK and Android App Bundle outputs from .NET for Android MSBuild targets.
ms.date: 06/16/2026
---

# Discover Android package outputs

When a .NET for Android project creates an APK or Android App Bundle (AAB), the final
file names and paths are resolved during the MSBuild target execution. They can depend
on properties such as [`$(AndroidPackageFormats)`](build-properties.md#androidpackageformats),
`$(Configuration)`, `$(RuntimeIdentifier)`, `$(OutputPath)`, `$(PublishDir)`, and whether
the package is signed.

Starting in .NET 11, instead of recalculating those paths in a custom target or
CI script, consume the package output item groups that are populated by the
.NET for Android build:

- [`@(AndroidPackageOutput)`](build-items.md#androidpackageoutput) contains package files
  produced in the build output directory.
- [`@(AndroidPublishedPackageOutput)`](build-items.md#androidpublishedpackageoutput)
  contains package files copied to the publish directory by `dotnet publish`.

These item groups include metadata such as the package format, whether the package is
signed, and whether an APK is the universal APK generated from an Android App Bundle.

## Capture outputs after package signing

Use `@(AndroidPackageOutput)` from a target that runs after
[`SignAndroidPackage`](build-targets.md#signandroidpackage), or call
[`GetAndroidPackageOutputs`](build-targets.md#getandroidpackageoutputs) from another target.

For example, the following target writes the build output package paths and selected
metadata to a text file:

```xml
<Project>
  <Target Name="WriteAndroidPackageOutputs" AfterTargets="SignAndroidPackage">
    <WriteLinesToFile
        File="$(OutputPath)android-package-outputs.txt"
        Lines="@(AndroidPackageOutput->'%(FullPath)|%(PackageFormat)|%(Signed)|%(IsUniversal)')"
        Overwrite="true" />
  </Target>
</Project>
```

## Capture outputs after publish

Use `@(AndroidPublishedPackageOutput)` from a target that runs after `Publish` when you
need the final files in `$(PublishDir)`:

```xml
<Project>
  <Target Name="WriteAndroidPublishedPackageOutputs" AfterTargets="Publish">
    <WriteLinesToFile
        File="$(PublishDir)android-published-package-outputs.txt"
        Lines="@(AndroidPublishedPackageOutput->'%(FullPath)|%(PackageFormat)|%(Signed)|%(OriginalPath)')"
        Overwrite="true" />
  </Target>
</Project>
```

`@(AndroidPublishedPackageOutput)` preserves the metadata from `@(AndroidPackageOutput)`
and adds `%(OriginalPath)`, which points to the corresponding artifact before it was
copied to the publish directory.

## Emit GitHub Actions outputs

In GitHub Actions, a custom target can append selected package paths to the file named by
the `GITHUB_OUTPUT` environment variable:

```xml
<Project>
  <Target Name="SetGitHubAndroidPackageOutputs"
      AfterTargets="Publish"
      Condition="'$(GITHUB_OUTPUT)' != ''">
    <ItemGroup>
      <_SignedApk Include="@(AndroidPublishedPackageOutput)"
          Condition="'%(PackageFormat)' == 'apk' and '%(Signed)' == 'true'" />
      <_SignedAab Include="@(AndroidPublishedPackageOutput)"
          Condition="'%(PackageFormat)' == 'aab' and '%(Signed)' == 'true'" />
    </ItemGroup>

    <WriteLinesToFile
        File="$(GITHUB_OUTPUT)"
        Lines="android_signed_apks=@(_SignedApk->'%(FullPath)', ',')"
        Overwrite="false" />
    <WriteLinesToFile
        File="$(GITHUB_OUTPUT)"
        Lines="android_signed_aabs=@(_SignedAab->'%(FullPath)', ',')"
        Overwrite="false" />
  </Target>
</Project>
```

This pattern lets later workflow steps upload or deploy the resolved packages without
hardcoding the package naming convention. For JSON, YAML, or other structured manifests,
pass `@(AndroidPublishedPackageOutput)` to a custom task or script and serialize the
metadata needed by your pipeline.

## Package output metadata

The package output item metadata provides the values most commonly needed by custom
targets and CI systems:

| Metadata | Description |
| --- | --- |
| `%(PackageFormat)` | `apk` or `aab`. |
| `%(Signed)` | `true` when the package is signed. |
| `%(PackageId)` | The resolved Android package name. |
| `%(RuntimeIdentifier)` | The current runtime identifier, if any. |
| `%(TargetFramework)` | The current target framework. |
| `%(Configuration)` | The current build configuration. |
| `%(AndroidPackageFormat)` | The effective primary package format. |
| `%(AndroidPackageFormats)` | The requested package formats. |
| `%(IsUniversal)` | `true` for the signed universal APK generated from an Android App Bundle. |
| `%(SourcePackageFormat)` | The package format that produced the item. |
| `%(RelativePath)` | The output file name. |
| `%(OriginalPath)` | On published items, the corresponding build output artifact before publish copy. |
