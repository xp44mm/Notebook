# 10 WPF资源

资源是保持有用对象在附近的一种方法。诸如：刷子，样式，或模板对象。因此，你能更容易地重用他们。

尽管能用代码创造和操纵资源，通常用XAML标记。一个资源被定义以后，你能使用它，在窗口中，标记的剩余部分(或，在一个应用资源的情况下，遍及你的应用的剩余部分)。这技术简化你的标记，节省重复的编码，并且允许你存储用户界面细节(诸如你的应用的颜色方案)在一个集中的地方因而他们能被容易地修改。对象资源也是重用WPF样式基础。

## 资源基础

你定义资源的位置可以在代码中、或在标记的各种地方（除了特殊控件、特殊窗口、或者整个应用程序）。

资源的好处包括：效率高，可维护性，适应性。

### 资源集合

每个元素包括一个`Resources`属性，存储一个资源的词典集合。是`ResourceDictionary`类的一个实例。资源集合能持有任何类型的对象，由字符串索引。

尽管每个元素包括`Resources`属性（定义在`FrameworkElement`类），最常在窗口元素上定义资源。那是因为每个元素首先访问自己的资源集合，然后是它父元素的资源集合。

例如，考虑带有三个按钮的窗口，第一个和最后一个按钮使用同样的刷子。为了重用，我们在资源中为窗口定义了图像刷子。

```xaml
<Window.Resources>
  <ImageBrush x:Key="TileBrush" TileMode="Tile"
    ViewportUnits="Absolute" Viewport="0 0 32 32"
    ImageSource="happyface.jpg" Opacity="0.3">
  </ImageBrush>
</Window.Resources>
```

考虑第一个特性`x:Key`，这为资源分配了一个名字，用于在资源集合中检索。名字可以使用任何字符串。

引用资源使用标记扩展，有两种：静态资源，动态资源。静态资源在窗口第一次创建时，被设置一次。动态资源每当资源改变时，被应用。这个例子中，图像刷子不会改变，所以使用静态资源。

```xaml
<Button Background="{StaticResource TileBrush}" 
  Margin="5" Padding="5" FontWeight="Bold" FontSize="14">
  A Tiled Button
</Button>
```

动态资源具有静态资源的功能，只是多一些开销。

```xaml
<Button Background="{DynamicResource TileBrush}">
```

### 资源的层次结构

每个元素都有自己的资源集合，WPF沿元素树向上执行一个递归的搜索，查找你希望的资源。在当前的例子中，你能移动图像刷子资源从窗口的资源集合到`StackPanel`的资源集合。应用程序的工作方式不会变化。你也能将图像刷子放在按钮的资源集合，但是每个按钮都要定义一次。

当使用一个静态的资源，必须在引用资源之前定义资源。

结果，如果你在按钮元素中放置资源，需要先定义资源，再设置背景。

```xaml
<Button Margin="5" Padding="5" FontWeight="Bold" FontSize="14">
  <Button.Resources>
    <ImageBrush x:Key="TileBrush" TileMode="Tile"
      ViewportUnits="Absolute" Viewport="0 0 10 10"
      ImageSource="happyface.jpg" Opacity="0.3"></ImageBrush>
  </Button.Resources>
  <Button.Background>
    <StaticResource ResourceKey="TileBrush"/>
  </Button.Background>
  <Button.Content>Another Tiled Button</Button.Content>
</Button>
```

这个例子中，使用嵌套元素引用资源。使用`ResourceKey`指向正确的资源。

可以在不同的资源集合中使用相同的资源名：

```xaml
<Window x:Class="Resources.TwoResources"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    Title="Resources" Height="300" Width="300" >

  <Window.Resources>
    <ImageBrush x:Key="TileBrush" TileMode="Tile"
                ViewportUnits="Absolute" Viewport="0 0 32 32"
                ImageSource="happyface.jpg" Opacity="0.3"></ImageBrush>
  </Window.Resources>

  <StackPanel Margin="5">
    <Button Background="{StaticResource TileBrush}" Padding="5"
      FontWeight="Bold" FontSize="14" Margin="5" >A Tiled Button</Button>

    <Button Padding="5" Margin="5"
      FontWeight="Bold" FontSize="14">A Normal Button</Button>
    <Button Background="{DynamicResource TileBrush}" Padding="5" Margin="5"
      FontWeight="Bold" FontSize="14">
      <Button.Resources>
        <ImageBrush x:Key="TileBrush" TileMode="Tile"
          ViewportUnits="Absolute" Viewport="0 0 32 32"
          ImageSource="sadface.jpg" Opacity="0.3"></ImageBrush>
      </Button.Resources>
      <Button.Content>Another Tiled Button</Button.Content>
    </Button>

  </StackPanel>
</Window>
```

在这种情况下，按钮使用它发现的第一个元素。因为他从自己开始搜索资源集合，第二个按钮使用`sadface.jpg`图像，而第一个按钮从容器窗口获得刷子，使用`happyface.jpg`图像。

### 静态资源和动态资源

静态资源一次性从资源集合抓取对象。

动态资源每当需要时查找资源集合中的对象。

仅当下列情况使用动态资源：

- 资源依赖于系统设置（如Windows当前颜色和字体）。
- 以编程方式替换资源对象（例如，实现某种动态皮肤特征）。

### 非共享资源

通常，当在多个地方使用一个资源时，使用的是相同的对象实例。也就是说，资源是共享的。

为关闭共享，使用`Shared`属性：

```xaml
<ImageBrush x:Key="TileBrush" x:Shared="False" ...></ImageBrush>
```

每次引用非共享元素，都会克隆一个新的资源实例。

