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
CI script, consume the [`@(ApplicationArtifact)`](build-items.md#applicationartifact)
item group that is populated by the .NET for Android build.

This item group includes metadata such as the package format, whether the package is
signed, the resolved package ID, and the ABI for per-ABI APKs.

## Capture outputs after package signing

Use `@(ApplicationArtifact)` from a target that runs after
[`SignAndroidPackage`](build-targets.md#signandroidpackage), or call
[`GetApplicationArtifacts`](build-targets.md#getapplicationartifacts) from another target.

For example, the following target writes the build output package paths and selected
metadata to a text file:

```xml
<Project>
  <Target Name="WriteApplicationArtifacts" AfterTargets="SignAndroidPackage">
    <WriteLinesToFile
        File="$(OutputPath)application-artifacts.txt"
        Lines="@(ApplicationArtifact->'%(FullPath)|%(PackageFormat)|%(Signed)|%(PackageId)|%(Abi)')"
        Overwrite="true" />
  </Target>
</Project>
```

## Extend application artifact metadata

Targets imported after .NET for Android can append to
`$(GetApplicationArtifactsDependsOn)` to update or enrich `@(ApplicationArtifact)`
before [`GetApplicationArtifacts`](build-targets.md#getapplicationartifacts)
or `Publish` returns the items. For example:

```xml
<Project>
  <PropertyGroup>
    <GetApplicationArtifactsDependsOn>
      $(GetApplicationArtifactsDependsOn);
      AddCustomApplicationArtifactMetadata
    </GetApplicationArtifactsDependsOn>
  </PropertyGroup>

  <Target Name="AddCustomApplicationArtifactMetadata">
    <ItemGroup>
      <ApplicationArtifact Update="@(ApplicationArtifact)" CustomMetadata="value" />
    </ItemGroup>
  </Target>
</Project>
```

## Capture outputs after publish

Use `@(ApplicationArtifact)` from a target that runs after `Publish` when you need the
final files in `$(PublishDir)`. During publish, .NET for Android updates the item
identities to the copied publish-directory paths while preserving package metadata:

```xml
<Project>
  <Target Name="WritePublishedApplicationArtifacts" AfterTargets="Publish">
    <WriteLinesToFile
        File="$(PublishDir)application-artifacts.txt"
        Lines="@(ApplicationArtifact->'%(FullPath)|%(PackageFormat)|%(Signed)|%(PackageId)|%(Abi)')"
        Overwrite="true" />
  </Target>
</Project>
```

## Emit GitHub Actions outputs

In GitHub Actions, a custom target can append selected package paths to the file named by
the `GITHUB_OUTPUT` environment variable:

```xml
<Project>
  <Target Name="SetGitHubApplicationArtifacts"
      AfterTargets="Publish"
      Condition="'$(GITHUB_OUTPUT)' != ''">
    <ItemGroup>
      <_SignedApk Include="@(ApplicationArtifact)"
          Condition="'%(PackageFormat)' == 'apk' and '%(Signed)' == 'true'" />
      <_SignedAab Include="@(ApplicationArtifact)"
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
pass `@(ApplicationArtifact)` to a custom task or script and serialize the
metadata needed by your pipeline.

## Package output metadata

The package output item metadata provides the values most commonly needed by custom
targets and CI systems:

| Metadata | Description |
| --- | --- |
| `%(PackageFormat)` | `apk` or `aab`. |
| `%(Signed)` | `true` when the package is signed. |
| `%(PackageId)` | The resolved Android package name. |
| `%(Abi)` | The Android ABI for per-ABI APK outputs. This metadata is omitted for non-per-ABI outputs. |
