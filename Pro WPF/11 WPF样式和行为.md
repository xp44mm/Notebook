# 11 WPF样式和行为

先创建一套样式描述细节，然后仅靠设置`Style`属性应用样式。

行为封装一些基本的UI功能，然后用一两行标记附加到元素上，实现功能。

## 样式基础

样式是一种重要的资源。

样式是属性值的集合，能被应用到一个元素。就像CSS，WPF样式允许你定义格式化特性并且遍及应用程序地应用它们去确保一致性。就像CSS，WPF样式可以级联通过元素树自动地应用到指定的元素类型的目标。而且，WPF样式更强力，因为他们可以设置任何依赖属性。那意味着你可以使用它们标准化格式化以外的特性，诸如控制控件的行为属性。WPF样式也支持触发器，这允许你当另一个属性被改变时改变控件的样式，并且他们可以使用模板去重定义控件的内建外观。

属性值继承，是依赖属性的特征之一。当你在窗口水平设置属性时，窗口内部所有的元素将获得相同的值，除非显式地覆盖他们。

考虑下面代码：

```
<Window.Resources>
  <Style x:Key="BigFontButtonStyle">
    <Setter Property="Control.FontFamily" Value="Times New Roman" />
    <Setter Property="Control.FontSize" Value="18" />
    <Setter Property="Control.FontWeight" Value="Bold" />
  </Style>    
</Window.Resources>
```

这段标记创造单个资源：一个`System.Windows.Style`对象。样式对象持有一个设置器集合，带有三设置器对象，每个设置器设置一个属性。每个设置器对象指出要设置属性的名字。和该属性的值。如同所有的资源，样式对象有一个用于检索资源的关键字。在本例中，关键字是`BigFontButtonStyle`。（按照约定，样式关键字的结尾是`Style`。）

每个WPF元素只能使用一个样式（或可以不使用样式）。一个元素通过Style属性应用样式。例如，为配置一个按钮使用你先前创造的样式，按钮像这样应用样式资源：

```
<Button Padding="5" Margin="5" Name="cmd"
 Style="{StaticResource BigFontButtonStyle}"> 
  A Customized Button
</Button>
```

当然，可以使用代码指定样式。方法是用`FindResource()`方法：

```
cmd.Style = (Style)cmd.FindResource("BigFontButtonStyle");
```

Style类一共有5个重要的属性：

| 属性       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| Setters    | Setter或EventSetter对象的集合。用于自动地设置属性值和附着事件处理器。 |
| Triggers   | TriggerBase对象的集合，并且允许你自动地改变样式设置。例如，当另一个属性改变时或当一个事件发生时，你能修改一个样式。 |
| Resources  | 你希望用于样式的资源集合。例如，有时需要用一个对象设置多个属性。在那种情况下，更有效的是创造对象作为一个资源，然后在你的设置器对象中使用那资源，而不是在每个设置器中，使用嵌套标签创造对象。 |
| BasedOn    | 一个属性，允许你创造一个更特殊的样式。此样式继承（和可选地覆盖）另一个样式的设置。 |
| TargetType | 一个属性，识别样式所作用于的元素类型。此属性允许你创造仅影响某些元素的设置器，它允许你创造对于匹配的元素类型自动生效的设置器。 |

### 创建一个样式对象

你能创建各种水平的资源集合，窗口的，容器的，或者应用程序的。

实际上，你可以直接设置控件的样式：

```
<Button>
  <Button.Style>
    <Style>
      <Setter Property="Control.FontFamily" Value="Times New Roman" />
      <Setter Property="Control.FontSize" Value="18" />
      <Setter Property="Control.FontWeight" Value="Bold" />
    </Style>
  </Button.Style>
  <Button.Content>A Customized Button</Button.Content>
</Button>
```

### 设置属性

如你所见，每个Style对象包裹一个Setter对象的集合。每个Setter对象设置一个元素的一个属性。唯一限制是一个设置器只能改变一个依赖属性—不能修改其他属性。

在一些情况下，你不能使用一个简单的字符串设置属性值。例如，不能使用一个简单字符串创造一个`ImageBrush`对象。在这种情况下，你能用一个嵌套元素设置属性。这是一个例子：

