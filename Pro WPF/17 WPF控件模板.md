# 17 WPF控件模板

## 理解逻辑树和可视树

逻辑树：你添加的元素的杂烩称为逻辑树。通过逻辑树实现的功能包括：属性值继承、事件路由、样式。

可视树是逻辑树的展开版。元素被分解为更小的单位。例如，一个按钮在逻辑树是最小单位，是一个黑盒子。但是在可视树中，按钮是一个组件。`ButtonChrome`、`ContentPresenter`、和`TextBlock`。

所有这些细节本身仍是元素，换句话说，它们派生自`FrameworkElement`类。

有一种以上的办法展开一个逻辑树到可视树。诸如你使用样式和你设置属性的细节能影响一个可视树的组成。例如，在前一个示例中，按钮持有文本内容，结果，它自动地创造一个嵌套`TextBlock`元素。但是如你所知，按钮控件是一个内容控件，所以它能持有任何其他的元素，只要你嵌套它到按钮内部。

可视树就是用于模板。

`System.Windows.LogicalTreeHelper`提供了几个方法浏览逻辑树：

| 名字              | 描述                                                       |
| ----------------- | ---------------------------------------------------------- |
| FindLogicalNode() | 依靠名字查找指定元素，开始于指定元素，沿逻辑树向下搜索。   |
| BringIntoView()   | 滚动容器（如果可能），使指定元素可见。                     |
| GetParent()       | 获得指定元素的父元素。                                     |
| GetChildren()     | 获得指定元素的子元素。这个方法不区分内容控件或者容器控件。 |

`VisualTreeHelper`提供了几个类似的方法：`GetChildrenCount()`，`GetChild()`，和`GetParent()`；还包括少量的用于低层次绘图的方法。

显示窗口可视树的代码：

```C#
public void ShowVisualTree(DependencyObject element)
{
    treeElements.Items.Clear();
    //启动处理元素的递归过程，开始于根
    ProcessElement(element, null);
}

private void ProcessElement(DependencyObject element, TreeViewItem previousItem)
{
    var item = new TreeViewItem();
    item.Header = element.GetType().Name;
    item.IsExpanded = true;

    //查看当前元素是否为空,为空是树根
    if (previousItem == null)
    {
        treeElements.Items.Add(item);
    }
    else
    {
        previousItem.Items.Add(item);
    }

    //查看这个元素是否包含子元素
    for (int i = 0; i < VisualTreeHelper.GetChildrenCount(element); i++)
    {
        //递归处理每个被包含的元素
        ProcessElement(VisualTreeHelper.GetChild(element, i), item);
    }
}
```

在窗口按钮点击事件添加如下代码，显示树：

```c#
VisualTreeDisplay treeDisplay = new VisualTreeDisplay();
treeDisplay.ShowVisualTree(this);
treeDisplay.Show();
```

 

## 理解模板

每个控件有一个内建食谱。决定了控件应该如何被呈现。（作为一组更基础的元素）那食谱被称作控件模板，并且它使用一块XAML标记定义。

WPF控件是无外观的，意味着控件的外观可以重新定义。不变的是控件的行为，它被固化在控件类中。

`ButtonChrome`

`ContentPresenter`：内容

`TargetName`

### 模板的类型

在WPF中存在三类模板，所有的派生自`FrameworkTemplate`基类。控件模板（`ControlTemplate`），数据模板（`DataTemplate`和`HierarchicalDataTemplate`）以及更特殊用于`ItemsControl`的面板模板（`ItemsPanelTemplate`）。

数据模板从一个对象抽取数据并且显示它在内容控件中或在单独的列表控件的项目中。数据模板在数据绑定场合极度有用的。在某种程度上，数据模板和控件模板重叠。例如，两个类型的模板允许你插入额外的元素，应用格式化，等等。但是，数据模板被用于在一个存在的控件内部添加元素。不改变控件的预建方面。另一方面，控件模板是更激烈的方法，允许你完全改写控件的内容模型。

这些控件可以组合一起使用。 

### Chrome类

它是一个底层类。详见473页。

### 解剖控件

本节介绍一个浏览标准控件模板的实用程序。本书源代码已经将此程序单独打包，名为`ControlTemplateBrowser`。

## 创造控件模板

### 简单的按钮

控件模板资源的基本轮廓：

```
<Window.Resources>
  <ControlTemplate x:Key="ButtonTemplate" TargetType="{x:Type Button}">
    ...
  </ControlTemplate>
</Window.Resources>
```

 

控件模板设置TargetType属性显式地指明它为按钮而设计。对于样式，这总是一个应该遵守的好约定。在内容控件中，诸如按钮，它也是一个必要条件，否则ContentPresenter将不工作。

