# Xamarin.Forms to .NET MAUI (.NET 9) Migration Guide

This guide provides detailed, step-by-step instructions for migrating a Xamarin.Forms application to .NET MAUI targeting .NET 9. It includes specific examples and can be used by GitHub Copilot agents to assist with migration tasks.

## Step 1: Organize Sample Folders in 9.0 and 10.0

Create the `9.0` and `10.0` folders if they do not exist. Inside `9.0`, find or create a subfolder that best matches the app's category (e.g., `data`, `ui`, `maps`). Use clear, descriptive folder names. If no suitable category exists, create one that makes sense for the app. Refer to the `Reference` folder for examples of folder naming, structure, and best practices.

## Step 2: In the folder Create New .NET MAUI Project Structure

### 2.1 Create New .NET MAUI Project

**IMPORTANT: Project and Solution Naming**
- **Solution File**: Use the **original project name** from the Xamarin.Forms solution (e.g., `TapGesture.sln`, `PinchGesture.sln`, `DragAndDropGesture.sln`), NOT the master folder name
- **Project Name**: Use the **original project name** from the Xamarin.Forms shared project (e.g., `WorkingWithGestures`, `TapGesture`, `PinchGesture`)
- **RootNamespace**: Should match the **original project's namespace** from the Xamarin.Forms project

### 2.2 Create a Project Structure

**Single Project Structure:**
```
[OriginalProjectName].sln          # Use ORIGINAL project name, not folder name
Screenshots/
README.md
[OriginalProjectName]/             # Use ORIGINAL project name
├── Platforms/
│   ├── Android/
│   ├── iOS/
│   ├── MacCatalyst/
│   ├── Windows/
│   └── Tizen/ (optional)
├── Resources/
│   ├── AppIcon/
│   ├── Fonts/
│   ├── Images/
│   └── Splash/
├── [OriginalProjectName].csproj    # Use ORIGINAL project name
├── App.xaml
├── App.xaml.cs
├── AppShell.xaml (optional)
├── AppShell.xaml.cs (optional)
├── MainPage.xaml (optional)
├── MainPage.xaml.cs (optional)
├── MauiProgram.cs
└── GlobalUsings.cs
```

**Example**: If migrating `TapGesture` from `WorkingWithGestures` folder:
- Solution: `TapGesture.sln` (NOT `WorkingWithGestures.sln` that is the root folder, not the project)
- Project: `TapGesture/TapGesture.csproj`
- RootNamespace: `TapGesture`

Add any missing files. Ensure the `Platforms` folder contains all required platform subfolders, even if some are empty. Always include a `README.md` and at least one screenshot in the `Screenshots` folder to document the sample.

### 2.3 Copy Necessary Parts from the Original App

**CRITICAL: Copy ALL relevant files from the Xamarin.Forms project**

1. **Copy ALL .cs, .xaml, and .xaml.cs files** from the original Xamarin.Forms shared project
2. **Copy ALL Resources** from the original Xamarin.Forms project:
   - Images → `Resources/Images/`
   - Fonts → `Resources/Fonts/`
   - Other assets → `Resources/Raw/`
