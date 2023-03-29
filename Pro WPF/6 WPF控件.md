# 6 WPF控件

WPF控件分类：

- 内容控件
- 标题内容控件
- 文本控件
- 列表控件
- 基于范围的控件
- 日期控件

## 控件类

控件是与用户交互的元素。控件可以获得焦点，能接受键盘或鼠标的输入。

所有控件的基类是`System.Windows.Control`类，这类包括一些基本功能：

- 对齐
- tab序列
- 背景、前景、边界
- 文本内容的字体

### 背景和前景刷子

控件包括两个属性`Background`和`Foreground`属性，这两个属性使用`Brush`对象。刷子对象的派生类包括`SolidColorBrush`、`LinearGradientBrush`、和`TileBrush`类。

#### 用代码设置颜色

为名为cmd的按钮设置背景色：

```csharp
cmd.Background = new SolidColorBrush(Colors.AliceBlue);
```

通过`Colors`类的静态属性获得预定义的颜色，将它传递给构造函数创建一个新的`SolidColorBrush`实例，将它赋值给按钮的背景属性。

也可使用系统颜色：

```csharp
cmd.Background = new SolidColorBrush(SystemColors.ControlColor);
```

`SystemColors`类也提供预制的属性返回`SolidColorBrush`对象：

```csharp
cmd.Background = SystemColors.ControlBrush;
```

你能创造一个颜色对象，依靠提供R，G，B值(红绿蓝)。每个值是从0到255一个数字：

```csharp
int red = 0; int green = 255; int blue = 0;
cmd.Foreground = new SolidColorBrush(Color.FromRgb(red, green, blue));
```

你能设置颜色的透明度，通过调用`Color.FromArgb()`方法，为其传递alpha值。alpha值为255是完全不透明，而为0是完全透明。

#### 用XAML设置颜色

在XAML中，只需要提供颜色的名字或颜色值，其他的工作由解析器负责。

```xaml
<Button Background="Red">A Button</Button>
```

用 #rrggbb 或 #aarrggbb格式提供颜色值：

```xaml
<Button Background="#FFFF0000">A Button</Button>
```

刷子支持自动改变通知。刷子从`System.Windows.Freezable`类派生而来。`Freezable`类有两个状态：可读状态，只读状态(冻结)

控件类还定义了`BorderBrush`和`BorderThickness`属性。

### 字体

Control类定义几个字体相关的属性。决定控件文本的外观。这些属性列在表6-1。

| 名字        | 描述 |
| ----------- | ---- |
| FontFamily  |      |
| FontSize    |      |
| FontStyle   |      |
| FontWeight  |      |
| FontStretch |      |

`Control`类没有定义任何使用它字体的属性。然而许多控件包括`Text`属性，没有定义为`Control`基类的成员。明显地，除非被派生类使用，字体属性没有任何意义。

#### 字体家族

 

### 鼠标光标

## 内容控件

内容控件是更特殊的控件类型，它能拥有并显示一件内容。技术上，内容控件是能包含单个嵌套元素的控件。内容控件与布局容器的区别是，内容控件只能包含一个子元素，而布局容器可以拥有任意个子元素。

当然，你仍然能把多个内容包装到内容控件中。诀窍是把它们都包裹到单个容器中，诸如一个`StackPanel`、或一个`Grid`。例如，`Window`类本身是一个内容控件。明显地，窗口经常有大量内容，但是，它们都被包裹在一顶层的容器中(典型地，`Grid`)。

所有的内容控件起源于`ContentControl`抽象类。内容控件类主要包括：一些公共控件`Label`、`ToolTip`。各种按钮控件`Button`、`RadioButton`、`CheckBox`。一些更专用的`ScrollViewer`、`UserControl`。`Window`本身是个内容控件。

最后，存在一个内容控件子集，依靠从`HeaderedContentControl`类派生，增加了一层继承。包括`GroupBox`，`TabItem`，和`Expander`控件。这些控件有内容区域和标题区域，标题区域用于显示某种标题。

