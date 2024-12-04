---
title: "Images and Icons in .NET for iOS, tvOS, macOS and Mac Catalyst"
description: "This section includes a variety of articles that cover working with images in a Xamarin.iOS app, such as using them as icons, launch screens or including them in controls and providing icons for custom document types."
ms.date: 11/01/2024
---
# Images and icons in .NET for iOS, tvOS, macOS and Mac Catalyst

There are several ways that image assets are used inside an app. From simply
displaying an image as part of an app's UI to, assigning it to a UI control
such as a [UIButton][uibutton] or [UIImageView][uiimageview], to providing icons and launch
screens, .NET for iOS, tvOS, macOS and Mac Catalyst makes it easy to add great
artwork to an app in the following ways:

- **Resolution Independent Images** – Use the OS's built-in support for working with images across different device resolutions and types (iPhone, iPad, etc.).
- **Asset Catalog Image Sets** - Use **Asset Catalog Image Sets** to manage and group all version of a given image asset required by an app.
- **Images in Code** – Use the [UIImage][uiimage] class's methods to load and work with image assets and assign them to UI controls in C# code.
- **Application Icon** - Define the app icon required by every app. This is the icon that the user will tap from the home screen to launch the app. Additionally, this icon is used by Game Center, if applicable.
- **Spotlight Icon** - Define the app's Spotlight icon. Whenever the user enters the name of an app in a Spotlight Search, this icon is displayed.
- **Settings Icon** - Define the app's **Settings** icon. If the user enters the **Settings** app on their device, this icon will be displayed at the end of the Settings list for the app. 
- **Launch Screens** - Define the app's Launch Screen. After the user taps the app icon and before the first view appears, a blank screen will be shown. Fortunately, it's possible to displaying an image in place of the blank screen by using a Storyboard. 
- **iTunes Icon** - Provide an iTunes icon. If using the Ad-Hoc method of delivering an app (either for corporate users or for beta testing on real devices), the developer also needs to include a 512x512 and a 1024x1024 image that will be used to represent the app in iTunes.
- **Document Icons** - Use an image as an icon for any specific document type that an app supports or creates.

There are several considerations that should be taken into account when
creating image assets for an app, as well as several places where those assets
will be used. Each of these have an affect on not only how many image assets
will be required, but how those assets are created. The following topics cover
the types of images assets that will be required, how those assets are
included in the application's bundle and how the image assets are consumed to
provide the required functionality:

## [Alternate app icons](~/ios/app-fundamentals/images-icons/alternate-app-icons.md)

Apple has several [UIApplication][uiapplication] APIs that allow an app to manage its icon:

- [UIApplication.SupportsAlternateIcons][supportsalternateicons] - If `true` the app has an alternate set of icons.
- [UIApplication.AlternateIconName][alternateiconname] - Returns the name of the alternate icon currently selected or `null` if using the primary icon.
- [UIApplication.SetAlternateIconName][setalternateiconname] - Use this method to switch the app's icon to the given alternate icon.
- `UNUserNotificationCenter.Current.SetBadgeCount` - Sets the badge count of the app icon in the Springboard (deprecated in iOS 16+ and tvOS 16+).

[uiapplication]: xref:UIKit.UIApplication
[applicationiconbadgenumber]: xref:UIKit.UIApplication.ApplicationIconBadgeNumber
[supportsalternateicons]: xref:UIKit.UIApplication.SupportsAlternateIcons
[alternateiconname]: xref:UIKit.UIApplication.AlternateIconName
[setalternateiconname]: xref:UIKit.UIApplication.SetAlternateIconName%2A
[uibutton]: xref:UIKit.UIButton
[uiimageview]: xref:UIKit.UIImageView
[uiimage]: xref:UIKit.UIImage
