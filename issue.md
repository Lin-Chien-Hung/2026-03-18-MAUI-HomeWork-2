## ◎ ISSUE
- [ ] 1. 注入週期 => MauiProgram.cs (DI) 還有哪些,除了 AddSingleton、AddTransient 之外?
- [ ] 2. APP.xaml 在做什麼事情 => 可以做到那些事情 => 架構是什麼?
- [ ] 3. AppShell.xaml 在做什麼事情 => 可以做到那些事情 => 架構是什麼?

- [1. MauiProgram.cs (Dependency Injection)](#1-mauiprogramcs-dependency-injection)
- [2. App.xaml](#2-appxaml)
- [3. AppShell.xaml](#3-appshellxaml)

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

# 4. 重點整理

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

# Memmory

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