另外，用于导航的`Frame`、用于其他控件内部的`ListBoxItem`、`StatusBarItem`等也是内容控件。

### Content属性

`ContentControl`类添加一个`Content`属性，接受单个对象。`Content`属性支持任何类型的对象，但是它把对象分为两组，并且区别对待：

- 不派生自`UIElement`的对象：内容控件调用`ToString()`获得这些控件的文本，然后显示文本。
- 派生自`UIElement`的对象：这些对象包括所有可视元素。使用`UIElement.OnRender()`方法，被显示在内容控件内部。

例如，一个提供简单字符串的按钮：

```xaml
<Button Margin="3">Text content</Button>
```

使用Image类放置一个图像到按钮内：

```xaml
<Button Margin="3">
  <Image Source="happyface.jpg" Stretch="None" />
</Button>
```

放置一个包裹图像和文本的容器到按钮内：

```xaml
<Button Margin="3">
  <StackPanel>
    <TextBlock Margin="3">Image and text button</TextBlock>
    <Image Source="happyface.jpg" Stretch="None" />
    <TextBlock Margin="3">Courtesy of the StackPanel</TextBlock>
  </StackPanel>
</Button>
```

甚至可以在按钮中放置其他内容控件，如文本框，按钮、组合框。虽然这样做不太合理，但是WPF允许这样。

`Window`是内容控件，但是他只能是顶级容器，不能嵌套在其他元素中。

内容控件的其他属性包括：

`HasContent`属性，为真时表示控件有内容。

`ContentTemplate`属性，是一个告诉控件如何显示对象的模板。使用`ContentTemplate`，更智能地显示非`UIElement`派生来的对象。你可以获取对象的各种属性值，并整理放入更复杂的标记中。

### 对齐内容

`HorizontalContentAlignment`、`VerticalContentAlignment`取值为`Top`, `Bottom`, `Left`, `Right`、`Center`、`Stretch`

`Padding`属性指控件边界到其内容的距离。

`HorizontalContentAlignment`，`VerticalContentAlignment`，和`Padding`属性定义在`Control`类中，不在更特殊的`ContentControl`类。因为，一些不是内容控件的控件也有某种内容。例如，`TextBox`不是内容控件，它使用上述属性设置输入文本。

### WPF内容哲学

内容控件减少了控件的数量，但是，增加了一点控件的复杂度。

### 标签(Label)

标签控件主要的功能是助记键，使链接控件获得焦点的快捷键。标签控件的`Target`属性指定链接控件。为了设置`Target`，使用绑定表达式指向另一个控件。

```xaml
<Label Target="{Binding ElementName=txtA}">Choose _A</Label>
<TextBox Name="txtA"></TextBox>
<Label Target="{Binding ElementName=txtB}">Choose _B</Label>
<TextBox Name="txtB"></TextBox>
```

标签文本下划线指明快捷键。如果真需要一个下划线，可以连续输入两个下划线转义。

同时按下Alt和所指定的快捷键，链接的控件就会获得焦点。例如，在本例中，按下Alt+A，焦点就会跳到`txtA`控件。

如果不需要助记键功能，可以考虑使用`TextBlock`。

### 按钮类的控件

按钮类的控件包括`Button`、`CheckBox`、和`RadioButton`。他们都从`ButtonBase`派生。

`Click`支持命令功能。

`ClickMode`属性，`ClickMode.Release`，`ClickMode.Press`，`ClickMode.Hover`

按钮也支持快捷键。

### 按钮

每个窗口可以有取消按钮和默认按钮，通过设置按钮的`IsCancel`、`IsDefault`属性。详见159页。

另外，还有一个容易混淆的`IsDefaulted`属性，见160页侧边条。

### ToggleButton、RepeatButton

从`ButtonBase`派生的类还有三个：

- `GridViewColumnHeader`
- `RepeatButton`
- `ToggleButton`