```
<Style x:Key="HappyTiledElementStyle">
  <Setter Property="Control.Background">
    <Setter.Value>
      <ImageBrush TileMode="Tile"
        ViewportUnits="Absolute" Viewport="0 0 32 32"
        ImageSource="happyface.jpg" Opacity="0.3">
      </ImageBrush>
    </Setter.Value>
  </Setter>
</Style>
```

 

为识别你希望设置的属性，你需要提供一个类和一个属性的名字。但是，类名字不需要是定义属性的类。它也能是该类的一个派生类。例如，考虑下面`BigFontButton`样式的版本，`Button`类的引用代替`Control`类的引用：

```
<Style x:Key="BigFontButtonStyle">
  <Setter Property="Button.FontFamily" Value="Times New Roman" />
  <Setter Property="Button.FontSize" Value="18" />
  <Setter Property="Button.FontWeight" Value="Bold" />
</Style>
```

区别是派生类缩小了样式应用的范围。例如，这个样式可以影响`Button`控件，但是不能影响`Label`控件。对于使用`Control`类的前面的例子，这个样式即可以影响`Button`控件，也可以影响`Label`控件。

WPF忽略无法被应用的属性。

如果样式中的所有属性都应用到同类型的元素，可以设置`TargetType`属性，指出属性应用的类：

```
<Style x:Key="BigFontButtonStyle" TargetType="Button">
  <Setter Property="FontFamily" Value="Times New Roman" />
  <Setter Property="FontSize" Value="18" />
  <Setter Property="FontWeight" Value="Bold" />
</Style>
```

可以看见，所有设置器的`Property`属性内没有指明类名。

### 附加事件处理器

```
<Style x:Key="MouseOverHighlightStyle">
  <EventSetter Event="TextBlock.MouseEnter" Handler="element_MouseEnter" />
  <EventSetter Event="TextBlock.MouseLeave" Handler="element_MouseLeave" />
  <Setter Property="TextBlock.Padding" Value="5"/>
</Style>
```

样式可以直接嵌套`Setter`元素和`EventSetter`元素。

这是事件处理的代码：

```
private void element_MouseEnter(object sender, MouseEventArgs e)
{
    ((TextBlock)sender).Background =
      new SolidColorBrush(Colors.LightGoldenrodYellow);
}

private void element_MouseLeave(object sender, MouseEventArgs e)
{
    ((TextBlock)sender).Background = null;
}
```

鼠标进入和鼠标离开都是直接事件路由。

应用样式：

```
<TextBlock Style="{StaticResource MouseOverHighlightStyle}">
 Hover over me.
</TextBlock>
```

这个功能也可以用事件触发器实现。

对于冒泡事件，最好直接在容器元素中连接事件处理函数。

### 多层样式

使用属性值继承特征的属性包括`IsEnabled`，`IsVisible`，`Foreground`，以及所有字体属性。

基于之前的样式建立一个新样式，使用`BasedOn`属性：

```
<Window.Resources>
  <Style x:Key="BigFontButtonStyle">
    <Setter Property="Control.FontFamily" Value="Times New Roman" />
    <Setter Property="Control.FontSize" Value="18" />
    <Setter Property="Control.FontWeight" Value="Bold" />
  </Style>

  <Style x:Key="EmphasizedBigFontButtonStyle"
    BasedOn="{StaticResource BigFontButtonStyle}">
    <Setter Property="Control.Foreground" Value="White" />
    <Setter Property="Control.Background" Value="DarkBlue" />
  </Style>
</Window.Resources>
```

 

除非两个样式有语义上的联系，否则不要应用`BasedOn`属性使他们相关联。

### 自动应用样式到类型

样式资源可以自动应用到指定类型的元素。只需要设置`TargetType`属性指定目标类型，并且完全省略关键字。这时，实际上隐式使用了类型标记扩展：

```
x:Key="{x:Type Button}"
```

这是自动应用按钮样式的例子：