为控件模板绘制轮廓：

```
<ControlTemplate x:Key="ButtonTemplate" TargetType="{x:Type Button}">
  <Border BorderBrush="Orange" BorderThickness="3" CornerRadius="2"
   Background="Red" TextBlock.Foreground="White">
    ...
  </Border>
</ControlTemplate>
```

所有内容控件都必须有***ContentPresenter\***元素—它是"在这里插入内容"的标记，告诉WPF填充内容的地方：

```
<ControlTemplate x:Key="ButtonTemplate" TargetType="{x:Type Button}">
  <Border BorderBrush="Orange" BorderThickness="3" CornerRadius="2"
   Background="Red" TextBlock.Foreground="White">
    <ContentPresenter RecognizesAccessKey="True"></ContentPresenter>
  </Border>
</ControlTemplate>
```

ContentPresenter设置RecognizesAccessKey属性为真。

如果一个控制从ContentControl派生，它的模板将包括一个ContentPresenter说明内容将被放置的地方。如果控制从ItemsControl派生，它的模板将包括一个ItemsPresenter指明包含列表项目的面板将被放置的地方。在少数情况下，控件可能使用一个派生版本—例如，ScrollViewer的控件模板使用ScrollContentPresenter，派生自ContentPresenter。

### 模板绑定

使用模板绑定，你的模板能提取控件的属性值。在这个例子中，你能使用模板绑定取回Padding属性的值，和使用它创造一个ContentPresenter元素的外边距：

```
<ControlTemplate x:Key="ButtonTemplate" TargetType="{x:Type Button}">
  <Border BorderBrush="Orange" BorderThickness="3" CornerRadius="2"
   Background="Red" TextBlock.Foreground="White">
    <ContentPresenterRecognizesAccessKey="True"
     Margin="{TemplateBinding Padding}"></ContentPresenter>
  </Border>
</ControlTemplate>
```

模板绑定类似于普通的数据绑定，但是他们更轻重量，因为他们是特别地为了控件模板设计。他们只支持单向数据绑定（换句话说，他们能传递信息从控件到模板但是倒过来不行），并且他们不能从Freezable派生类的属性提取信息。如果你遭遇模板绑定不工作，你能使用一个成熟的数据绑定代替。

你能预期需要什么模板绑定唯一办法是检查默认控件模板。如果你考虑Button类控件模板，你将发现与自定义模板相同的方式的一个模板绑定—它取按钮的内边距，并且转换它为ContentPresenter的外边距。标准按钮模板包含几个更多模板绑定，在简单的自定义模板不使用，诸如HorizontalAlignment，VerticalAlignment，和Background。那意味着如果你在按钮元素上设置这些属性，他们将并不影响简单的自定义模板。

### 改变属性的触发器

控件模板使用触发器响应属性改变。

```
<ControlTemplate x:Key="ButtonTemplate" TargetType="{x:Type Button}">
  <Border Name="Border" BorderBrush="Orange" BorderThickness="3" CornerRadius="2"
   Background="Red" TextBlock.Foreground="White">
    <ContentPresenter RecognizesAccessKey="True"
     Margin="{TemplateBinding Padding}"></ContentPresenter>
  </Border>
  <ControlTemplate.Triggers>
    <Trigger Property="IsMouseOver" Value="True">
      <Setter TargetName="Border" Property="Background" Value="DarkRed" />
    </Trigger>                
    <Trigger Property="IsPressed" Value="True">
      <Setter TargetName="Border" Property="Background" Value="IndianRed" />
      <Setter TargetName="Border" Property="BorderBrush" Value="DarkKhaki" />
    </Trigger>        
  </ControlTemplate.Triggers>
</ControlTemplate>
```

Border元素有一个名字，名字被用于设置器的***TargetName\***属性。设置器能更新模板Border的Background和BorderBrush属性。使用名字最容易确保被更新模板的单个的特殊的部分。你能创造一个元素类型规则，影响所有的Border元素（因为你知道按钮模板只存在一个边界），但是如果你后来改变模板，命名方法更清楚、更灵活。

焦点指示器。不能改变现有的边界，但可以添加一个透明的，虚线边界长方形元素。为了确保虚线长方形重叠在内容上，把它放置在内容的单元格内。