`RepeatButton` 和`ToggleButton`都位于`System.Windows.Controls.Primitives`名字空间。常用于组合、或派生成其他控件，也可以单独使用。

### CheckBox

`CheckBox`和`RadioButton`都是从`ToggleButton`派生。

`ToggleButton`添加了一个`IsChecked`属性，它是一个可空布尔值。

为了在WPF标记中分配一个空值，使用空标记扩展，如下所示：

```xaml
<CheckBox IsChecked="{x:Null}">A check box in indeterminate state</CheckBox>
```

`ToggleButton`还有一个`IsThreeState`属性，它决定是否可以设置复选框为未定态。默认为false。

`ToggleButton`类定义三事件：`Checked`，`Unchecked`，和`Indeterminate`事件。

### RadioButton

默认情况下，单选按钮按它们的容器分组。`RadioButton`的`GroupName`属性允许你覆盖这行为。

```xaml
<StackPanel>
  <GroupBox Margin="5">
    <StackPanel>
      <RadioButton>Group 1</RadioButton>
      <RadioButton>Group 1</RadioButton>
      <RadioButton>Group 1</RadioButton>
      <RadioButton Margin="0,10,0,0" GroupName="Group2">Group 2</RadioButton>
    </StackPanel>
  </GroupBox>
  <GroupBox Margin="5">
    <StackPanel>
      <RadioButton>Group 3</RadioButton>
      <RadioButton>Group 3</RadioButton>
      <RadioButton>Group 3</RadioButton>
      <RadioButton Margin="0,10,0,0" GroupName="Group2">Group 2</RadioButton>
    </StackPanel>
  </GroupBox>
</StackPanel>
```

不需要使用`GroupBox`容器包裹单选按钮组，但这是一个普遍的约定。`GroupBox`显示一个边界和一个标题。

## 专用的容器

内容控件也包括一些专用的容器。

`ScrollViewer`直接从`ContentControl`继承。

`ContentControl`类派生了`HeaderedContentControl`类，这个类包括一个标题和一个内容。标题和内容都可以嵌套单个子元素。`HeaderedContentControl`类派生了几个子类：`GroupBox`，`TabItem`，和`Expander`。

### ScrollViewer

尽管`ScrollViewer`能包裹任何元素，但是一般情况下，它包裹一个布局容器。

```xaml
<ScrollViewer>
  <Grid Margin="3,3,10,3">
  </Grid>
</ScrollViewer>
```

`VerticalScrollBarVisibility`属性，此属性是`ScrollBarVisibility`枚举。`Visible`、`Auto`、`Disabled`。默认值是`Visible`。

`HorizontalScrollBarVisibility`属性，默认值是`Hidden`。

#### 编程控制滚动

详见171页。

#### 自定义滚动

详见172页。

### GroupBox

`GroupBox`从`HeaderedContentControl`类派生。

```xaml
<GroupBox Header="A GroupBox Test" Padding="5"
  Margin="5" VerticalAlignment="Top">
  <StackPanel>
    <RadioButton Margin="3">One</RadioButton>
    <RadioButton Margin="3">Two</RadioButton>
    <RadioButton Margin="3">Three</RadioButton>
    <Button Margin="3">Save</Button>
  </StackPanel>
</GroupBox>
```

`GroupBox`仍然要求一个布局容器布置内容。`GroupBox`没有特别的功能，只是一个装饰控件。

### TabItem

`TabItem`代表`TabControl`的一个选项卡。`TabItem`类添加了`IsSelected`属性，指示选项卡是否是`TabControl`当前显示的选项卡。

```xaml
<TabControl Margin="5">
  <TabItem Header="Tab One">
    <StackPanel Margin="3">
      <CheckBox Margin="3">Setting One</CheckBox>
      <CheckBox Margin="3">Setting Two</CheckBox>
      <CheckBox Margin="3">Setting Three</CheckBox>
    </StackPanel>
  </TabItem>
  <TabItem Header="Tab Two">
    ...
  </TabItem>
</TabControl>
```

