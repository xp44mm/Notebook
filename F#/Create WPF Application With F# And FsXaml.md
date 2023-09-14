# Create WPF Application With F# And FsXaml

In this article, I will show you how to start a WPF (Windows application) with F# by using a type of provider called FsXaml to work easily with XAML files.

F# is a multi-paradigm, functional-first programming language, the syntax is very elegant and lightweight. I decided to write some articles about F# and Windows development to show you how to use this language with WPF. There are many tools for facilitating F# integration with WPF and in this article, I will present one of them: FsXaml.

F# has no WPF project template in Visual Studio, however, by creating a Console application we can create a WPF application in few lines of code. In .NET, UI development is based on OOP paradigm, seems like unnatural with F#; but I will show you some tools for keep using benefits of F# in Windows development.

FsXaml enables us to use the type provider features for XAML. The type provider is made up of components that use data to provide types, properties, and methods for use in our program, this is a very powerful feature in F#.

## Create F# application

Once Visual Studio is launched, let’s create a new project. Navigate to Visual F# node and after that choose **Console Application (.NET Framework)**. Give a name, a location and click OK.

The project created is very lightweight and contains one source file (Program.fs).

## Set up the project for use WPF

In solutions explorer, right click on your project and go to **Properties**. Change **Output type** from Console application to **Windows application**.

Save all.

Then right-click on References section in the project from solutions explorer and click **Manage NuGet packages…**

In the browse tab, search for **FsXaml** and type Enter, then select FsXaml for WPF and click Install.

It will add WPF dependencies to our project (like `PresentationCore`, etc…). Now, configuration step is done, we can start!

## Create `App.xaml`

In a WPF project, `App.xaml` is associated with Application class and is the entry point of the program. What we are going to do is create the XAML file `App.xaml` and define the entry point with this file in our F# code.

Rename the file Program.fs into `App.fs`.

Let’s add a XAML file in the project: to achieve this, there is not an XAML template available, so we will simply add an XML file and name it with `.xaml` extension. Right-click in the project from solutions explorer and click Add > New item…

Select XML file and name it `App.xaml`,

Add the following code

```xaml
<Application 
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" 
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" 
  StartupUri="MainWindow.xaml"> 
 
  <Application.Resources> 
  </Application.Resources> 
 
</Application> 
```

This is exactly the same as a WPF C# project. Currently, I defined a `StartupUri` to a page called `MainWindow.xaml` that doesn’t exist yet. Before creating this page, we will configure the application entry point. It will be very simple.

Open `App.fs` source file and add the following code

```fsharp
module App 

open System 
open FsXaml 

type App = XAML<"App.xaml"> 

[<EntryPoint;STAThread>] 
let main argv =
   App().Run() 
```

As you can see, using FsXaml is as simple as this,

```fsharp
type App = XAML<"App.xaml">
```

We give as a parameter a string corresponding to the path to our XAML file (referenced from Visual Studio), for now, the second parameter (of type bool) is not important, it is used for exposing named properties.

The FsXaml type provider generated an F# type (here `App`) ready to use in our code. We use it in the entry point (main function) for calling the `Run()` method, for launch the application. We can currently work directly with the XAML file from this type like you will do with C# in the code behind.

This is extremely powerful: based on a simple XAML file, FsXaml generated us a type recognized as application class and we have access to all members of it:

## Add a window

Currently, we have our entry point and we need to add a window to display some UI elements. If you remember, I defined a `StartuUri` to `MainWindow.xaml` in the `App.xaml` file. Let’s create the main window in the same way we added `App.xaml` file (see above). Add this XAML code,

```xaml
<Window xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" 
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" 
  xmlns:local="clr-namespace:Views;assembly=FsXamlApp" 
  xmlns:fsxaml="http://github.com/fsprojects/FsXaml"  
  Title="F# Windows App" Height="300" Width="400"> 
 
  <Grid> 
    <TextBlock Text="Hello F# from WPF!" 
          HorizontalAlignment="Center" 
           VerticalAlignment="Center" />
   </Grid> 
</Window> 
```

Visual Studio supports the XAML designer for the view,

Next, we will add an F# source file corresponding to the code behind of this window, we will just simply load our XAML file with FsXaml. Add an F# source file and name it `MainWindow.xaml.fs`.

```fsharp
namespace FsXamlApp 
open FsXaml 
type MainWindow = XAML<"MainWindow.xaml"> 
```

At the moment, we are not going to do anything more. Just take care `MainWindow.xaml.fs` file is above `App.xaml` file in solutions explorer, in F# file order is significative and the Entry point must be the last file in the hierarchy (located the lowest),

You can move a file by selecting one of the solutions explorers and use Alt + Up or Down arrow shortcut.

## Launch the application

Press F5 or click on the green arrow Start button from Visual Studio to see our application,

## Summary

I hope you enjoyed this article and that it made you want to use F# by showing you one of its powerful features. I will continue to talk about WPF development with F # in my next articles. Thanks for reading!

参考文献：

- [Create WPF Application With F# And FsXaml](https://www.c-sharpcorner.com/article/create-wpf-application-with-f-sharp-and-fsxaml/)

## 太长不看

Console Application (.NET Framework)，名称`FsXamlApp`

输出类型：windows应用

NuGet: `FsXaml.Wpf`

项目文件(.fsproj)：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net481</TargetFramework> 
    <WarnOn>3390;$(WarnOn)</WarnOn>
  </PropertyGroup>

  <ItemGroup>
    <Resource Include="App.xaml" />
    <Resource Include="MainWindow.xaml" />
    <Compile Include="MainWindow.xaml.fs" />
    <Compile Include="App.fs" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="FsXaml.Wpf" Version="3.1.6" />
    <Reference Include="UIAutomationTypes" />
  </ItemGroup>
</Project>
```

App.xaml:

```xaml
<Application 
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    StartupUri="MainWindow.xaml">
</Application>
```

MainWindow.xaml

```xaml
<Window 
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d"
        Title="MainWindow" Height="300" Width="400">
    <Grid>
        <TextBlock Text="Hello F# from WPF!" 
          HorizontalAlignment="Center" 
           VerticalAlignment="Center" />
    </Grid>
</Window>
```

MainWindow.xaml.fs

```fsharp
namespace FsXamlApp

open FsXaml

type MainWindowBase = XAML<"MainWindow.xaml">

type MainWindow() =
    inherit MainWindowBase()
```

App.fs

```fsharp
module FsXamlApp.App

open System
open FsXaml

type App = XAML<"App.xaml">

[<STAThread;EntryPoint>]
let main argv = App().Run()
```



FSharp.ViewModule.Core?