```
<Window.Resources>
  <Style TargetType="Button">
    <Setter Property="FontFamily" Value="Times New Roman" />
    <Setter Property="FontSize" Value="18" />
    <Setter Property="FontWeight" Value="Bold" />
  </Style>
</Window.Resources>

<StackPanel Margin="5">
  <Button Padding="5" Margin="5">Customized Button</Button>
  <TextBlock Margin="5">Normal Content.</TextBlock>
  <Button Padding="5" Margin="5" Style="{x:Null}">A Normal Button</Button>
  <TextBlock Margin="5">More normal Content.</TextBlock>
  <Button Padding="5" Margin="5">Another Customized Button</Button>
</StackPanel>
```

中间的按钮显式删除了样式。不用提供新样式，只需要设置`Style`属性为空。

## 触发器

使用触发器，你能自动化简单的样式改变。

触发器通过`Style.Triggers`集合链接到样式。每个样式能有无数个触发器，并且每个触发器是`System.Windows.TriggerBase`派生类的一个实例。

| 名字             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| Trigger          | 观察依赖属性一个改变，然后使用一个设置器改变样式。           |
| MultiTrigger     | 这类似于触发器，但是结合多重的条件。触发器生效之前，所有条件必须被满足。 |
| DataTrigger      | 触发器与数据绑定一起工作。它类似于触发器，除了它监视绑定数据的改变。 |
| MultiDataTrigger | 这结合多个数据触发器。                                       |
| EventTrigger     | 事件发生时，触发一个动画。                                   |

### 简单触发器

你能附加简单触发器到任何依赖属性。例如，你能响应`Control`类的`IsFocused`，`IsMouseOver`，和`IsPressed`属性改变，创造鼠标悬停和焦点效果。

每个简单触发器标识你监视的属性，以及你期望的值。当这值发生，你存储在`Trigger.Setters`集合中的设置器被应用。（不幸地，不能使用更世故的触发器逻辑，比较一个值落在一个范围内，执行一个计算，等等。在这些情况下，你应转而使用一个事件处理器。）

这里是一个触发器，当按钮获得键盘焦点时，背景变为暗红色：

```
<Style x:Key="BigFontButton">
  <Style.Setters>
    <Setter Property="Control.FontFamily" Value="Times New Roman" />
    <Setter Property="Control.FontSize" Value="18" />
  </Style.Setters>
  
  <Style.Triggers>
    <Trigger Property="Control.IsFocused" Value="True">
      <Setter Property="Control.Foreground" Value="DarkRed" />
    </Trigger>
  </Style.Triggers>
</Style>    
```

不需要写逻辑反转触发器。触发器停止应用，元素就恢复它的正常的外观。

触发器没有修改依赖属性的原始值。原始值可能是本地值，也可能是样式值。触发器一失活，触发之前的值就再次可用。

触发器集合可以包括多个触发器。如果，一个以上的触发器修改同一个属性，位于列表最后的触发器赢。

它不关心哪一个触发器先发生。例如，WPF不关心在你点击按钮之前一个按钮是否获得焦点。触发器被排列在你标记中的位置是它唯一关心的事。

`MultiTrigger`提供一个条件集合，让你定义一系列的属性和值的结合。只有所有的条件都是真，触发器开启：

```
<Style x:Key="BigFontButton">
  <Style.Setters>
    ...
  </Style.Setters>
  <Style.Triggers>
    <MultiTrigger>
      <MultiTrigger.Conditions>
        <Condition Property="Control.IsFocused" Value="True">
        <Condition Property="Control.IsMouseOver" Value="True">
      </MultiTrigger.Conditions>
      <MultiTrigger.Setters>
        <Setter Property="Control.Foreground" Value="DarkRed" />
      </MultiTrigger.Setters>
    </MultiTrigger>
  </Style.Triggers>
</Style>
```

### 事件触发器

事件触发器等待指定的事件被引发。事件触发器要求你提供一系列行为修改控件。这些行为被用于应用一个动画。

事件触发器也是放在`Style.Triggers`集合中：

```
<Style.Triggers>
    <EventTrigger RoutedEvent="Mouse.MouseEnter">
        。。。
    </EventTrigger>
</Style.Triggers>
```

见296页。

## 行为

### 支持行为

### 理解行为模型

### 创建行为

### 使用行为

### Blend的设计时行为支持