### 在代码中访问资源

在代码中使用 `FrameworkElement.FindResource()`方法引用资源：

```c#
private void cmdChange_Click(object sender, RoutedEventArgs e)
{
    Button cmd = (Button)sender;
    ImageBrush brush = (ImageBrush)sender.FindResource("TileBrush");
    ...
}
```

另一个是`TryFindResource()`方法。

### 应用程序资源

应用程序范围的资源定义在App.xaml文件中，例如：

```xaml
<Application x:Class="Resources.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    StartupUri="Menu.xaml"
    >
    <Application.Resources>
      <ImageBrush x:Key="TileBrush" TileMode="Tile"
        ViewportUnits="Absolute" Viewport="0 0 32 32"
        ImageSource="happyface.jpg" Opacity="0.3">
      </ImageBrush>
    </Application.Resources>
</Application>
```

### 系统资源

`System.Windows`名字空间有三个类： `SystemColors`, `SystemFonts`, 和`SystemParameters`。

`SystemParameters`包裹一个巨大的设置列表。用于描述各种屏幕元素的标准尺寸、键盘和鼠标设置，和屏幕尺寸，和是否打开各种图形效果（如热跟踪，滴落阴影，和拖拽时显示窗口内容）。

这三个类暴露的接口都是静态属性。

以Color结尾的属性用法：

```c#
label.Foreground = new SolidBrush(SystemColors.WindowTextColor);
```

更有效率地，类中也预定义了刷子属性，这些属性以Brush结尾：

```c#
label.Foreground = SystemColors.WindowTextBrush;
```

使用属性扩展设置前景属性：

```xaml
<Label Foreground="{x:Static SystemColors.WindowTextBrush}">
```

这个例子没有使用动态资源，这得到的只是系统变量的一个快照，当变量改变时，用户界面不会随之改变。系统类还定义了一族以`Key`结尾的属性，用于引用动态资源。这是使用动态资源的例子：

```xaml
<Label Foreground="{DynamicResource {x:Static SystemColors.WindowTextBrushKey}}">
```

动态资源使用的键由`SystemColors.WindowTextBrushKey`属性定义。因为该属性是静态的，又需要包裹在静态标记扩展中。

## 资源词典

如果你希望多个工程之间共享资源，你可以创造资源词典。资源词典只是一种XAML文档,存储你希望使用的资源。

Creating a Resource Dictionary
Here’s an example of a resource dictionary that has one resource:

```xaml
<ResourceDictionary
 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <ImageBrush x:Key="TileBrush" TileMode="Tile"
    ViewportUnits="Absolute" Viewport="0 0 32 32"
    ImageSource="happyface.jpg" Opacity="0.3">
  </ImageBrush>
</ResourceDictionary>
```

Using a Resource Dictionary

```xaml
<Application x:Class="Resources.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
>
  <Application.Resources>
    <ResourceDictionary>
      <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="AppBrushes.xaml"/>
        <ResourceDictionary Source="WizardBrushes.xaml"/>
      </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
  </Application.Resources>
</Application>
```

合并的资源与本地资源：
```xaml
<Application.Resources>
  <ResourceDictionary>
    <ResourceDictionary.MergedDictionaries>
      <ResourceDictionary Source="AppBrushes.xaml"/>
      <ResourceDictionary Source="WizardBrushes.xaml"/>
    </ResourceDictionary.MergedDictionaries>
    <ImageBrush x:Key="GraphicalBrush1" ... ></ImageBrush>
    <ImageBrush x:Key="GraphicalBrush2" ... ></ImageBrush>
  </ResourceDictionary>
</Application.Resources>
```

Sharing Resources Between Assemblies

```c#
ResourceDictionary resourceDictionary = new ResourceDictionary();
resourceDictionary.Source = new Uri(
  "ResourceLibrary;component/ReusableDictionary.xaml", UriKind.Relative);
```

用法：

```C#
cmd.Background = (Brush)resourceDictionary["TileBrush"];
```

资源词典的定义：

```xaml
<ResourceDictionary
 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
 xmlns:local="clr-namespace:ResourceLibrary;assembly=ResourceLibrary">
  <ImageBrush
   x:Key="{ComponentResourceKey TypeInTargetAssembly={x:Type local:CustomResources},ResourceId=SadTileBrush}"
   TileMode="Tile" ViewportUnits="Absolute" Viewport="0 0 32 32"
   ImageSource="ResourceLibrary;component/sadface.jpg" Opacity="0.3">
  </ImageBrush>
</ResourceDictionary>
```

引用名字空间：

```xaml
<Window 
 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
 xmlns:res="clr-namespace:ResourceLibrary;assembly=ResourceLibrary"
 ... >
```

窗口中的按钮：

```xaml
<Button Background="{DynamicResource {ComponentResourceKey TypeInTargetAssembly={x:Type res:CustomResources},ResourceId=SadTileBrush}}"
 Padding="5" Margin="5" FontWeight="Bold" FontSize="14">
  A Resource From ResourceLibrary
</Button>
```

动态资源的内容可以定义在后台代码中，定义：

```C#
public class CustomResources
{
    public static ComponentResourceKey SadTileBrushKey
    {
        get
        {
            return new ComponentResourceKey(
              typeof(CustomResources), "SadTileBrush");
        }
    }
}
```

Now you use the Static markup extension to access this property and apply the resource without using the long-winded `ComponentResourceKey` in your markup:

```xaml
<Button
 Background="{DynamicResource {x:Static res:CustomResources.SadTileBrushKey}}"
 Padding="5" Margin="5" FontWeight="Bold" FontSize="14">
  A Resource From ResourceLibrary
</Button>
```

