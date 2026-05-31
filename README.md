# 整合相機與地理定位之人員基本資料維護系統設計與實作
## Design and Implementation of a Personnel Information Management System with Camera and Geolocation Integration

* Title      : Design and Implementation of a Personnel Information Management System with Camera and Geolocation Integration
* Author     : Chien-Hung Lin
* Conference : Homework_2
* Date       : 2026-04-05

<p align="center">
  <img src="./Images/MainPage.png" width="300"/>
  <img src="./Images/DetailPage.png" width="300"/>
</p>

## 摘要 (Abstract)
建立一個「人員基本資料維護」畫面（延續 HW1 架構）。
當新增人員後，點選人員項目可進入明細頁，並具備返回功能。

*明細頁設計如下：

上方置中顯示人員姓名
提供「開啟相機」按鈕
下方顯示照片區（畫面占比需超過 40%）

*功能說明：
- [x] 1. 點擊開啟相機後，於照片區顯示相機畫面
- [x] 2. 提供拍攝按鈕，拍照後顯示照片於畫面中
- [x] 3. 照片下方顯示拍攝當下的地理位置（Geolocation）資訊
- [ ] 4. 最後需將應用程式部署至手機進行測試。

## 環境 (Requirements)
* .Net Core 10
* Android Emulator - Pixcel_7_api_35_0

## NuGet 安裝套件
* Microsoft.Maui.Controls (10.0.51)
* CommunityToolkit.Maui (using CommunityToolkit.Maui;)
* CommunityToolkit.Maui.Camera (6.0.0), Request Microsoft.Maui.Controls >= 10.0.30
  
## 操作流程：
* dotnet workload restore

## 注意事項
* 中文輸入法 - 系統設定變更語言
* Plateform - Android - Resources - AndroidManifest.xml - Camera 加入權限 (看是要用打勾，還是編輯的都可以)

* Plateform - Android - Resources - Access_Coarse_Location 地理權限
* Plateform - Android - Resources - Access_Fine_Location 地理權限
* Windows - Package.appxmanifest - Capabilities - Location 地理權限 (功能 - 位置)

## 生命週期管理 (Lifecycle)
1. 進入明細頁： DI 容器實例化新的 DetailPage 與 DetailViewModel，確保每次進入時拍照資訊都是乾淨的。
2. 返回主頁： 點擊返回鍵，明細頁實例被銷毀並釋放資源（如相機鏡頭掛載），回到 Singleton 狀態的 MainPage。

## 流程圖
<p align="center">
  <img src="./Images/Gemini.png" width="800"/>
</p>
<p align="center">
  <img src="./Images/flow2.png" width="800"/>
</p>

## ◎ ISSUE
- [ ] 1. 注入週期 => MauiProgram.cs (DI) 還有哪些,除了 AddSingleton、AddTransient 之外?
- [ ] 2. APP.xaml 在做什麼事情 => 可以做到那些事情 => 架構是什麼?
- [ ] 3. AppShell.xaml 在做什麼事情 => 可以做到那些事情 => 架構是什麼?