通过设置`TabControl`的`TabStripPlacement`属性，可以将选项卡从正常的顶部放到侧边。

正如`Content`属性，`Header`属性能接受任何类型的对象。这是一个例子：

```xaml
<TabControl Margin="5">
  <TabItem>
    <TabItem.Header>
      <StackPanel>
        <TextBlock Margin="3" >Image and Text Tab Title</TextBlock>
        <Image Source="happyface.jpg" Stretch="None" />
      </StackPanel>
    </TabItem.Header>

    <StackPanel Margin="3">
      <CheckBox Margin="3">Setting One</CheckBox>
      <CheckBox Margin="3">Setting Two</CheckBox>
      <CheckBox Margin="3">Setting Three</CheckBox>
    </StackPanel>
  </TabItem>

  <TabItem Header="Tab Two"></TabItem>
</TabControl>
```

### Expander

见175页。

## 文本控件

WPF包括三文本输入控件：`TextBox`，`RichTextBox`，和`PasswordBox`。`PasswordBox`直接从`Control`派生。`TextBox`和`RichTextBox`控件派生自`TextBoxBase`。

不同于内容控件，文本框仅限于他们能包含的内容类型。`TextBox`永远存储一个字符串(`Text`属性)。`PasswordBox`也处理字符串内容(`Password`属性)，尽管它使用`SecureString`。只有`RichTextBox`有能力存储更世故的内容：一个`FlowDocument`。

### 多行文本

`MaxLength`属性，设置`TextBox`允许的最大字符数。

`TextWrapping`属性，设置为`Wrap`或`WrapWithOverflow`，表示自动换行。

`MinLines`和`MaxLines`属性，设置`TextBox`的最小（最大）可见的行数。

`LineCount`属性，可以检索出文本框中的文本有多少行。

`VerticalScrollBarVisibility`、`HorizontalScrollBarVisibility`属性，设置滚动条的可视状态。

`AcceptsReturn`属性，设置为真表示`TextBox`接受回车。默认情况下，`TextBox`不接受回车。

`AcceptsTab`属性为真表示接受`Tab`键，默认情况下，`TextBox`不接受Tab键。

`IsReadOnly`属性，阻止编辑文本。

### 文本选择

见180页。

### 拼写检查

见181页。

### 密码框

见183页。

## 列表控件

列表控件的基类是`ItemsControl`类。

每个`ItemsControl`类都有项目列表。有二种方法填充项目列表。最直白的方法是直接添加项到项集合，使用代码或XAML。更常用的方法是数据绑定。这种方法是设置`ItemsSource`属性为要显示的数据项集。

`ItemsControl`类派生的一个主要分支为Selector类，包括`ListBox`, `ComboBox`，和`TabControl`类。可以跟踪当前选择项(`SelectedItem`)，或它的位置(`SelectedIndex`)。

其余的列表类不支持当前项，直接从`ItemsControl`类派生。这些类包括`Menu`、`Toolbar`、`TreeView`等。

### ListBox

设置`SelectionMode`属性为`Multiple`可以多选。在多选模式下，要使用`SelectedItems`集合而不是`SelectedItem`。

添加列表框项：

```xaml
<ListBox>
  <ListBoxItem>Green</ListBoxItem>
  <ListBoxItem>Blue</ListBoxItem>
  <ListBoxItem>Yellow</ListBoxItem>
  <ListBoxItem>Red</ListBoxItem>
</ListBox>
```

`ListBoxItem`派生自`ContentControl`。

例如，创建一个图像的列表框：

```xaml
<ListBox>
  <ListBoxItem>
    <Image Source="happyface.jpg"></Image>
  </ListBoxItem>
  <ListBoxItem>
    <Image Source="happyface.jpg"></Image>
  </ListBoxItem>
</ListBox>
```

可以省略上例的`ListBoxItem`，列表框足够智能，可以识别列表项：