3. **Preserve the original project structure** - don't reorganize files unless necessary
4. **Copy platform-specific files** from Android/iOS projects to appropriate `Platforms/` folders
5. **Review and modernize** code where possible (e.g., use new C# features, simplify logic, remove obsolete patterns)

**Example File Migration:**
```
Original Xamarin.Forms project structure:
WorkingWithGestures/
├── App.cs
├── TapInsideFrame.cs
├── TapInsideFrameXaml.xaml
├── TapInsideFrameXaml.xaml.cs
├── TapInsideImage.cs
├── TapViewModel.cs
└── WorkingWithGestures.csproj

New .NET MAUI project structure:
WorkingWithGestures/
├── App.xaml                    # Create new
├── App.xaml.cs                 # Create new (migrate from App.cs)
├── MauiProgram.cs              # Create new
├── GlobalUsings.cs             # Create new
├── TapInsideFrame.cs           # Copy from original
├── TapInsideFrameXaml.xaml     # Copy from original
├── TapInsideFrameXaml.xaml.cs  # Copy from original
├── TapInsideImage.cs           # Copy from original
├── TapViewModel.cs             # Copy from original
├── Platforms/                  # Create new
├── Resources/                  # Create new and copy assets
└── WorkingWithGestures.csproj  # Transform to MAUI format
```

Review the `Reference` folder for examples of well-migrated code and structure.

## Step 3: Update Project File (.csproj)

**CRITICAL: Complete Project File Transformation**

The .csproj file must be completely transformed from Xamarin.Forms format to .NET MAUI format. This is NOT just an update - it's a complete rewrite using the SDK-style format.

### 3.1 Transform Project File to SDK-Style

**⚠️ CRITICAL: Project File Migration Requirements**

When migrating from Xamarin.Forms to .NET MAUI, the following project file changes are mandatory:

1. **Convert to SDK-style projects**: All Xamarin.Forms class library, Xamarin.iOS, and Xamarin.Android projects must be converted to SDK-style projects
2. **Update target frameworks**: Change to `net9.0-android`, `net9.0-ios`, `net9.0-maccatalyst`, etc.
3. **Set UseMaui property**: Add `<UseMaui>true</UseMaui>` to enable MAUI
4. **Remove incompatible packages**: Remove all NuGet packages incompatible with .NET 9
5. **Update package references**: Replace Xamarin-specific packages with MAUI equivalents
6. **Replace obsolete APIs**: Replace APIs that are no longer needed
7. **Add required properties**: Add new MAUI-specific properties

**Before (Xamarin.Forms shared project):**
```xml
<Project>
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>
  
  <ItemGroup>
    <PackageReference Include="Xamarin.Forms" Version="5.0.0.2654" />
    <PackageReference Include="Xamarin.Essentials" Version="1.8.1" />
    etc.
  </ItemGroup>
</Project>
```

**After (.NET MAUI project - COMPLETE REPLACEMENT):**
Replace the ENTIRE .csproj file content with the following template and customize the `RootNamespace`, `ApplicationTitle`, and `ApplicationId` to match your **original project**:

```xml
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFrameworks>net9.0-android;net9.0-ios;net9.0-maccatalyst</TargetFrameworks>
        <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net9.0-windows10.0.19041.0</TargetFrameworks>
		<!-- Uncomment to also build the tizen app. You will need to install tizen by following this: https://github.com/Samsung/Tizen.NET -->
		<!-- <TargetFrameworks>$(TargetFrameworks);net9.0-tizen</TargetFrameworks> -->

		<!-- Note for MacCatalyst:
		The default runtime is maccatalyst-x64, except in Release config, in which case the default is maccatalyst-x64;maccatalyst-arm64.
		When specifying both architectures, use the plural <RuntimeIdentifiers> instead of the singular <RuntimeIdentifier>.
		The Mac App Store will NOT accept apps with ONLY maccatalyst-arm64 indicated;
		either BOTH runtimes must be indicated or ONLY macatalyst-x64. -->
		<!-- For example: <RuntimeIdentifiers>maccatalyst-x64;maccatalyst-arm64</RuntimeIdentifiers> -->

		<OutputType>Exe</OutputType>
		<RootNamespace><!-- USE ORIGINAL PROJECT NAMESPACE --></RootNamespace>
		<UseMaui>true</UseMaui>
		<SingleProject>true</SingleProject>
		<ImplicitUsings>enable</ImplicitUsings>
		<Nullable>enable</Nullable>

		<!-- Display name -->
		<ApplicationTitle><!-- USE ORIGINAL PROJECT NAME --></ApplicationTitle>

		<!-- App Identifier -->
		<ApplicationId>com.companyname.<!-- USE ORIGINAL PROJECT NAME IN LOWERCASE --></ApplicationId>

		<!-- Versions -->
		<ApplicationDisplayVersion>1.0</ApplicationDisplayVersion>
		<ApplicationVersion>1</ApplicationVersion>

		<!-- To develop, package, and publish an app to the Microsoft Store, see: https://aka.ms/MauiTemplateUnpackaged -->
		<WindowsPackageType>None</WindowsPackageType>

		<SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'ios'">15.0</SupportedOSPlatformVersion>
		<SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'maccatalyst'">15.0</SupportedOSPlatformVersion>
		<SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'android'">21.0</SupportedOSPlatformVersion>
		<SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">10.0.17763.0</SupportedOSPlatformVersion>
		<TargetPlatformMinVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'windows'">10.0.17763.0</TargetPlatformMinVersion>
		<SupportedOSPlatformVersion Condition="$([MSBuild]::GetTargetPlatformIdentifier('$(TargetFramework)')) == 'tizen'">6.5</SupportedOSPlatformVersion>
	</PropertyGroup>

	<ItemGroup>
		<!-- App Icon -->
		<MauiIcon Include="Resources\AppIcon\appicon.svg" ForegroundFile="Resources\AppIcon\appiconfg.svg" Color="#512BD4" />

		<!-- Splash Screen -->
		<MauiSplashScreen Include="Resources\Splash\splash.svg" Color="#512BD4" BaseSize="128,128" />

		<!-- Images -->
		<MauiImage Include="Resources\Images\*" />
		<MauiImage Update="Resources\Images\dotnet_bot.png" Resize="True" BaseSize="300,185" />

		<!-- Custom Fonts -->
		<MauiFont Include="Resources\Fonts\*" />

		<!-- Raw Assets (also remove the "Resources\Raw" prefix) -->
		<MauiAsset Include="Resources\Raw\**" LogicalName="%(RecursiveDir)%(Filename)%(Extension)" />
	</ItemGroup>

  <ItemGroup>
        <PackageReference Include="Microsoft.Maui.Controls" Version="$(MauiVersion)" />
        <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="9.0.0" />
        <!-- any other packages from the original xamarin app, that are still supported  -->
  </ItemGroup>

</Project>
```

### 3.2 Remove Obsolete Package References

**⚠️ CRITICAL: Remove these packages completely:**
- `Xamarin.Forms` - Replaced by `Microsoft.Maui.Controls`
- `Xamarin.Essentials` - Integrated into .NET MAUI
- `Xamarin.AndroidX.*` (if explicitly referenced)
- `Xamarin.iOS`
- Any other `Xamarin.*` packages

**⚠️ CRITICAL: Replace with MAUI equivalents:**
- `Xamarin.CommunityToolkit` → `CommunityToolkit.Maui`
- `SkiaSharp.Views.Forms` → `SkiaSharp.Views.Maui.Controls`
- `Xamarin.Forms.Maps` → `Microsoft.Maui.Controls.Maps`
- `Lottie.Forms` → `SkiaSharp.Extended.UI.Maui`

**Add these packages:**
- `Microsoft.Maui.Controls` with `$(MauiVersion)`
- `Microsoft.Extensions.Logging.Debug` with version `9.0.0`

## Step 4: Update Namespace References

Tip: Use global usings to simplify code. Remove unnecessary or obsolete using statements. See `GlobalUsings.cs` examples in the `Reference` folder.

### 4.1 Update Using Statements

**Before:**
```csharp
using Xamarin.Forms;
using Xamarin.Essentials;
```

**After:**
```csharp
using Microsoft.Maui.Controls;
using Microsoft.Maui.Essentials;
// Note: Many usings can be removed due to global usings in .NET 9
```

### 4.2 Update XAML Namespace Declarations

**Before:**
```xml
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">
```

**After:**
```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">
```

### 4.3 Global Using Directives (GlobalUsings.cs)

Create a `GlobalUsings.cs` file in your project root:
```csharp
// Global using directives for .NET MAUI
global using Microsoft.Maui.Controls;
global using Microsoft.Maui.Controls.Xaml;
global using Microsoft.Maui.Essentials;
global using System.Collections.ObjectModel;
global using System.ComponentModel;
global using System.Windows.Input;
```

**⚠️ CRITICAL: Keep GlobalUsings.cs Minimal**

Only include commonly used namespaces that are used across multiple files in your project. Avoid adding rarely used namespaces to keep the global scope clean. Remove any unused global using statements to prevent compilation issues.

## Step 5: Migrate Application Startup

Tip: Use dependency injection for services and follow the recommended startup pattern. Review `MauiProgram.cs` in the `Reference` folder for best practices.

### 5.1 Create MauiProgram.cs

```csharp
using Microsoft.Extensions.Logging;

namespace YourAppName;

public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
            .ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
            });

        // Add services for dependency injection
        builder.Services.AddSingleton<IConnectivity>(Connectivity.Current);
        builder.Services.AddSingleton<IGeolocation>(Geolocation.Default);
        builder.Services.AddSingleton<IMap>(Map.Default);

#if DEBUG
        builder.Logging.AddDebug();
#endif

        return builder.Build();
    }
}
```

### 5.2 Update App.xaml and App.xaml.cs

**App.xaml:**
```xml
<Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="YourAppName.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Styles/Colors.xaml" />
                <ResourceDictionary Source="Resources/Styles/Styles.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

**App.xaml.cs:**
```csharp
namespace YourAppName;

public partial class App : Application
{
    public App()
    {
        InitializeComponent();
    }

    protected override Window CreateWindow(IActivationState? activationState)
    {
        return new Window(new AppShell());
    }
}
```

**⚠️ Critical App.xaml.cs Pattern**

**NEVER use the old Xamarin.Forms pattern:**
```csharp
// ❌ WRONG - Don't do this in .NET MAUI
public App()
{
    InitializeComponent();
    MainPage = new AppShell(); // This is Xamarin.Forms pattern
}
```

**✅ Always use the .NET MAUI pattern:**
```csharp
// ✅ CORRECT - .NET MAUI pattern
public App()
{
    InitializeComponent();
}

protected override Window CreateWindow(IActivationState? activationState)
{
    return new Window(new AppShell());
}
```

This ensures proper application lifecycle management and enables multi-window support.

## Step 6: Migrate Pages and Views

Tip: Update XAML and code-behind to use .NET MAUI conventions. Refactor code to use MVVM and Shell navigation if possible. Check the `Reference` folder for modern page and view implementations.

### 6.1 Copy All Pages and Views
Copy all `.xaml` and `.xaml.cs` files from your Xamarin.Forms project to the .NET MAUI project.

### 6.2 Critical Layout Changes

**⚠️ CRITICAL: Review ALL Grid Layouts**

Every `Grid` must have explicit `RowDefinitions` and `ColumnDefinitions`. .NET MAUI does not auto-generate them:

```xml
<!-- ❌ WRONG - Missing explicit definitions -->
<Grid>
    <Label Text="Hello" Grid.Row="0"/>
    <Label Text="World" Grid.Row="1"/>
</Grid>

<!-- ✅ CORRECT - Explicit definitions required -->
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <Label Text="Hello" Grid.Row="0"/>
    <Label Text="World" Grid.Row="1"/>
</Grid>
```

**⚠️ CRITICAL: Remove ALL *AndExpand Usage**

Search for and remove all `*AndExpand` usage in `StackLayout`, `HorizontalStackLayout`, and `VerticalStackLayout`:

```csharp
// ❌ WRONG - AndExpand is treated as regular Fill
VerticalOptions = LayoutOptions.StartAndExpand
HorizontalOptions = LayoutOptions.FillAndExpand

// ✅ CORRECT - Remove AndExpand
VerticalOptions = LayoutOptions.Start
HorizontalOptions = LayoutOptions.Fill
```

**⚠️ CRITICAL: RelativeLayout Updates**

Identify and update ALL `RelativeLayout` implementations. Refactor to use `Grid` instead:

```xml
<!-- ❌ WRONG - RelativeLayout is deprecated -->
<RelativeLayout>
    <Label Text="Hello" RelativeLayout.XConstraint="0" RelativeLayout.YConstraint="0"/>
</RelativeLayout>

<!-- ✅ CORRECT - Use Grid -->
<Grid>
    <Label Text="Hello" Grid.Row="0" Grid.Column="0"/>
</Grid>
```

**⚠️ CRITICAL: Add Implicit Styles for Default Spacing**

Add implicit styles to resource dictionaries to preserve Xamarin.Forms default spacing values:

```xml
<!-- Add to App.xaml or resource dictionary -->
<Application.Resources>
    <ResourceDictionary>
        <!-- Preserve Xamarin.Forms default spacing -->
        <Style TargetType="Grid">
            <Setter Property="ColumnSpacing" Value="6"/>
            <Setter Property="RowSpacing" Value="6"/>
        </Style>
        <Style TargetType="StackLayout">
            <Setter Property="Spacing" Value="6"/>
        </Style>
        <Style TargetType="Frame">
            <Setter Property="Padding" Value="20"/>
        </Style>
    </ResourceDictionary>
</Application.Resources>
```

**⚠️ CRITICAL: Apply Explicit Sizing**

Ensure explicit sizing is applied to controls, as .NET MAUI enforces device-independent units exactly as specified:

```xml
<!-- ❌ WRONG - Relying on implicit sizing -->
<Label Text="Long text content"/>

<!-- ✅ CORRECT - Explicit sizing for proper layout -->
<Label Text="Long text content" WidthRequest="200" HeightRequest="50"/>
```

**⚠️ CRITICAL: Review ScrollView Usage**

Check ScrollView usage in infinite layouts like `VerticalStackLayout` and constrain their size:

```xml
<!-- ❌ WRONG - ScrollView in infinite layout -->
<VerticalStackLayout>
    <ScrollView>
        <StackLayout>
            <!-- Content -->
        </StackLayout>
    </ScrollView>
</VerticalStackLayout>

<!-- ✅ CORRECT - Constrain ScrollView size -->
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <ScrollView Grid.Row="0">
        <StackLayout>
            <!-- Content -->
        </StackLayout>
    </ScrollView>
</Grid>
```

**⚠️ CRITICAL: Frame to Border Migration**

Check ALL Frame usages and update to Border:

```xml
<!-- ❌ WRONG - Frame is deprecated -->
<Frame BorderColor="Gray" CornerRadius="10" Padding="20">
    <Label Text="Content"/>
</Frame>

<!-- ✅ CORRECT - Use Border -->
<Border Stroke="Gray" StrokeShape="RoundRectangle 10" Padding="20">
    <Label Text="Content"/>
</Border>
```

### 6.3 Update Page Base Classes (if needed)
Most page types remain the same, but some have been renamed:

**Navigation Changes:**
```csharp
// Before (Xamarin.Forms)
await Navigation.PushAsync(new DetailPage());

// After (.NET MAUI) - Same syntax
await Navigation.PushAsync(new DetailPage());
```

### 6.4 Update Resource References

**Before:**
```xml
<Image Source="icon.png" />
```

**After:**
```xml
<Image Source="icon.png" />
<!-- Resources now go in Resources/Images/ folder -->
```

## Step 7: Migrate Platform-Specific Code

Tip: Place platform-specific code in the correct `Platforms` subfolder. Remove obsolete platform code and update to .NET MAUI handlers if needed. See the `Reference` folder for platform code organization.

### 7.1 Copy Platform Code to Platforms Folders

**From Xamarin.Forms Android project** → `Platforms/Android/`
**From Xamarin.Forms iOS project** → `Platforms/iOS/`

### 7.2 Update Platform-Specific Files

**Platforms/Android/MainActivity.cs:**
```csharp
using Android.App;
using Android.Content.PM;

namespace YourAppName.Platforms.Android;

[Activity(Theme = "@style/Maui.SplashTheme", MainLauncher = true, 
          ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation | 
                               ConfigChanges.UiMode | ConfigChanges.ScreenLayout | 
                               ConfigChanges.SmallestScreenSize | ConfigChanges.Density)]
public class MainActivity : MauiAppCompatActivity
{
    protected override void OnCreate(Bundle savedInstanceState)
    {
        base.OnCreate(savedInstanceState);
        
        // Add any custom initialization here
    }
}
```

**Platforms/iOS/AppDelegate.cs:**
```csharp
using Foundation;

namespace YourAppName.Platforms.iOS;

[Register("AppDelegate")]
public class AppDelegate : MauiUIApplicationDelegate
{
    protected override MauiApp CreateMauiApp() => MauiProgram.CreateMauiApp();
    
    public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
    {
        // Add any custom initialization here
        return base.FinishedLaunching(application, launchOptions);
    }
}
```

**Platforms/Windows/App.xaml:**

The namespace should be updated to match the solution name. 


### 7.3 Required Platform Configuration Files

**⚠️ CRITICAL: Always include these platform-specific configuration files**

Many migration issues occur because these essential platform files are missing:

**Android Platform Files:**
- `Platforms/Android/AndroidManifest.xml`
- `Platforms/Android/Resources/values/colors.xml`

**iOS Platform Files:**
- `Platforms/iOS/Info.plist`
- `Platforms/iOS/Resources/PrivacyInfo.xcprivacy` (required for App Store)

**MacCatalyst Platform Files:**
- `Platforms/MacCatalyst/Entitlements.plist`
- `Platforms/MacCatalyst/Info.plist`

**Windows Platform Files:**
- `Platforms/Windows/Package.appxmanifest`
- `Platforms/Windows/app.manifest`

**Example AndroidManifest.xml:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application android:allowBackup="true" android:theme="@style/Maui.SplashTheme">
    </application>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

**Example iOS Info.plist:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>LSRequiresIPhoneOS</key>
    <true/>
    <key>UIDeviceFamily</key>
    <array>
        <integer>1</integer>
        <integer>2</integer>
    </array>
    <key>UIRequiredDeviceCapabilities</key>
    <array>
        <string>arm64</string>
    </array>
    <key>UISupportedInterfaceOrientations</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationLandscapeLeft</string>
        <string>UIInterfaceOrientationLandscapeRight</string>
    </array>
    <key>UISupportedInterfaceOrientations~ipad</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationPortraitUpsideDown</string>
        <string>UIInterfaceOrientationLandscapeLeft</string>
        <string>UIInterfaceOrientationLandscapeRight</string>
    </array>
    <key>XSAppIconAssets</key>
    <string>Assets.xcassets/appicon.appiconset</string>
</dict>
</plist>
```

Copy these files from a working .NET MAUI project in the `Reference` folder or create them based on the examples above.

## Step 8: Migrate Custom Renderers to Handlers

Tip: Prefer handlers over custom renderers. Register handlers in `MauiProgram.cs`. Review handler migration examples in the `Reference` folder.

### 8.1 Example: Custom Entry Renderer → Handler

**Before (Custom Renderer):**
```csharp
// Xamarin.Forms
public class CustomEntryRenderer : EntryRenderer
{
    protected override void OnElementChanged(ElementChangedEventArgs<Entry> e)
    {
        base.OnElementChanged(e);
        // Custom logic
    }
}
```

**After (Handler):**
```csharp
// .NET MAUI Handler
public class CustomEntryHandler : Microsoft.Maui.Handlers.EntryHandler
{
    protected override void ConnectHandler(Microsoft.UI.Xaml.Controls.TextBox platformView)
    {
        base.ConnectHandler(platformView);
        // Custom logic for Windows
    }

#if ANDROID
    protected override void ConnectHandler(AndroidX.AppCompat.Widget.AppCompatEditText platformView)
    {
        base.ConnectHandler(platformView);
        // Custom logic for Android
    }
#endif

#if IOS
    protected override void ConnectHandler(UIKit.UITextField platformView)
    {
        base.ConnectHandler(platformView);
        // Custom logic for iOS
    }
#endif
}
```

### 8.2 Register Handlers in MauiProgram.cs

```csharp
public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
            .ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
            })
            .ConfigureMauiHandlers(handlers =>
            {
#if ANDROID
                handlers.AddHandler<Entry, CustomEntryHandler>();
#elif IOS
                handlers.AddHandler<Entry, CustomEntryHandler>();
#elif WINDOWS
                handlers.AddHandler<Entry, CustomEntryHandler>();
#endif
            });

        return builder.Build();
    }
}
```

## Step 9: Update Dependencies and NuGet Packages

Tip: Always use the latest compatible package versions. Remove all Xamarin.* packages. Check the `Reference` folder for up-to-date package lists and compatibility notes.

### 9.1 Common Package Migrations

| Xamarin.Forms Package | .NET MAUI Package |
|----------------------|-------------------|
| `Xamarin.CommunityToolkit` | `CommunityToolkit.Maui` |
| `SkiaSharp.Views.Forms` | `SkiaSharp.Views.Maui.Controls` |
| `Xamarin.Forms.Maps` | `Microsoft.Maui.Controls.Maps` |
| `Lottie.Forms` | `SkiaSharp.Extended.UI.Maui` |

### 9.2 Update Package References

```xml
<ItemGroup>
  <!-- Core MAUI packages -->
  <PackageReference Include="Microsoft.Maui.Controls" Version="9.0.0" />
  <PackageReference Include="Microsoft.Maui.Controls.Compatibility" Version="9.0.0" />
  
  <!-- Community packages -->
  <PackageReference Include="CommunityToolkit.Maui" Version="9.0.0" />
  <PackageReference Include="CommunityToolkit.Mvvm" Version="8.2.2" />
  
  <!-- Third-party packages (verify .NET 9 compatibility) -->
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
</ItemGroup>
```

## Step 10: Handle Breaking Changes and API Updates

Tip: Review all code for breaking changes and update to new APIs. Use the `Reference` folder for examples of updated code and common migration patterns.

### 10.1 Critical Layout Changes

**⚠️ CRITICAL: Grid Layout Changes**

In .NET MAUI, `Grid` layouts require explicit row and column definitions. Xamarin.Forms automatically added missing rows and columns, but .NET MAUI does not:

```csharp
// ❌ WRONG - This will not work in .NET MAUI
<Grid>
    <Label Text="Hello"/>
    <Label Grid.Row="1" Text="World"/>
</Grid>

// ✅ CORRECT - Must explicitly define rows
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <Label Text="Hello"/>
    <Label Grid.Row="1" Text="World"/>
</Grid>
```

**⚠️ CRITICAL: AndExpand Layout Options**

All `*AndExpand` layout options are either deprecated or have no effect in .NET MAUI:

```csharp
// ❌ WRONG - AndExpand options are deprecated/ineffective
VerticalOptions = LayoutOptions.StartAndExpand
HorizontalOptions = LayoutOptions.FillAndExpand

// ✅ CORRECT - Use regular options
VerticalOptions = LayoutOptions.Start
HorizontalOptions = LayoutOptions.Fill
```

**⚠️ CRITICAL: Frame to Border Migration**

`Frame` is deprecated in .NET MAUI 9 and will be removed in future versions. Use `Border` instead:

```xml
<!-- ❌ WRONG - Frame is deprecated -->
<Frame BorderColor="DarkGray" CornerRadius="5" Margin="20">
    <Label Text="Content"/>
</Frame>

<!-- ✅ CORRECT - Use Border -->
<Border Stroke="DarkGray" StrokeShape="RoundRectangle 5" Margin="20" Padding="20">
    <Label Text="Content"/>
</Border>
```

**⚠️ CRITICAL: RelativeLayout Migration**

`RelativeLayout` only exists in the compatibility namespace. Use `Grid` instead:

```csharp
// ❌ WRONG - RelativeLayout is deprecated
<RelativeLayout>
    <Label Text="Hello"/>
</RelativeLayout>

// ✅ CORRECT - Use Grid or add compatibility namespace
<Grid>
    <Label Text="Hello"/>
</Grid>

// OR add compatibility namespace (temporary solution)
xmlns:compat="clr-namespace:Microsoft.Maui.Controls.Compatibility;assembly=Microsoft.Maui.Controls"
<compat:RelativeLayout>
    <Label Text="Hello"/>
</compat:RelativeLayout>
```

### 10.2 Deprecated Device APIs

**⚠️ CRITICAL: Device Class is Deprecated**

The entire `Device` class is deprecated. Use specific replacements:

```csharp
// ❌ WRONG - Device class is deprecated
Device.RuntimePlatform
Device.Idiom
Device.GetNamedSize(NamedSize.Medium, typeof(Label))

// ✅ CORRECT - Use specific APIs
DeviceInfo.Platform
DeviceInfo.Idiom
16 // Use explicit numeric values for font sizes
```

**Device Platform Detection:**
```csharp
// ❌ WRONG - Deprecated constants
Device.Android
Device.iOS
Device.macOS  // No longer exists

// ✅ CORRECT - Use DevicePlatform
DevicePlatform.Android
DevicePlatform.iOS
DevicePlatform.MacCatalyst  // Note: macOS becomes MacCatalyst
```

### 10.3 Color System Changes

**Color values changed from `double` to `float`:**
```csharp
// ❌ WRONG - Xamarin.Forms used double
Color.FromRgb(1.0, 0.5, 0.2)

// ✅ CORRECT - .NET MAUI uses float
Color.FromRgb(1.0f, 0.5f, 0.2f)

// ❌ WRONG - Old color names
Color.Red

// ✅ CORRECT - New color names
Colors.Red
```

### 10.4 Layout Behavior Changes

**⚠️ CRITICAL: Children Collection Changes**

Do not manipulate the `Children` collection directly in .NET MAUI:

```csharp
// ❌ WRONG - Don't manipulate Children directly
Grid grid = new Grid();
grid.Children.Add(new Label { Text = "Hello" });

// ✅ CORRECT - Add directly to layout
Grid grid = new Grid();
grid.Add(new Label { Text = "Hello" });
```

**⚠️ CRITICAL: Default Spacing Values**

.NET MAUI changed default spacing values to 0. Add implicit styles to preserve Xamarin.Forms defaults:

```xml
<!-- Add these styles to preserve Xamarin.Forms defaults -->
<Style TargetType="Grid">
    <Setter Property="ColumnSpacing" Value="6"/>
    <Setter Property="RowSpacing" Value="6"/>
</Style>
<Style TargetType="StackLayout">
    <Setter Property="Spacing" Value="6"/>
</Style>
```

### 10.5 Application Properties Migration

**⚠️ CRITICAL: Application Properties Removed**

`Application.Properties` and `Application.SavePropertiesAsync()` have been removed:

```csharp
// ❌ WRONG - Removed in .NET MAUI
Application.Current.Properties["key"] = value;
await Application.Current.SavePropertiesAsync();

// ✅ CORRECT - Use Preferences instead
await Preferences.SetAsync("key", value);
var value = await Preferences.GetAsync("key", defaultValue);
```

### 10.6 Removed and Deprecated Controls

**⚠️ CRITICAL: Removed Controls**

The following controls have been removed:
- `OpenGLView` - Completely removed
- `PhoneDialer.Current` - Use `PhoneDialer.Default` instead

**⚠️ CRITICAL: Deprecated Controls**

The following controls are deprecated and will be removed:
- `Frame` - Use `Border` instead
- `ClickGestureRecognizer` - Deprecated
- `MainPage` property - Will be removed in future versions

### 10.7 Deprecated Automation Properties

**⚠️ CRITICAL: Automation Properties Deprecated**

The following automation properties are deprecated:
- `AutomationProperties.Name` - Deprecated
- `AutomationProperties.HelpText` - Deprecated  
- `AutomationProperties.LabeledBy` - Deprecated

### 10.8 Deprecated Measure Methods

**⚠️ CRITICAL: Legacy Measure Methods Deprecated**

The following measure methods are deprecated:
- `VisualElement.OnMeasure` - Deprecated
- `VisualElement.Measure(Double, Double, MeasureFlags)` - Deprecated
- `SizeRequest` struct - Use `Size` instead

```csharp
// ❌ WRONG - Deprecated measure method
var sizeRequest = element.Measure(width, height, MeasureFlags.IncludeMargins);

// ✅ CORRECT - Use new measure method
var size = element.Measure(width, height);
```

### 10.9 StackLayout Behavior Changes

**⚠️ CRITICAL: StackLayout Behavior**

.NET MAUI stack layouts behave differently:
- `HorizontalStackLayout` and `VerticalStackLayout` ignore `*AndExpand` options
- Traditional `StackLayout` honors `*AndExpand` but marks them as obsolete
- Stack layouts will continue beyond available space instead of subdividing

```csharp
// ❌ WRONG - AndExpand has no effect in HorizontalStackLayout/VerticalStackLayout
<VerticalStackLayout>
    <Label VerticalOptions="FillAndExpand" Text="Won't expand"/>
</VerticalStackLayout>

// ✅ CORRECT - Use Grid for space subdivision
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <Label Text="Will expand"/>
</Grid>
```

### 10.10 Focus Management Changes

**⚠️ CRITICAL: Focus Methods Deprecated**

```csharp
// ❌ WRONG - Deprecated focus event
element.FocusChangeRequested += OnFocusChangeRequested;

// ✅ CORRECT - Use Focus() method
element.Focus();
```

### 10.11 Navigation Changes

**Shell Navigation (Recommended):**
```csharp
// Register routes in AppShell.xaml.cs
Routing.RegisterRoute("details", typeof(DetailPage));

// Navigate
await Shell.Current.GoToAsync("details");
```

### 10.12 Complete Migration Checklist

**⚠️ CRITICAL: Always verify these changes during migration:**

1. **Grid Layouts**: Add explicit `RowDefinitions` and `ColumnDefinitions`
2. **AndExpand Usage**: Remove or refactor all `*AndExpand` layout options
3. **Frame Controls**: Replace all `Frame` with `Border`
4. **RelativeLayout**: Replace with `Grid` or add compatibility namespace
5. **Device APIs**: Replace all `Device.*` calls with appropriate alternatives
6. **Color Values**: Update `double` to `float` values
7. **Application Properties**: Migrate to `Preferences` API
8. **Children Collection**: Use direct `Add()` methods instead of `Children.Add()`
9. **Default Spacing**: Add implicit styles to preserve Xamarin.Forms defaults
10. **Deprecated Controls**: Remove or replace deprecated controls
11. **Font Sizes**: Replace `Device.GetNamedSize()` with explicit values
12. **Measure Methods**: Update to non-deprecated measure methods
13. **Focus Management**: Update focus handling code
14. **Stack Layouts**: Review and update stack layout behavior expectations

This comprehensive list covers all known breaking changes and deprecated APIs between Xamarin.Forms and .NET MAUI. Always test thoroughly after applying these changes.

## Step 11: Resources and Assets Migration

**CRITICAL: Complete Resource Structure Migration**

This step is essential and often missed. ALL resources must be moved to the new MAUI structure and properly configured.

### 11.1 Move Resources to New Structure

For defaults, copy the Resources folder from a Reference folder inside the repo. Other resources you need to move from the Xamarin.Forms project.

**Create the complete Resources folder structure:**

```
Resources/
├── AppIcon/
│   ├── appicon.svg              # Default MAUI app icon
│   └── appiconfg.svg           # Default MAUI app icon foreground
├── Fonts/
│   ├── OpenSans-Regular.ttf    # Default MAUI font
│   └── OpenSans-Semibold.ttf   # Default MAUI font
├── Images/
│   ├── dotnet_bot.png          # Default MAUI image
│   └── [all your original images from Xamarin.Forms project]
├── Raw/
│   └── aboutassets.txt
├── Splash/
│   └── splash.svg
└── Styles/
    ├── Colors.xaml
    └── Styles.xaml
```

**⚠️ CRITICAL: Image File Naming Requirements**

.NET MAUI has strict requirements for image file names:
- File names must be **lowercase**
- Must start and end with a **letter character**
- Can contain only **alphanumeric characters** or **underscores**
- No spaces, hyphens, or other special characters allowed

**❌ WRONG - These will cause build errors:**
```
Icon-Small.png      # Contains hyphen
01All.png          # Starts with number
MyImage.PNG        # Contains uppercase
my image.jpg       # Contains space
```

**✅ CORRECT - Proper image naming:**
```
icon_small.png     # Lowercase with underscore
all_platforms.png  # Descriptive lowercase name
my_image.jpg       # Lowercase with underscore
dotnet_bot.png     # Standard MAUI naming
```

**Batch rename command for fixing image names:**
```bash
# Navigate to your Resources/Images folder
cd Resources/Images

# Rename all files to lowercase and replace invalid characters
for file in *; do
    if [ -f "$file" ]; then
        # Convert to lowercase and replace invalid characters
        newname=$(echo "$file" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9._]/_/g')
        if [ "$file" != "$newname" ]; then
            mv "$file" "$newname"
            echo "Renamed: $file -> $newname"
        fi
    fi
done
```

### 11.2 Update Resource References in Code

```csharp
// Before
ImageSource.FromResource("YourApp.Images.icon.png")

// After
ImageSource.FromFile("icon.png")
```

### 11.3 Handle Embedded Resources

**⚠️ CRITICAL: Embedded Resources Migration**

If your Xamarin.Forms app uses embedded resources, you need to update the approach:

**Before (Xamarin.Forms):**
```csharp
// In .csproj
<EmbeddedResource Include="Images\beach.jpg" />

// In code
ImageSource.FromResource("YourApp.Images.beach.jpg", typeof(App).Assembly)
```

**After (.NET MAUI):**
```csharp
// In .csproj - Mark as embedded resource
<EmbeddedResource Include="Resources\Images\beach.jpg" />

// Update EmbeddedImageResourceExtension to use correct namespace
[ContentProperty("Source")]
public class EmbeddedImageResourceExtension : IMarkupExtension
{
    public string Source { get; set; }

    public object ProvideValue(IServiceProvider serviceProvider)
    {
        if (Source == null)
            return null;

        var imageSource = ImageSource.FromResource(Source, typeof(EmbeddedImageResourceExtension).GetTypeInfo().Assembly);
        return imageSource;
    }
}
```

**Example usage in XAML:**
```xml
<Image Source="{local:EmbeddedImageResource YourAppName.Resources.Images.beach.jpg}" />
```

Make sure to update the namespace in the embedded resource path to match your project's namespace structure.

## Step 12: Create README.md Documentation

Tip: Write a clear, concise README.md using the provided template. Include a screenshot and describe the app's features and learning objectives. Review README.md files in the `Reference` folder for examples.

### 12.1 README.md Template

Create a `README.md` file in your project root using this template. **Use the ORIGINAL PROJECT NAME**, not the folder name:

```markdown
---
name: .NET MAUI - [OriginalProjectName]
description: [Brief description of what your app does, keep it concise and descriptive...]
page_type: sample
languages:
- csharp
- xaml
products:
- dotnet-maui
urlFragment: [category-originalprojectname]
---

# [OriginalProjectName]

[Detailed description of your application. Explain what it does, its main purpose, and any key features or technologies it demonstrates.]

[Add any additional paragraphs that explain the technical aspects, learning objectives, or notable implementation details.]

![App screenshot](Screenshots/[your-app-screenshot].png "[OriginalProjectName] app screenshot")
```

### 12.2 Template Guidelines

Follow these guidelines when filling out the template:

**Front Matter (YAML):**
- **name**: Always start with ".NET MAUI - " followed by your **ORIGINAL PROJECT NAME**
- **description**: Keep under 160 characters, describe functionality concisely
- **page_type**: Always use "sample"
- **languages**: Include "csharp" and "xaml" (add others if applicable)
- **products**: Always include "dotnet-maui"
- **urlFragment**: Use format "category-originalprojectname" in lowercase with hyphens

**Main Content:**
- **Header**: Use your **ORIGINAL PROJECT NAME** as H1
- **Description**: 1-3 paragraphs explaining the app's purpose and features
- **Screenshot**: Include at least one screenshot with descriptive alt text

### 12.3 Example urlFragment Formats

- `[foldername]-[appname]`

### 12.4 Complete Example

Here's how your README.md might look for a todo app:

```markdown
---
name: .NET MAUI - TodoApp
description: A cross-platform todo application demonstrating local data storage, MVVM pattern, and Shell navigation in .NET MAUI.
page_type: sample
languages:
- csharp
- xaml
products:
- dotnet-maui
urlFragment: data-todoapp
---

# TodoApp

This application demonstrates building a complete todo list app using .NET MAUI with local SQLite storage. The app showcases the MVVM pattern, data binding, Shell navigation, and cross-platform data persistence.

Key features include adding, editing, and deleting todo items, marking items as complete, and filtering by completion status. The app uses SQLite-net for local data storage and demonstrates best practices for .NET MAUI development.

![TodoApp screenshot](Screenshots/todo-main.png "TodoApp main screen showing todo list")
```

## Step 13: Create .NET 10 Version

Tip: After migrating to .NET 9, duplicate the solution for .NET 10. Update all target frameworks and package versions. Check for new breaking changes and deprecated APIs. Use the `Reference` folder for .NET 10 migration examples.

### 13.1 Duplicate Solution for .NET 10

After completing the .NET 9 migration, create a .NET 10 version:

9.0/[foldername]/YourApp -> 10.0/[foldername]/YourApp

### 13.2 Update Project File for .NET 10

Edit the `.csproj` file to target .NET 10 frameworks:

**Before (.NET 9):**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net9.0-android;net9.0-ios;net9.0-maccatalyst</TargetFrameworks>
    <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net9.0-windows10.0.19041.0</TargetFrameworks>
```

**After (.NET 10):**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net10.0-android;net10.0-ios;net10.0-maccatalyst</TargetFrameworks>
    <TargetFrameworks Condition="$([MSBuild]::IsOSPlatform('windows'))">$(TargetFrameworks);net10.0-windows10.0.19041.0</TargetFrameworks>
```

**Batch command to update all .NET 9 to .NET 10:**
```bash
# Navigate to your 10.0 project directory
cd 10.0/[your-project-path]

# Update all .csproj files to target .NET 10
find . -name "*.csproj" -exec sed -i 's/net9\.0/net10.0/g' {} \;

# Verify changes
grep -r "net10.0" . --include="*.csproj"
```

### 13.3 Update Package References

Update all package references to .NET 10 compatible versions.

### 13.4 Check for deprecated and obsolete classes and methods

**⚠️ CRITICAL: Comprehensive API Migration Requirements**

When migrating to .NET 9 and .NET 10 MAUI, the following APIs and namespaces have been deprecated, obsoleted, or completely removed. **All instances must be updated or removed to ensure compatibility.**

#### **Namespace Changes (Xamarin.Forms → .NET MAUI)**

| **Xamarin.Forms Namespace** | **.NET MAUI Namespace** | **Status** |
|---------------------------|------------------------|------------|
| `Xamarin.Forms` | `Microsoft.Maui.Controls` | **Required** |
| `Xamarin.Essentials` | `Microsoft.Maui.Essentials` | **Required** |
| `Xamarin.Forms.Maps` | `Microsoft.Maui.Controls.Maps` | **Required** |
| `Xamarin.Forms.PlatformConfiguration` | `Microsoft.Maui.Controls.PlatformConfiguration` | **Required** |

#### **Completely Removed APIs (.NET 8+)**

| **Removed API** | **Replacement** | **Action Required** |
|----------------|----------------|-------------------|
| `Application.Properties` | `Microsoft.Maui.Storage.Preferences` | **Migrate data** |
| `Application.SavePropertiesAsync()` | `Preferences.SetAsync()` | **Update method calls** |
| `PhoneDialer.Current` | `PhoneDialer.Default` | **Update property** |
| `OpenGLView` | No replacement | **Remove completely** |

```csharp
// ❌ WRONG - Removed in .NET 8+
Application.Current.Properties["key"] = value;
await Application.Current.SavePropertiesAsync();

// ✅ CORRECT - Use Preferences
await Preferences.SetAsync("key", value);
var value = await Preferences.GetAsync("key", defaultValue);
```

#### **Deprecated/Obsolete Controls (.NET 9)**

| **Deprecated Control** | **Replacement** | **Status** |
|----------------------|----------------|-----------|
| `Frame` | `Border` | **Obsolete in .NET 9, removed in future** |
| `RelativeLayout` | `Grid` or `Microsoft.Maui.Controls.Compatibility.RelativeLayout` | **Use compatibility namespace** |
| `ClickGestureRecognizer` | Standard gesture recognizers | **Deprecated in .NET 8** |

```xml
<!-- ❌ WRONG - Frame is obsolete -->
<Frame BorderColor="Gray" CornerRadius="10" Padding="20">
    <Label Text="Content"/>
</Frame>

<!-- ✅ CORRECT - Use Border -->
<Border Stroke="Gray" StrokeShape="RoundRectangle 10" Padding="20">
    <Label Text="Content"/>
</Border>
```

#### **Layout Options - Obsolete (.NET 9)**

**All `*AndExpand` layout options are deprecated:**

| **Deprecated Option** | **Replacement** | **Warning Message** |
|---------------------|----------------|-------------------|
| `LayoutOptions.FillAndExpand` | `LayoutOptions.Fill` | "The StackLayout expansion options are deprecated; please use a Grid instead." |
| `LayoutOptions.CenterAndExpand` | `LayoutOptions.Center` | "The StackLayout expansion options are deprecated; please use a Grid instead." |
| `LayoutOptions.StartAndExpand` | `LayoutOptions.Start` | "The StackLayout expansion options are deprecated; please use a Grid instead." |
| `LayoutOptions.EndAndExpand` | `LayoutOptions.End` | "The StackLayout expansion options are deprecated; please use a Grid instead." |

```csharp
// ❌ WRONG - AndExpand options are deprecated
VerticalOptions = LayoutOptions.FillAndExpand,
HorizontalOptions = LayoutOptions.CenterAndExpand

// ✅ CORRECT - Use regular options and Grid for expansion
VerticalOptions = LayoutOptions.Fill,
HorizontalOptions = LayoutOptions.Center

// ✅ BETTER - Use Grid for space subdivision
<Grid RowDefinitions="*, Auto">
    <Label Grid.Row="0" Text="Expanded content"/>
    <Button Grid.Row="1" Text="Fixed size"/>
</Grid>
```

#### **Device Class - Completely Deprecated (.NET 9)**

**The entire `Device` class is deprecated. All methods must be replaced:**

| **Deprecated Device API** | **.NET MAUI Replacement** |
|-------------------------|--------------------------|
| `Device.RuntimePlatform` | `DeviceInfo.Platform` |
| `Device.Idiom` | `DeviceInfo.Idiom` |
| `Device.GetNamedSize()` | Use explicit numeric values |
| `Device.GetNamedColor()` | No equivalent - use explicit colors |
| `Device.Invalidate()` | `VisualElement.InvalidateMeasure()` |
| `Device.InvokeOnMainThreadAsync()` | `MainThread.InvokeOnMainThreadAsync()` |
| `Device.OpenUri()` | `Launcher.OpenAsync()` |
| `Device.StartTimer()` | `DispatcherExtensions.StartTimer()` |
| `Device.SetFlags()` | No equivalent |
| `Device.SetFlowDirection()` | `Window.FlowDirection` |

```csharp
// ❌ WRONG - Device class is deprecated
if (Device.RuntimePlatform == Device.iOS)
{
    var size = Device.GetNamedSize(NamedSize.Large, typeof(Label));
    Device.InvokeOnMainThreadAsync(() => { /* code */ });
}

// ✅ CORRECT - Use specific APIs
if (DeviceInfo.Platform == DevicePlatform.iOS)
{
    var size = 18; // Use explicit numeric values
    await MainThread.InvokeOnMainThreadAsync(() => { /* code */ });
}
```

#### **Automation Properties - Deprecated (.NET 8+)**

| **Deprecated Property** | **Replacement** | **Status** |
|-----------------------|----------------|-----------|
| `AutomationProperties.Name` | `SemanticProperties.Description` | **Deprecated in .NET 8** |
| `AutomationProperties.HelpText` | `SemanticProperties.Hint` | **Deprecated in .NET 8** |
| `AutomationProperties.LabeledBy` | `SemanticProperties.Description` binding | **Deprecated in .NET 8** |

```xml
<!-- ❌ WRONG - Deprecated automation properties -->
<Button Text="Save" 
        AutomationProperties.Name="Save button"
        AutomationProperties.HelpText="Saves the current document"/>

<!-- ✅ CORRECT - Use semantic properties -->
<Button Text="Save" 
        SemanticProperties.Description="Save button"
        SemanticProperties.Hint="Saves the current document"/>
```

#### **Application Lifecycle - Deprecated (.NET 9)**

| **Deprecated Property** | **Replacement** | **Status** |
|-----------------------|----------------|-----------|
| `Application.MainPage` | `Window.Page` via `CreateWindow()` | **Obsolete in .NET 9, removed in future** |

```csharp
// ❌ WRONG - MainPage is deprecated
public App()
{
    InitializeComponent();
    MainPage = new AppShell();
}

// ✅ CORRECT - Use CreateWindow override
public App()
{
    InitializeComponent();
}

protected override Window CreateWindow(IActivationState? activationState)
{
    return new Window(new AppShell());
}
```

#### **Legacy Measure Methods - Deprecated (.NET 9)**

| **Deprecated Method** | **Replacement** |
|---------------------|----------------|
| `VisualElement.OnMeasure()` | Use new layout system |
| `VisualElement.Measure(Double, Double, MeasureFlags)` | `VisualElement.Measure(Double, Double)` |
| `SizeRequest` struct | `Size` struct |

```csharp
// ❌ WRONG - Legacy measure methods
var sizeRequest = element.Measure(width, height, MeasureFlags.IncludeMargins);
var size = sizeRequest.Request;

// ✅ CORRECT - Use new measure method
var size = element.Measure(width, height);
```

#### **XAML Markup Extensions - Deprecated (.NET 10)**

| **Deprecated Extension** | **Replacement** |
|------------------------|----------------|
| `FontImageExtension` | `FontImageSource` |

```xml
<!-- ❌ WRONG - FontImageExtension is deprecated (.NET 10) -->
<Button ImageSource="{FontImage Glyph=★, FontFamily=Icons}" />

<!-- ✅ CORRECT - Use FontImageSource -->
<Button>
    <Button.ImageSource>
        <FontImageSource Glyph="★" FontFamily="Icons" Size="18" />
    </Button.ImageSource>
</Button>
```

#### **Focus Management - Deprecated (.NET 8+)**

| **Deprecated API** | **Replacement** |
|------------------|----------------|
| `VisualElement.FocusChangeRequested` event | `Focus()` method |

```csharp
// ❌ WRONG - FocusChangeRequested is deprecated
element.FocusChangeRequested += OnFocusChangeRequested;

// ✅ CORRECT - Use Focus() method directly
element.Focus();
```

#### **Layout Collection Changes**

| **Deprecated Pattern** | **Replacement** |
|----------------------|----------------|
| `layout.Children.Add()` | `layout.Add()` |

```csharp
// ❌ WRONG - Don't manipulate Children collection directly
Grid grid = new Grid();
grid.Children.Add(new Label { Text = "Hello" });

// ✅ CORRECT - Add directly to layout
Grid grid = new Grid();
grid.Add(new Label { Text = "Hello" });
```

#### **MessagingCenter - Deprecated (.NET 9)**

**The entire `MessagingCenter` class is deprecated. Use `WeakReferenceMessenger` from CommunityToolkit.Mvvm instead:**

| **Deprecated MessagingCenter API** | **CommunityToolkit.Mvvm Replacement** |
|-----------------------------------|-------------------------------------|
| `MessagingCenter.Send<TSender, TArgs>()` | `WeakReferenceMessenger.Default.Send<TMessage>()` |
| `MessagingCenter.Subscribe<TSender, TArgs>()` | `WeakReferenceMessenger.Default.Register<TMessage>()` |
| `MessagingCenter.Unsubscribe<TSender, TArgs>()` | `WeakReferenceMessenger.Default.Unregister<TMessage>()` |

**First, install the NuGet package:**
```xml
<PackageReference Include="CommunityToolkit.Mvvm" />
```

**Usage Examples:**

```csharp
// ❌ WRONG - MessagingCenter is deprecated
using Xamarin.Forms;

// Sending a message
MessagingCenter.Send<MainPage, string>(this, "UpdateLabel", "Hello World");

// Subscribing to messages
MessagingCenter.Subscribe<MainPage, string>(this, "UpdateLabel", (sender, message) =>
{
    DisplayAlert("Message", message, "OK");
});

// Unsubscribing
MessagingCenter.Unsubscribe<MainPage, string>(this, "UpdateLabel");
```

```csharp
// ✅ CORRECT - Use WeakReferenceMessenger
using CommunityToolkit.Mvvm.Messaging;

// Define a message class
public class UpdateLabelMessage
{
    public string Text { get; set; }
    public UpdateLabelMessage(string text) => Text = text;
}

// Sending a message
WeakReferenceMessenger.Default.Send(new UpdateLabelMessage("Hello World"));

// Subscribing to messages (implement IRecipient<T>)
public partial class MainPage : ContentPage, IRecipient<UpdateLabelMessage>
{
    public MainPage()
    {
        InitializeComponent();
        WeakReferenceMessenger.Default.Register<UpdateLabelMessage>(this);
    }

    public void Receive(UpdateLabelMessage message)
    {
        DisplayAlert("Message", message.Text, "OK");
    }
}

// Or subscribe with a callback
WeakReferenceMessenger.Default.Register<UpdateLabelMessage>(this, (recipient, message) =>
{
    DisplayAlert("Message", message.Text, "OK");
});

// Unsubscribing (usually in cleanup)
WeakReferenceMessenger.Default.Unregister<UpdateLabelMessage>(this);
```

#### **Migration Verification Commands**

Use these commands to find deprecated APIs in your codebase:

```bash
# Find deprecated Device usage
grep -r "Device\." . --include="*.cs" --include="*.xaml"

# Find deprecated layout options
grep -r "AndExpand" . --include="*.cs" --include="*.xaml"

# Find deprecated Frame usage
grep -r "<Frame" . --include="*.xaml"

# Find deprecated automation properties
grep -r "AutomationProperties\." . --include="*.cs" --include="*.xaml"

# Find deprecated Application.MainPage usage
grep -r "MainPage\s*=" . --include="*.cs"
```

**Check the Microsoft Learn MCP for more obsolete or deprecated APIs**

DONE