- [1. MauiProgram.cs (Dependency Injection)](#1-mauiprogramcs-dependency-injection)
- [2. App.xaml](#2-appxaml)
- [3. AppShell.xaml](#3-appshellxaml)
- [4. 整體架構流程](#4-整體架構流程)
- [5. 面試重點整理](#5-面試重點整理)

---

# 1. MauiProgram.cs (Dependency Injection)

`MauiProgram.cs` 是 MAUI 應用程式的啟動設定檔，主要負責：

- 建立 App
- 註冊 DI (Dependency Injection)
- 註冊 Fonts
- 註冊 Services
- 註冊 ViewModels
- 註冊 Pages
- 設定第三方套件

---

## DI 生命週期 (Service Lifetime)

### AddSingleton

整個 App 只建立一次物件。

```csharp
builder.Services.AddSingleton<IUserService, UserService>();
```

#### 特性

- App 啟動時建立
- 所有人共用同一份 Instance
- App 關閉才釋放

#### 適合

- API Service
- Config Service
- Login State
- Cache Service

---

### AddTransient

每次使用都建立新物件。

```csharp
builder.Services.AddTransient<LoginPage>();
builder.Services.AddTransient<LoginViewModel>();
```

#### 特性

- 每次 Resolve 都建立新 Instance
- 不保留狀態

#### 適合

- Page
- ViewModel
- 短生命週期 Service

---

### AddScoped

同一個 Scope 共用同一份 Instance。

```csharp
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
```

#### 特性

- 同一 Scope 共用
- 新 Scope 會建立新 Instance

#### 適合

- Blazor Hybrid
- Database Context

#### 注意

一般 MAUI 幾乎不使用 Scoped。

---

## 常見註冊方式

```csharp
public static MauiApp CreateMauiApp()
{
    var builder = MauiApp.CreateBuilder();

    builder.Services.AddSingleton<ApiService>();

    builder.Services.AddTransient<HomePage>();
    builder.Services.AddTransient<HomeViewModel>();

    return builder.Build();
}
```

---

# 2. App.xaml

## App.xaml 是什麼？

`App.xaml` 是整個 Application 的全域設定中心。

主要用途：

- Theme
- Color
- Style
- Resource
- Converter
- Global Font

---

## 架構

```text
Application
└── Resources
    ├── Color
    ├── Style
    ├── Converter
    ├── Font
    └── ResourceDictionary
```

---

## App.xaml 範例

```xml
<Application.Resources>

    <ResourceDictionary>

        <Color x:Key="PrimaryColor">
            #0078D4
        </Color>

        <Style TargetType="Button">
            <Setter Property="BackgroundColor"
                    Value="{StaticResource PrimaryColor}" />
        </Style>

    </ResourceDictionary>

</Application.Resources>
```

---

## 可以做什麼？

### 全域顏色

```xml
<Color x:Key="PrimaryColor">
    #0078D4
</Color>
```

---

### 全域樣式

```xml
<Style TargetType="Label">
    <Setter Property="FontSize"
            Value="18"/>
</Style>
```

---

### Dark / Light Theme

```xml
<AppThemeBinding
    Light="White"
    Dark="Black"/>
```

---

### Resource Dictionary

```xml
<ResourceDictionary Source="Styles/Colors.xaml"/>
```

---

# App.xaml.cs

## 功能

Application 啟動入口。

---

### 範例

```csharp
public partial class App : Application
{
    public App()
    {
        InitializeComponent();

        MainPage = new AppShell();
    }
}
```

---

## 啟動流程

```text
App 啟動
    ↓
載入 App.xaml
    ↓
載入 Resource
    ↓
建立 AppShell
    ↓
顯示首頁
```

---

# 3. AppShell.xaml

## AppShell 是什麼？

Shell 是 MAUI 的導航框架。

負責：

- 頁面導航
- TabBar
- Flyout Menu
- Route 管理
- Navigation

---

## 架構

```text
Shell
├── Flyout
├── TabBar
├── Tab
└── ShellContent
```

---

## AppShell.xaml 範例

```xml
<Shell
    x:Class="Demo.AppShell">

    <TabBar>

        <ShellContent
            Title="Home"
            ContentTemplate="{DataTemplate local:HomePage}" />

        <ShellContent
            Title="Setting"
            ContentTemplate="{DataTemplate local:SettingPage}" />

    </TabBar>

</Shell>
```

---

## 畫面結構

```text
TabBar
├── HomePage
└── SettingPage
```

---

## Flyout Menu

```xml
<FlyoutItem
    Title="會員中心">

    <ShellContent
        ContentTemplate="{DataTemplate local:MemberPage}" />

</FlyoutItem>
```

---

## Route 註冊

AppShell.xaml.cs

```csharp
public partial class AppShell : Shell
{
    public AppShell()
    {
        InitializeComponent();

        Routing.RegisterRoute(
            nameof(DetailPage),
            typeof(DetailPage));
    }
}
```

---

## 頁面導航

```csharp
await Shell.Current.GoToAsync(nameof(DetailPage));
```

---

## 帶參數導航

```csharp
await Shell.Current.GoToAsync(
    $"DetailPage?Id=1001");
```

---

## 接收參數

```csharp
[QueryProperty(nameof(Id), "Id")]
public partial class DetailViewModel
{
    public string Id { get; set; }
}
```

---

# 4. 整體架構流程

```text
MauiProgram.cs
│
├─ DI 註冊
├─ Service 註冊
├─ ViewModel 註冊
└─ Page 註冊
│
▼

App.xaml
│
├─ 全域 Style
├─ Theme
├─ Color
└─ Resource
│
▼

App.xaml.cs
│
└─ MainPage = AppShell
│
▼

AppShell.xaml
│
├─ Route
├─ Navigation
├─ Flyout
├─ TabBar
└─ ShellContent
│
▼

Page
│
├─ HomePage
├─ DetailPage
└─ SettingPage
```

---

# 5. 面試重點整理

## Q1：DI 有哪些生命週期？

| 類型 | 說明 |
|--------|--------|
| Singleton | App 共用同一個 Instance |
| Transient | 每次建立新 Instance |
| Scoped | 同一 Scope 共用 |

---

## Q2：App.xaml 是做什麼的？

管理：

- Theme
- Style
- Color
- Resource
- Converter

屬於：

```text
Application Layer
```

---

## Q3：AppShell.xaml 是做什麼的？

管理：

- Navigation
- Route
- TabBar
- Flyout Menu
- 頁面結構

屬於：

```text
Navigation Layer
```

---

# 一句話記憶法

```text
MauiProgram.cs
= 註冊服務(DI)

App.xaml
= 全域樣式與資源

App.xaml.cs
= App 啟動入口

AppShell.xaml
= 導航架構中心

Page
= 畫面

ViewModel
= 畫面邏輯

Service
= 商業邏輯
```

## 參考連結
1. https://learn.microsoft.com/zh-tw/dotnet/communitytoolkit/maui/get-started?tabs=CommunityToolkitMauiCamera
2. https://learn.microsoft.com/zh-tw/dotnet/communitytoolkit/maui/views/camera-view?tabs=android
3. https://youtu.be/PhBa263M8vg?si=lWmK2tiD9jix8GOl

## Tools
1. ScrollView - https://learn.microsoft.com/en-us/dotnet/maui/user-interface/controls/scrollview?view=net-maui-10.0

## Answer Code
1. https://youtu.be/ZmZb12hNTWk?si=BuxSt0LH8jCsf1Ud

## 引文(Citation)：
Please cite the following if you make use of the code.

>@inproceedings{kye2026meta,
  title={Design and Implementation of a Personnel Information Management System with Camera and Geolocation Integration},
  author={Chien-Hung Lin},
  year={2026-04-05}
}
