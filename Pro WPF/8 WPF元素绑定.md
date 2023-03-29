# 8 WPF元素绑定

数据绑定是一个关系，告诉WPF从一个来源对象抽取一些信息，并且使用数据绑定设置一个目标对象中的属性。目标属性总是一个依赖属性，并且它通常是WPF元素—毕竟，WPF数据绑定的终极目标是在用户界面上显示一些信息。但是，源对象能是任何事物，范围从另一个WPF元素到一个ADO.NET数据对象或仅是一个自定义的数据对象。

## 捆绑几个元素在一起

数据绑定的最简单场景：源对象是WPF元素并且源属性是依赖属性。依赖属性内建支持改变通知，当源对象的依赖属性值改变时，目标对象的绑定属性被立即更新。

看一个例子，这个例子用滑杆控制字体的大小。

[![运行](http://images.cnitblog.com/blog/128579/201304/21141813-0e7d3853930b47d380d8292cdac1ffb2.png)](http://images.cnitblog.com/blog/128579/201304/21141812-dd101e1be4854a4caa4cdc7f859806d5.png)

### 绑定表达式

当使用数据绑定时，不需要修改源对象。在这个例子中是滑杆，仅仅像往常一样，设置正确的范围值就行。

```
<Slider Name="sliderFontSize" Margin="3"
 Minimum="1" Maximum="40" Value="10"
 TickFrequency="1" TickPlacement="TopLeft">
</Slider>
```

绑定定义在TextBlock元素上，不是使用字面值设置FontSize，而是使用绑定表达式。

```
<TextBlock Margin="10" Text="Simple Text" Name="lblSampleText"
 FontSize="{Binding ElementName=sliderFontSize, Path=Value}" >
</TextBlock>
```

数据绑定表达式使用XAML标记扩展，因此有花括号。表达式以Binding开始，表明正在构造System.Windows.Data.Binding类的一个实例。尽管你可以用几种方法配置绑定对象，目前，你只需要设置两个属性：ElementName指明源元素。Path指明源元素的属性。

之所以命名为Path而不是Property，是因为路径可能指向属性的属性(例如，FontFamily.Source)，或者使用索引器(Content.Children[0])。可以使用多重点号深入属性的属性的属性，等等。

如果希望引用一个附加属性，需要为属性名字加上括弧。例如，如果你绑定放置在网格中的一个元素，路径"(Grid.Row)”取回它所放置的行数。

如果数据源本身是数组则使用点号索引如.[\d+]

### 绑定错误

见229页。

###  

### 绑定模式

可以设置绑定对象的Mode参数，使绑定变成双路的。修改TextBlock的FontSize如下：

```
FontSize="{Binding ElementName=sliderFontSize, Path=Value, Mode=TwoWay}"
```

System.Windows.Data.BindingMode枚举有五个值：

| 名称           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| OneWay         | 源变化，则目标变化                                           |
| TwoWay         | 源变化，则目标变化；目标变化，则源变化                       |
| OneTime        | 在初始化时，目标根据源的值设置一次。此后源的变化与目标没有关系。 |
| OneWayToSource | 目标变化，则源变化                                           |
| Default        | 如果目标是用户可设置的属性（如TextBox.Text）是TwoWay，其余的为OneWay。 |

#### OneWayToSource

因为目标属性必须是依赖属性，所以当真正的目标属性不是依赖属性，而真正的源是依赖属性时，可以应用这个选项。

#### Default

这一段讨论为什么默认值有时双路，有时单路。详见231页。

### 用代码创建绑定

用代码创建前面例子中的绑定：

```
var binding = new Binding();
binding.Source = sliderFontSize;
binding.Path = new PropertyPath("Value");
binding.Mode = BindingMode.TwoWay;
lblSampleText.SetBinding(TextBlock.FontSizeProperty, binding);
```

用代码移除绑定的方法是，使用BindingOperations类的两个静态方法。ClearBinding()方法引用一个依赖属性，拥有你希望移除的绑定，而ClearAllBindings()移除一个元素所有的数据绑定：

```
BindingOperations.ClearAllBindings(lblSampleText);
```

ClearBinding()和ClearAllBindings()都使用ClearValue()方法。从DependencyObject基类继承而来的方法。ClearValue()简单地移除一个属性的局部值(本例中，是一个数据绑定表达)。

应尽量使用标记绑定元素，但是有时最好或者必须使用代码：

- 创造动态绑定：基于运行时信息修改绑定，或根据情况创造另一个绑定。
- 移除绑定：借助于ClearBinding()或ClearAllBindings()方法移除绑定，用常规办法设置属性。
- 创建自定义控件：略。

### 用代码取回绑定

有二个办法获得绑定信息。第一个办法是使用静态的BindingOperations.GetBinding()方法取回相应的绑定对象。提供二参数：被绑定的元素，和拥有绑定表达式属性。

例如，如果你有像这样的一个绑定：

```
<TextBlock Margin="10" Text="Simple Text" Name="lblSampleText"
 FontSize="{Binding ElementName=sliderFontSize, Path=Value}" >
</TextBlock>
```

可以用下面代码获得绑定：

```
var binding = BindingOperations.GetBinding(lblSampleText, TextBlock.FontSizeProperty);
```

一旦获得了绑定对象，就可以使用各种属性了如：Binding.ElementName、Binding.Path、Binding.Path.Path、和Binding.Mode等等。

调用BindingOperations.GetBindingExpression()方法可以获得BindingExpression对象，参数与GetBinding()方法一样：

```
var expression = BindingOperations.GetBindingExpression(lblSampleText, TextBlock.FontSizeProperty);

// 获得源元素
var boundObject = (Slider)expression.ResolvedSource;

// 从源元素获得你想要的数据，包括它的被绑定属性。
string boundData = boundObject.FontSize;
```

这个技术适用于，开始绑定数据对象，然后，使用ResolvedSource属性获得被绑定数据对象的一个引用。

### 多重绑定

同一个控件的几个属性都使用绑定。接本章开始的例子，可以增加一个文本框，用于设置TextBlock的Text属性：

```
<TextBlock Margin="3" Name="lblSampleText"
  FontSize="{Binding ElementName=sliderFontSize, Path=Value}"
  Text="{Binding ElementName=txtContent, Path=Text}">
</TextBlock>
```

你也可以链接数据绑定。TextBox的Text属性链接到TextBlock.FontSize属性，后者又包含一个绑定到Slider.Value的绑定表达式。

**绑定表达式要尽可能直接绑定真正的数据源**。

如何将目标属性绑定到多个源？有几种方法：

最简单的方法是改变数据绑定模式。最后设置的属性值生效。

```
<Slider Margin="3"
        TickFrequency="1" TickPlacement="TopLeft" Minimum="1" Maximum="40" 
        Value="{Binding ElementName=lblSampleText, Path=FontSize, Mode=TwoWay}">
</Slider>

<TextBox Margin="3" 
         Text="{Binding ElementName=lblSampleText, Path=FontSize, Mode=TwoWay}">
</TextBox>

<TextBlock Margin="3" Name="lblSampleText" FontSize="10">
    Sample Text
</TextBlock>
```

这个例子，既可以通过Slider，也可以通过TextBox控制文本的大小。

双路绑定使绑定表达式的位置非常灵活。但是，绑定的方法应该符合逻辑。在每个源元素设置数据表达式。在目标元素上设置初始值。

### 绑定更新

当使用单路或双路绑定时，改变的值会立即从源到目标传播。反方向从目标到源却不一定立即发生。值回传的行为取决于Binding.UpdateSourceTrigger属性：

| 名字            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| PropertyChanged | 当目标属性改变时，源被立即更新。                             |
| LostFocus       | 当目标属性改变并且目标失去焦点时，源被更新                   |
| Explicit        | 源不被更新除非调用BindingExpression.UpdateSource()方法。     |
| Default         | 更新行为取决于目标属性的元数据（从技术上，它的FrameworkPropertyMetadata.DefaultUpdateSourceTrigger属性）。大多数属性的默认行为是PropertyChanged，而TextBox.Text属性的默认行为是LostFocus。 |

记住这个表的值对于目标值如何更新是无效的，他们只在TwoWay或OneWayToSource绑定中控制源是如何更新的。

```
<TextBox Text="{Binding ElementName=txtSampleText, Path=FontSize, Mode=TwoWay,
 UpdateSourceTrigger=PropertyChanged}" Name="txtFontSize"></TextBox>
```

也可以使用UpdateSourceTrigger.Explicit模式，添加一个按钮更新源元素，按钮的代码如下：

```
// 获得文本框的绑定对象
var binding = txtFontSize.GetBindingExpression(TextBox.TextProperty);

// 更新链接的源(TextBlock)
binding.UpdateSource();
```

### 绑定延迟

绑定对象有个Delay属性，表示提交改变以后等待若干毫秒更新源对象。

```
<TextBox Text="{Binding ElementName=txtSampleText, Path=FontSize, Mode=TwoWay,
 UpdateSourceTrigger=PropertyChanged, Delay=500}" Name="txtFontSize"></TextBox>
```

## 绑定到非元素对象

数据绑定对数据源对象仅有的要求是要显示的信息必须是公有属性。

要绑定到非元素对象，你要去掉ElementName属性，用下面属性之一代替：

Source：这是直接指向源对象的一个引用，换句话说，提供数据的对象。

RelativeSource：这是使用RelativeSource对象指向源对象的一个引用。RelativeSource对象允许你相对于当前元素（持有绑定表达式的元素）指定数据源。RelativeSource属性是一个特殊的工具。常用于控件模板和数据模板。

DataContext：如果你没有指定一个数据源，WPF从当前元素开始向上搜索元素树。检测每个元素的DataContext属性，使用第一个不为空。DataContext属性适用于绑定相同对象的几个属性到不同的元素，因为你可以在更上层容器对象上设置DataContext属性，而不是直接在目标元素上设置它。

### Source

获得数据最简单的方法是，设置Source为静态对象。

```
<TextBlock Text="{Binding Source={x:Static SystemFonts.IconFontFamily}, Path=Source}">
</TextBlock>
```

另一个办法是绑定数据源到事先定义的资源。

```
    <Window.Resources>
        <FontFamily x:Key="CustomFont">Calibri</FontFamily>
    </Window.Resources>
    
    <TextBlock Text="{Binding Source={StaticResource CustomFont}, Path=Source}"></TextBlock>
```

### RelativeSource

RelativeSource属性允许你基于它和目标的相对关系指向数据源。例如，你能使用RelativeSource属性绑定一个元素到它自己，或绑定到父元素，或父元素的父元素。

设置RelativeSource属性需要创建一的RelativeSource对象，除了使用扩展标记，还可以通过属性元素设置它。下面代码是如何在TextBlock中显示窗口的标题：

```
<TextBlock>
    <TextBlock.Text>
        <Binding Path="Title">
            <Binding.RelativeSource>
                <RelativeSource Mode="FindAncestor" AncestorType="{x:Type Window}" />
            </Binding.RelativeSource>
        </Binding>
    </TextBlock.Text>
</TextBlock>
```

用扩展标记完成同样的功能，这里需要结合使用Binding和RelativeSource扩展标记：

```
<TextBlock Text="{Binding Path=Title, RelativeSource={RelativeSource FindAncestor, AncestorType={x:Type Window}} }">
</TextBlock>
```

RelativeSourceMode 枚举值：

| 名字            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| Self            | 表达式绑定到同一个元素的另一个属性                           |
| FindAncestor    | 表达式绑定到祖先元素。WPF沿着元素树向上搜索，直到发现要找的祖先元素。为了指定祖先元素，你必须也设置AncestorType属性，指出要找的祖先元素类型。可选地，你使用AncestorLevel属性，忽略一定数量的指定元素。例如，如果你希望绑定到沿树向上第三个ListBoxItem类型的元素，你将设置AncestorType={x:Type ListBoxItem}和AncestorLevel=3，因此掠过前两个ListBoxItem。默认，AncestorLevel是1，搜索发现第一次匹配时就停止。 |
| PreviousData    | 表达式绑定到数据绑定列表的前一个数据项，你将会在列表项目使用。 |
| TemplatedParent | 表达式绑定到应用模板的元素。这个模式只应用于控件模板或者数据模板的内部。 |

源对象和目标对象在标记不同部分，在控件模板和数据模板时。例如，如果你建立一个数据模板，改变项目在列表呈现的方式，你可能需要访问顶层的列表框对象读取一个属性。

### DataContext

设置上下文属性，可以通过属性元素，静态属性，或者资源，同Binding.Source的设置方式。使用上下文的示例：

```
    <StackPanel DataContext="{x:Static SystemFonts.IconFontFamily}">
        <TextBlock Text="{Binding Path=Source}"></TextBlock>
        <TextBlock Text="{Binding Path=LineSpacing}"></TextBlock>
        <TextBlock Text="{Binding Path=FamilyTypefaces[0].Style}"></TextBlock>
        <TextBlock Text="{Binding Path=FamilyTypefaces[0].Weight}"></TextBlock>
    </StackPanel>
```

当绑定表达式的source信息缺失，WPF检查元素的DataContext属性。如果它为空，沿元素树向上搜索，寻找第一个非空数据上下文。（所有元素的DataContext属性初始化为空）如果WPF发现一个非空数据上下文，使用它到绑定。如果没有找到，绑定表达式不应用任何值到目标属性。