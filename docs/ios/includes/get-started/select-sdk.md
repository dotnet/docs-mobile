Select .NET 10 for this directory so that a newer, side-by-side SDK doesn't change the template or target framework:

```shell
dotnet new globaljson --sdk-version 10.0.100 --roll-forward latestFeature
dotnet --version
```

The reported SDK version must start with `10.0.`. This `global.json` allows installed .NET 10 SDK feature bands and servicing updates, but doesn't select .NET 11 previews.

If installing a workload reports a permissions error, follow the [.NET workload installation guidance](/dotnet/core/tools/dotnet-workload-install) for your SDK installation.