```xaml
<ListBox>
  <StackPanel Orientation="Horizontal">
    <Image Source="happyface.jpg"  Width="30" Height="30"></Image>
    <Label VerticalContentAlignment="Center">A happy face</Label>
  </StackPanel>
  
  <StackPanel Orientation="Horizontal">
    <Image Source="redx.jpg" Width="30" Height="30"></Image>
    <Label VerticalContentAlignment="Center">A warning sign</Label>
  </StackPanel>
  
  <StackPanel Orientation="Horizontal">
    <Image Source="happyface.jpg"  Width="30" Height="30"></Image>
    <Label VerticalContentAlignment="Center">A happy face</Label>
  </StackPanel>
</ListBox>
```

下面是一个列表项为复选框的例子：

```xaml
<ListBox Name="lst" SelectionChanged="lst_SelectionChanged"
  CheckBox.Click="lst_SelectionChanged">
  <CheckBox Margin="3">Option 1</CheckBox>
  <CheckBox Margin="3">Option 2</CheckBox>
</ListBox>
```

如果你没有使用`ListBoxItem`填充列表项，当你读`SelectedItem`值时，将不会得到`ListBoxItem`对象，而是你放置到列表中的对象。在上例中，`SelectedItem`将提供一个`CheckBox`对象。

下面代码获取当前的选择项，显示该项目是否被选中。

```csharp
private void lst_SelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (lst.SelectedItem == null) return;
    txtSelection.Text = String.Format(
      "You chose item at position {0}.\r\nChecked state is {1}.",
      lst.SelectedIndex,
      ((CheckBox)lst.SelectedItem).IsChecked);
}
```

如果你希望知道哪一个项目失去选择，你能使用`SelectionChangedEventArgs`对象的`RemovedItems`属性。类似地，`AddedItems`属性告诉你哪一个项目被添加到选择。在单选模式，无论何时选择改变，永远是一项目被添加和一项目被移除。在`multiple`或`extended`模式，就不一定了。

下面是遍历列表项目集合的代码：

```csharp
private void cmd_ExamineAllItems(object sender, RoutedEventArgs e)
{
    var sb = new StringBuilder();
    foreach (CheckBox item in lst.Items)
    {
        if (item.IsChecked == true)
        {
            sb.Append(item.Content);
            sb.Append(" is checked.");
            sb.Append("\r\n");
        }
    }
    txtSelection.Text = sb.ToString();
}
```

`ListBoxItem`也有一些额外的功能：`IsSelected`属性、`Selected`事件和`Unselected`事件。这些功能也可以通过`ListBox`的`SelectedItem`属性和`SelectionChanged`事件实现。

有趣地，存在一个技术，当你使用嵌套对象方法时，获取指定对象的`ListBoxItem`包裹器。诀窍是`ContainerFromElement()`方法。这是使用这个技术的代码检查第一项目是否是列表的被选择项：

```csharp
var item = (ListBoxItem)lst.ContainerFromElement(
    (DependencyObject)lst.SelectedItems[0]);
MessageBox.Show("IsSelected: " + item.IsSelected.ToString());
```

### ComboBox

组合框用法与列表框基本相同。

如果你允许用户通过在组合框键入文本来选择一个项目，你必须设置`IsEditable`属性为true，并且你必须确保你存储平凡的仅文本的`ComboBoxItem`对象，或一个提供有意义的`ToString()`表示法的对象。例如，如果你填充一个可编辑的带有图像对象组合框，文本出现在上面的部分是Image类的全名，这不是非常使用。

## 基于范围的控件

基于范围的控件基类是`RangeBase`类，从`Control`类派生。包括`ScrollBar`, `Slider`, 和`ProgressBar`类。`RangeBase`类的属性包括：

`Value`、`Maximum`、`Minimum`、`SmallChange`、`LargeChange`

因为有`ScrollView`控件，`ScrollBar`很少用。现在关注`Slider`和`ProgressBar`。

### Slider

见188页

### ProgressBar

见190页

## 日期控件

见190页。