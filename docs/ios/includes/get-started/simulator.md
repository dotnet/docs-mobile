In **Xcode > Settings > Components**, install a simulator runtime for the platform you're targeting if one isn't installed already. Depending on your Xcode version, this section might be called **Platforms**. In **Window > Devices and Simulators > Simulators**, create a simulator for that runtime if needed.

List available simulators:

```shell
xcrun simctl list devices available
```

Copy the UDID (the identifier in parentheses) of a simulator for this tutorial's platform. Replace `<UDID>` in the run command below with that identifier; don't include the angle brackets. The run target builds the app, starts the selected simulator, and launches the app.

> [!NOTE]
> These .NET 10 commands use `_DeviceName` to select the simulator. The `Device` property and `dotnet run --device` device-selection support require .NET 11; don't substitute them in this tutorial.
