# 3 WPF布局

## 理解WPF布局

WPF采用了流布局。流布局允许控件变大，并且能挤开其他控件。

### WPF布局哲学

在WPF中，容器决定了布局。尽管有几种容器，WPF窗口遵守几个关键的原则：

- 元素（诸如控件）不应显式设置尺寸
- 元素不指明定位坐标
- 布局容器在子元素之间共享可用空间
- 布局容器能被嵌套

### 布局过程

WPF布局分二阶段：测量阶段和布置阶段。测量阶段，容器遍历它的所有子元素，要求它们提供它们的优先尺寸。布置阶段，容器摆放子元素到合适的位置。

有时容器不够大不能容纳元素。在这种情况下，为了适应可见区域，容器必须截断冒犯元素。如你所见，通过设置一个最小窗口尺寸，经常能避免这情况。

### 布局容器

所有WPF布局容器都派生自`System.Windows.Controls.Panel`抽象类。`Panel`类增加了几个成员，包括三个公开的属性。

| `Background` 
Brush，如果要接收鼠标事件，必须设置此属性非空。

| `Children`
是面板中项集。这是第一层项目—换句话说，这些项本身可能包含更多项。 

| `IsItemsHost` 
一个布尔值。如果为真，面板被用于显示`ItemsControl`控件的项集。 

`Panel`本身只是一个起点。

核心布局面板包括：`StackPanel`、`WrapPanel`、`DockPanel`、`Grid`、`UniformGrid`

专用的布局面板包括：`TabPanel`、`ToolbarPanel`、`ToolbarOverflowPanel`、`VirtualizingStackPanel`

## StackPanel

栈面板是最简单的布局容器之一，它简单地以单行或单列的方式堆垛它的子元素。

例如，考虑这个窗口，它包含有四个按钮的栈：

```xaml
    <StackPanel>
        <Label>A Button Stack</Label>
        <Button>Button 1</Button>
        <Button>Button 2</Button>
        <Button>Button 3</Button>
        <Button>Button 4</Button>
    </StackPanel>
```

默认情况下，StackPanel自顶到底排列元素。

通过设置Orientation属性，可以水平排列控件：

```xaml
<StackPanel Orientation="Horizontal">
```

### 布局属性

布局属性包括`HorizontalAlignment`,`VerticalAlignment`;`Margin`; `MinWidth`, `MinHeight`, `MaxWidth`, `MaxHeight`; `Width`, `Height`。

```fsharp
| HorizontalAlignment |      |
| VerticalAlignment   |      |
| Margin              |      |
| MinWidth、MinHeight |      |
| MaxWidth、MaxHeight |      |
| Width、Height       |      |
```

所有这些属性都在`FrameworkElement`类中定义。所有的图形部件都可以使用这些属性。

### Alignment

见59页

### Margin

见60页

### 最小、最大、和显式尺寸

见62页

### Border

`Border`不是一个布局面板，但是常与布局面板一起使用。

`Border`类非常简单。它取单个的嵌套内容（这是经常一个布局面板）并且添加一个背景或一个围绕的边界。

`Border`类的属性：

| `Background`
设置内容背后的背景依靠Brush对象。你能使用一个实体颜色或更奇异的什么。 

| `BorderBrush`、`BorderThickness`
设置出现在Border对象的边界的边界的颜色，使用一个刷子对象，并且设置边界的宽度，各自地。为显示一个边界，你必须二个属性都设置。 

| `CornerRadius` 
边界拐角的倒圆角半径

| `Padding`
边界与内容之间的距离，内边距。

这里有一个倒圆角的边界，环绕一个栈面板，栈面板包括一组按钮：

```xaml
<Border Margin="5" Padding="5" Background="LightYellow"
 BorderBrush="SteelBlue" BorderThickness="3,5,3,5" CornerRadius="3"
 VerticalAlignment="Top">
  <StackPanel>
    <Button Margin="3">One</Button>
    <Button Margin="3">Two</Button>
    <Button Margin="3">Three</Button>
  </StackPanel>
</Border>
```

注意：`Border`是一个装饰者，所有的装饰者都派生自`System.Windows.Controls.Decorator`类。

## WrapPanel

`WrapPanel.Orientation`属性默认为`Horizontal`，子元素按行的布置。但是,设置`Orientation`为`Vertical`后，则按列布置。

## DockPanel

通过一个附加属性`Dock`，子元素可以选择他们希望停靠的边。这个属性能被设置为`Left`，`Right`，`Top`，或`Bottom`。

