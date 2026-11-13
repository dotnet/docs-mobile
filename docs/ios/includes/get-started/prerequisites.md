These steps use the command line on a Mac. You need:

- A Mac running a version of macOS supported by both your .NET SDK and Xcode.
- The latest servicing release of the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0). Install the Arm64 SDK on an Apple silicon Mac or the x64 SDK on an Intel Mac.
- The full Xcode application, not just the Command Line Tools package. Each Apple workload requires a **specific Xcode version**. Before installing a workload, check the [.NET for Apple platforms release notes](https://github.com/dotnet/macios/releases) for a .NET 10 release and its required Xcode and workload set versions. See [Xcode requirement](/dotnet/ios/troubleshooting/xcode-requirement) if you need to select another Xcode or install an older workload set.

Open Xcode once to accept its license and install its required components. Check the selected Xcode installation:

```shell
xcode-select -p
xcodebuild -version
```

If these commands don't identify the required Xcode application, select it in **Xcode > Settings > Locations > Command Line Tools** before continuing.

> [!IMPORTANT]
> The workload installation commands below install the workload selected by your .NET SDK's workload set. If you need the workload set specified in a particular release's notes, append `--version <workload-set-version>` to the installation command, replacing the placeholder with that release's workload set version. Don't assume that the newest workload works with an older Xcode.
