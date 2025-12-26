# .NET Overview - Tổng quan về .NET

## Mục lục

- [.NET là gì?](#net-là-gì)
- [Lịch sử phát triển](#lịch-sử-phát-triển)
- [Kiến trúc .NET](#kiến-trúc-net)
- [Các thành phần chính](#các-thành-phần-chính)
- [.NET Ecosystem](#net-ecosystem)
- [So sánh các phiên bản](#so-sánh-các-phiên-bản)
- [Cài đặt và Setup](#cài-đặt-và-setup)

---

## .NET là gì?

.NET là một **nền tảng phát triển phần mềm miễn phí, mã nguồn mở** của Microsoft, cho phép xây dựng nhiều loại ứng dụng khác nhau.

```
┌─────────────────────────────────────────────────────────────────┐
│                        .NET PLATFORM                            │
├─────────────────────────────────────────────────────────────────┤
│  Applications                                                   │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │   Web   │ │ Mobile  │ │ Desktop │ │  Cloud  │ │   IoT   │   │
│  │ ASP.NET │ │  MAUI   │ │   WPF   │ │  Azure  │ │         │   │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘   │
├─────────────────────────────────────────────────────────────────┤
│  Languages: C# | F# | VB.NET                                    │
├─────────────────────────────────────────────────────────────────┤
│  .NET Runtime (CLR) + Base Class Library (BCL)                  │
├─────────────────────────────────────────────────────────────────┤
│  Operating Systems: Windows | Linux | macOS                     │
└─────────────────────────────────────────────────────────────────┘
```

### Đặc điểm chính

| Đặc điểm         | Mô tả                                  |
| ---------------- | -------------------------------------- |
| Cross-platform   | Chạy trên Windows, Linux, macOS        |
| Open Source      | Mã nguồn mở trên GitHub                |
| High Performance | Hiệu năng cao, tối ưu cho cloud        |
| Modern           | Hỗ trợ các tính năng ngôn ngữ hiện đại |
| Secure           | Bảo mật tích hợp sẵn                   |
| Large Ecosystem  | Thư viện NuGet phong phú               |

---

## Lịch sử phát triển

```
Timeline:
─────────────────────────────────────────────────────────────────►

2002        2016         2019         2020         2022         2024
 │           │            │            │            │            │
 ▼           ▼            ▼            ▼            ▼            ▼
.NET        .NET         .NET         .NET 5       .NET 6       .NET 8
Framework   Core 1.0     Core 3.0     (Unified)    (LTS)        (LTS)
1.0
            Cross-       WPF,         Merge        Minimal      Performance
            Platform     WinForms     .NET Core    APIs         Improvements
                                      + Framework
```

### Các phiên bản quan trọng

| Phiên bản          | Năm  | Đặc điểm                        |
| ------------------ | ---- | ------------------------------- |
| .NET Framework 1.0 | 2002 | Phiên bản đầu tiên, chỉ Windows |
| .NET Core 1.0      | 2016 | Cross-platform, open source     |
| .NET Core 3.0      | 2019 | Hỗ trợ WPF, WinForms            |
| .NET 5             | 2020 | Hợp nhất .NET Core + Framework  |
| .NET 6             | 2021 | LTS, Minimal APIs, Hot Reload   |
| .NET 7             | 2022 | Performance, Native AOT         |
| .NET 8             | 2023 | LTS, Blazor United, Performance |
| .NET 9             | 2024 | Latest features                 |

> **LTS (Long Term Support)**: Được hỗ trợ 3 năm. Nên dùng cho production.

---

## Kiến trúc .NET

### Common Language Runtime (CLR)

CLR là **máy ảo** thực thi code .NET, tương tự JVM của Java.

```
┌─────────────────────────────────────────────────────────────┐
│                    Source Code                               │
│              (C#, F#, VB.NET)                                │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼ Compile
┌─────────────────────────────────────────────────────────────┐
│              Intermediate Language (IL)                      │
│                    + Metadata                                │
│                   (.dll / .exe)                              │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼ JIT Compile (Runtime)
┌─────────────────────────────────────────────────────────────┐
│                   Native Machine Code                        │
└─────────────────────────────────────────────────────────────┘
```

### CLR Services

```csharp
// CLR cung cấp các dịch vụ:

// 1. Memory Management (Garbage Collection)
var obj = new MyClass();  // CLR tự động quản lý bộ nhớ
// Không cần free/delete như C++

// 2. Type Safety
int number = "hello";  // Compile error - type safe

// 3. Exception Handling
try {
    // code
} catch (Exception ex) {
    // CLR quản lý exception
}

// 4. Thread Management
Task.Run(() => DoWork());  // CLR quản lý thread pool
```

### Base Class Library (BCL)

BCL là tập hợp các thư viện cơ bản:

```csharp
// System namespace - Core types
using System;
string text = "Hello";
int number = 42;
DateTime now = DateTime.Now;

// System.Collections - Collections
using System.Collections.Generic;
List<int> numbers = new List<int>();
Dictionary<string, int> dict = new Dictionary<string, int>();

// System.IO - File/Stream
using System.IO;
File.WriteAllText("file.txt", "content");

// System.Net.Http - HTTP Client
using System.Net.Http;
var client = new HttpClient();

// System.Linq - LINQ
using System.Linq;
var filtered = numbers.Where(x => x > 10).ToList();

// System.Threading.Tasks - Async
using System.Threading.Tasks;
await Task.Delay(1000);
```

---

## Các thành phần chính

### 1. Languages

```csharp
// C# - Ngôn ngữ chính, phổ biến nhất
public class HelloWorld
{
    public static void Main()
    {
        Console.WriteLine("Hello, World!");
    }
}
```

```fsharp
// F# - Functional programming
let hello = "Hello, World!"
printfn "%s" hello
```

```vb
' VB.NET - Visual Basic
Module HelloWorld
    Sub Main()
        Console.WriteLine("Hello, World!")
    End Sub
End Module
```

### 2. Application Models

| Model   | Mục đích                  | Framework           |
| ------- | ------------------------- | ------------------- |
| Web     | Web apps, APIs            | ASP.NET Core        |
| Mobile  | iOS, Android              | .NET MAUI           |
| Desktop | Windows apps              | WPF, WinForms, MAUI |
| Cloud   | Serverless, Microservices | Azure Functions     |
| Game    | Game development          | Unity (Mono)        |
| IoT     | Embedded devices          | .NET IoT            |
| AI/ML   | Machine Learning          | ML.NET              |

### 3. Tools

```bash
# .NET CLI - Command Line Interface
dotnet new console          # Tạo project mới
dotnet build               # Build project
dotnet run                 # Chạy project
dotnet test                # Chạy tests
dotnet publish             # Publish để deploy

# Package Management
dotnet add package Newtonsoft.Json
dotnet restore

# Project Templates
dotnet new webapi          # Web API
dotnet new mvc             # MVC Web App
dotnet new blazor          # Blazor App
dotnet new maui            # MAUI App
```

---

## .NET Ecosystem

### NuGet - Package Manager

```xml
<!-- .csproj file -->
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  <PackageReference Include="Serilog" Version="3.1.1" />
  <PackageReference Include="Dapper" Version="2.1.24" />
</ItemGroup>
```

### Popular NuGet Packages

| Package               | Mục đích           |
| --------------------- | ------------------ |
| Newtonsoft.Json       | JSON serialization |
| Serilog               | Logging            |
| AutoMapper            | Object mapping     |
| Dapper                | Micro ORM          |
| Entity Framework Core | Full ORM           |
| MediatR               | Mediator pattern   |
| FluentValidation      | Validation         |
| Polly                 | Resilience         |
| xUnit/NUnit           | Testing            |

### IDEs và Editors

| Tool               | Platform       | Mô tả              |
| ------------------ | -------------- | ------------------ |
| Visual Studio      | Windows, Mac   | Full-featured IDE  |
| VS Code            | Cross-platform | Lightweight editor |
| JetBrains Rider    | Cross-platform | Powerful IDE       |
| Visual Studio Code | Cross-platform | Free, extensions   |

---

## So sánh các phiên bản

### .NET Framework vs .NET Core vs .NET 5+

| Feature     | .NET Framework | .NET Core      | .NET 5+        |
| ----------- | -------------- | -------------- | -------------- |
| Platform    | Windows only   | Cross-platform | Cross-platform |
| Open Source | Không          | Có             | Có             |
| Performance | Tốt            | Rất tốt        | Xuất sắc       |
| Deployment  | Machine-wide   | Side-by-side   | Side-by-side   |
| Container   | Hạn chế        | Tốt            | Tốt            |
| Future      | Maintenance    | → .NET 5+      | Active         |

### Khi nào dùng gì?

```
Dự án mới → .NET 8 (LTS)

Dự án cũ .NET Framework:
├── Có thể migrate → .NET 8
└── Không thể migrate → Giữ .NET Framework 4.8

Dự án .NET Core → Upgrade lên .NET 8
```

---

## Cài đặt và Setup

### 1. Download SDK

```bash
# Windows - winget
winget install Microsoft.DotNet.SDK.8

# macOS - Homebrew
brew install dotnet-sdk

# Linux - apt
sudo apt-get install dotnet-sdk-8.0
```

### 2. Verify Installation

```bash
dotnet --version
# 8.0.100

dotnet --list-sdks
# 8.0.100 [C:\Program Files\dotnet\sdk]

dotnet --list-runtimes
# Microsoft.AspNetCore.App 8.0.0
# Microsoft.NETCore.App 8.0.0
```

### 3. Tạo Project đầu tiên

```bash
# Tạo console app
dotnet new console -n MyFirstApp
cd MyFirstApp

# Xem cấu trúc
├── MyFirstApp.csproj    # Project file
├── Program.cs           # Entry point
└── obj/                 # Build artifacts

# Chạy
dotnet run
# Hello, World!
```

### 4. Project File (.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

---

## Series .NET Knowledge

Đây là file đầu tiên trong series. Các file tiếp theo:

1. **01-DotNET-Overview.md** (file này)
2. **02-CSharp-Fundamentals.md** - Cơ bản C#
3. **03-CSharp-Advanced.md** - C# nâng cao
4. **04-ASPNET-Core-Basics.md** - ASP.NET Core cơ bản
5. **05-ASPNET-Core-WebAPI.md** - Web API
6. **06-Entity-Framework-Core.md** - EF Core ORM
7. **07-Dependency-Injection.md** - DI trong .NET
8. **08-Authentication-Authorization.md** - Auth
9. **09-Testing-DotNET.md** - Unit Testing
10. **10-Performance-BestPractices.md** - Performance