```
<ControlTemplate x:Key="ButtonTemplate" TargetType="{x:Type Button}">
  <Border Name="Border" BorderBrush="Orange" BorderThickness="3" CornerRadius="2"
   Background="Red" TextBlock.Foreground="White">
    <Grid>
      <Rectangle Name="FocusCue" Visibility="Hidden" Stroke="Black"
       StrokeThickness="1" StrokeDashArray="1 2"
       SnapsToDevicePixels="True"></Rectangle>
      <ContentPresenter RecognizesAccessKey="True"
       Margin="{TemplateBinding Padding}"></ContentPresenter>
    </Grid>
  </Border>
  <ControlTemplate.Triggers>
    <Trigger Property="IsMouseOver" Value="True">
      <Setter TargetName="Border" Property="Background" Value="DarkRed" />
    </Trigger>                
    <Trigger Property="IsPressed" Value="True">
      <Setter TargetName="Border" Property="Background" Value="IndianRed" />
      <Setter TargetName="Border" Property="BorderBrush" Value="DarkKhaki" />
    </Trigger>
    <Trigger Property="IsKeyboardFocused" Value="True">
      <Setter TargetName="FocusCue" Property="Visibility" Value="Visible" />
    </Trigger>        
  </ControlTemplate.Triggers>
</ControlTemplate>
```

 

最后一个触发器是：

```
<Trigger Property="IsEnabled" Value="False">
  <Setter TargetName="Border" Property="TextBlock.Foreground" Value="Gray" />
  <Setter TargetName="Border" Property="Background" Value="MistyRose" />
</Trigger>
```

 这个触发器应该有最高优先级。所以，将其放在触发器列表的最后。

### 使用动画的触发器

详见484页。

## 组织模板资源

这一部分讲如何利用资源词典，见485页。

在资源词典中定义控件模板：

```
<ResourceDictionary 
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" >
  <ControlTemplate x:Key="ButtonTemplate" TargetType="{x:Type Button}">
    ...
  </ControlTemplate>
</ResourceDictionary>  
```

在应用程序中合并资源词典：

```
<Application x:Class="SimpleApplication.App"
 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
 StartupUri="Window1.xaml">
  <Application.Resources>
    <ResourceDictionary>
      <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="Resources\Button.xaml" />
      </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
  </Application.Resources>
</Application>
```

 

### 重构按钮控件模板

见486页。

### 使用样式应用模板

模板代码：

```
<ControlTemplate x:Key="CustomButtonTemplate" TargetType="{x:Type Button}">
  <Border Name="Border" BorderThickness="2" CornerRadius="2"
   Background="{TemplateBinding Background}"
   BorderBrush="{TemplateBinding BorderBrush}">
    <Grid>
      <Rectangle Name="FocusCue" Visibility="Hidden" Stroke="Black"
       StrokeThickness="1" StrokeDashArray="1 2" SnapsToDevicePixels="True">
      </Rectangle>
      <ContentPresenter Margin="{TemplateBinding Padding}"
       RecognizesAccessKey="True"></ContentPresenter>
    </Grid>
  </Border>
  <ControlTemplate.Triggers>        
    <Trigger Property="IsKeyboardFocused" Value="True">
      <Setter TargetName="FocusCue" Property="Visibility"
        Value="Visible"></Setter>
    </Trigger>
  </ControlTemplate.Triggers>
</ControlTemplate>   
```

 样式：

```
<Style x:Key="CustomButtonStyle" TargetType="{x:Type Button}">
  <Setter Property="Control.Template"
   Value="{StaticResource CustomButtonTemplate}"></Setter>
  <Setter Property="BorderBrush"
   Value="{StaticResource Border}"></Setter>
  <Setter Property="Background"
     Value="{StaticResource DefaultBackground}"></Setter>
  <Setter Property="TextBlock.Foreground"
       Value="White"></Setter>
  <Style.Triggers>
    <Trigger Property="IsMouseOver" Value="True">
      <Setter Property="Background"
       Value="{StaticResource HighlightBackground}" />
    </Trigger>
    <Trigger Property="IsPressed" Value="True">
      <Setter Property="Background"
       Value="{StaticResource PressedBackground}" />
    </Trigger>
    <Trigger Property="IsEnabled" Value="False">
      <Setter Property="Background"
       Value="{StaticResource DisabledBackground}"></Setter>
    </Trigger>
  </Style.Triggers>
</Style>
```

为使用新的模板，你需要设置Style属性而不是Template属性：

```
<Button Margin="10" Padding="5" Style="{StaticResource CustomButtonStyle}">
 A Simple Button with a Custom Template</Button>
```

### 自动应用模板

类型化的样式：

```
<Style TargetType="{x:Type Button}">
  <Setter Property="Control.Template" Value="{StaticResource ButtonTemplate}">
</Style>
```

将样式属性设为空，表示使用默认样式：

```
<Button Style="{x:Null}" ... ></Button>
```

 