---
title: "Install .NET for Android"
description: "Learn how to install .NET for Android so you can create native android applications."
ms.date: 11/01/2023
---
# Install .NET for Android

Developing native, .NET for Android apps requires dotnet 6 or higher. Various IDE's can be used however
we recommend Visual Studio 2022 17.3 or greater, or Visual Studio Code.

<!-- markdownlint-disable MD025 -->
## [Install via the Command Prompt or Terminal](#tab/commandline)
<!-- markdownlint-enable MD025 -->

* Install the latest .net <https://dotnet.microsoft.com/download> for your particular platform
  and follow its installation instructions <dotnet/core/install>.

* From a Command Prompt or Terminal run:

    ```dotnet
    dotnet workload install android
    ```

* In order to build Android applications you also need to install the [Android SDK and Java Sdk](dependencies.md#using-installandroiddependencies).


<!-- markdownlint-disable MD025 -->
## [Install via Visual Studio](#tab/visualstudio)
<!-- markdownlint-enable MD025 -->

* Install the latest Visual Studio <https://visualstudio.microsoft.com/downloads/>
* Select the .NET Multi Platform App UI Development and any other workloads you want.

  ![Select .Net Multi Platform App UI WorkLoad](images/vs-install-select-maui.png)
* Or select the .NET for Android SDK from the Individual Components Tab.

  ![Select .NET for Android SDK Component](images/vs-install-select-android-components.png)
* Let the installer run, it may take a while depending on your Internet Connection.

  ![The Running Installer](images/vs-install-installing.png)
* Once installed you can run Visual Studio. You will be presented with the start up screen.
  Select New Project

  ![Select the New Project Menu](images/vs-new-project.png)
* Look through the templates to find the Android Application Template

  ![Select the Android Application Template](images/vs-select-android-application.png)