```xaml
<TextBox DockPanel.Dock="Top"
```

`DockPanel`的`LastChildFill`属性表示其最后一个子元素将被拉伸充满容器：

```xaml
<DockPanel LastChildFill="True"
```

子元素定义的顺序显著地影响布局。

## 嵌套布局容器

创造一个标准对话框，在右下角落带有一个OK和Cancel按钮，剩余部分是一个大的内容区域。

[![简单对话框](http://images.cnitblog.com/blog/128579/201306/26174238-4e637670d24e452985bbc55cbc3759c2.png)](http://images.cnitblog.com/blog/128579/201306/26174237-33c3f45cd6b6470bba5765e2cb879d3d.png)

这个对话框将用几种不同的方式实现。这是用嵌套容器方法的标记：

```xaml
<DockPanel LastChildFill="True">
  <StackPanel DockPanel.Dock="Bottom" HorizontalAlignment="Right"
   Orientation="Horizontal">
    <Button Margin="10,10,2,10" Padding="3">OK</Button>
    <Button Margin="2,10,10,10" Padding="3">Cancel</Button>
  </StackPanel>
  <TextBox DockPanel.Dock="Top" Margin="10">This is a test.</TextBox>
</DockPanel>
```

## Grid

每个单元格通常只放一个元素。当然，这个元素可以是另一个布局容器。

创造一个基于网格的布局是一个两步过程。首先，你选择列数和行数。其次你分配合适的行和列到每个包含元素，这样放置它在恰好正确地点。

提示：`Grid`的`ShowGridLines`为真表示在设计时显示网格线。

使用`Grid`需要先定义行和列，例如，这是一个两行、三列的网格：

```xaml
<Grid ShowGridLines="True">
  <Grid.RowDefinitions>
    <RowDefinition></RowDefinition>
    <RowDefinition></RowDefinition>
  </Grid.RowDefinitions>
  <Grid.ColumnDefinitions>
    <ColumnDefinition></ColumnDefinition>
    <ColumnDefinition></ColumnDefinition>
    <ColumnDefinition></ColumnDefinition>
  </Grid.ColumnDefinitions>
  ...
</Grid>
```

在这个例子中，没有为行和列定义提供信息。网格控件均分行和列。每个单元格的尺寸相同。

为放置元素到一个单元格内，你使用`Row`和`Column`附加属性。这些属性都取基于零的索引编号。例如：

```xaml
<Grid ShowGridLines="True">
  ...
  <Button Grid.Row="0" Grid.Column="0">Top Left</Button>
  <Button Grid.Row="0" Grid.Column="1">Middle Left</Button>
  <Button Grid.Row="1" Grid.Column="2">Bottom Right</Button>
  <Button Grid.Row="1" Grid.Column="1">Bottom Middle</Button>
</Grid>
```

每个元素必须被显式地放置到它的单元格内。这允许你放置一个以上元素到同一个单元格内，或留下某些单元格为空白。它也意味着你能不按顺序声明你的元素，就像在这个例子中最后二个按钮。

如果控件位于`Grid`的第一行，可以不指定`Grid.Row`。如果控件位于`Grid`的第一列，可以不指定`Grid.Column`。

注意：如果网格有一个以上行和列，你必须使用`RowDefinition`和`ColumnDefinition`对象显式地定义你的行和列。

### 精确调整行和列

`Grid`支持三种尺寸策略：

绝对尺寸：

```xaml
<ColumnDefinition Width="100"></ColumnDefinition>
```

自动尺寸，是可以容纳内容的最小尺寸：

```xaml
<ColumnDefinition Width="Auto"></ColumnDefinition>
```

比例尺寸，一个星号代表一份：

```xaml
<ColumnDefinition Width="*"></ColumnDefinition>
```

如果你混合使用成比例尺寸模式和其他尺寸模式，成比例尺寸行或列获得剩余空间。

星号之前放置一个数字代表权重，可以解读为份数，下例表示第二行是第一行的2倍高：

```xaml
<RowDefinition Height="*"></RowDefinition>
<RowDefinition Height="2*"></RowDefinition>
```

注意：以编程方式与`ColumnDefinition`和`RowDefinition`对象互相作用是容易的。你只需知道`Width`和`Height`属性是`GridLength`对象。为使`GridLength`代表一个指定的尺寸，只要传递合适的值到`GridLength`构造函数。为使`GridLength`代表一个成比例的（`*`）尺寸，传递数目到`GridLength`构造函数，并传递`GridUnitType.Star`作为构造函数的第二个参数。为指明自动的尺寸，使用静态属性`GridLength.Auto`。

使用这些尺寸模式，简单对话框的例子：

```xaml
<Grid ShowGridLines="True">
  <Grid.RowDefinitions>
    <RowDefinition Height="*"></RowDefinition>
    <RowDefinition Height="Auto"></RowDefinition>
  </Grid.RowDefinitions>
  <TextBox Margin="10" Grid.Row="0">This is a test.</TextBox>
  <StackPanel Grid.Row="1" HorizontalAlignment="Right" Orientation="Horizontal">
    <Button Margin="10,10,2,10" Padding="3">OK</Button>
    <Button Margin="2,10,10,10" Padding="3">Cancel</Button>
  </StackPanel>
</Grid>
```

提示：`Grid`没有声明任何列。这是一个快捷方式，表示你的网格只使用一列，并且那列是成比例地尺寸（因而它充满网格的全宽度）。

### 跨行和跨列

你也能使用二个附加属性使一个元素拉伸占据几个单元格：`RowSpan`和`ColumnSpan`。这两个属性取元素要占用的行或列数。

这个按钮将占据第一行的第一和第二单元格的所有可用空间

```xaml
<Button Grid.Row="0" Grid.Column="0" Grid.RowSpan="2">Span Button</Button>
```

这个按钮将占据跨二列和二行合计四单元格

```xaml
<Button Grid.Row="0" Grid.Column="0" Grid.RowSpan="2" Grid.ColumnSpan="2">
  Span Button</Button>
```

使用跨列重写前面简单对话框的例子：

```xaml
<Grid ShowGridLines="True">
  <Grid.RowDefinitions>
    <RowDefinition Height="*"></RowDefinition>
    <RowDefinition Height="Auto"></RowDefinition>
  </Grid.RowDefinitions>
  <Grid.ColumnDefinitions>
    <ColumnDefinition Width="*"></ColumnDefinition>
    <ColumnDefinition Width="Auto"></ColumnDefinition>
    <ColumnDefinition Width="Auto"></ColumnDefinition>
  </Grid.ColumnDefinitions>

  <TextBox Margin="10" Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="3">
    This is a test.</TextBox>
  <Button Margin="10,10,2,10" Padding="3"
    Grid.Row="1" Grid.Column="1">OK</Button>
  <Button Margin="2,10,10,10" Padding="3"
    Grid.Row="1" Grid.Column="2">Cancel</Button>
</Grid>
```

注意，按钮的地址为合并单元格左上角单元格的地址。

### 分割窗口

定义分隔栏应该遵循几个法则：

- 保留一个专用于`GridSplitter`的列或行，设置行`Height`或列`Width`值为`Auto`。
- `GridSplitter`始终修改整行或整列的尺寸(不是单个的单元格)。你应该用`RowSpan`或`ColumnSpan`属性使`GridSplitter`横跨整行或整列而不局限于单个的单元格。
- `GridSplitter`的`Width`设置为固定尺寸如10，长度方向`Alignment`设置为`Stretch`，分隔方向`Alignment`设置为`Center`。

下面是应用这些法则的一个例子，这个例子是一个垂直分隔栏：

```xaml
<Grid> 
    <Grid.RowDefinitions> 
        <RowDefinition></RowDefinition> 
        <RowDefinition></RowDefinition> 
    </Grid.RowDefinitions> 
    <Grid.ColumnDefinitions> 
        <ColumnDefinition MinWidth="100"></ColumnDefinition> 
        <ColumnDefinition Width="Auto"></ColumnDefinition> 
        <ColumnDefinition MinWidth="50"></ColumnDefinition> 
    </Grid.ColumnDefinitions> 
      
    <Button Grid.Row="0" Grid.Column="0" Margin="3">Left</Button> 
    <Button Grid.Row="1" Grid.Column="0" Margin="3">Left</Button> 
    
    <GridSplitter Grid.Row="0" Grid.Column="1" Grid.RowSpan="2" 
                  Width="10" VerticalAlignment="Stretch" HorizontalAlignment="Center" 
                  ShowsPreview="False">             
    </GridSplitter> 
        
    <Button Grid.Row="0" Grid.Column="2" Margin="3">Right</Button> 
    <Button Grid.Row="1" Grid.Column="2" Margin="3">Right</Button> 
 
</Grid>
```

代码中`GridSplitter`出现了`ShowPreview`属性。`GridSplitter`还有没有出现在代码中的`DragIncrement`属性。

