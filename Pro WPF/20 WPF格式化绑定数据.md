# 20 WPF格式化绑定数据

## 数据绑定回顾

见601页。

## 数据转换

### 使用StringFormat属性

StringFormat属性适用于显示数字为文本。

使用Binding对象的StringFormat属性，可以将绑定对象的属性转化为指定格式的文本：

```
<TextBox Margin="5" Grid.Row="2" Grid.Column="1"
 Text="{Binding Path=UnitCost, StringFormat={}{0:C}}">
</TextBox>
```

StrngFormat值开始的成对花括号，是逃跑序列，表式StringFormat值不是标记扩展。

顺便，只有StringFormat值以花括号开头时，才需要{}逃跑序列。

WPF列表控件也支持字符串格式化列表项目。为使用它，你简单地设置列表的ItemStringFormat属性（定义在ItemsControl类）。这是一个例子，列出产品价格：

```
<ListBox Name="lstProducts" DisplayMemberPath="UnitCost" ItemStringFormat="{}{0:C}">
</ListBox>
```

 

### 介绍值转换器

值转换器负责源数据和目标的转换。恰好在源数据被显示之前转换，以及在两路绑定的情况下，恰好在目标值被应用回源数据之前转换。

创造值转换器，需要4步：

1. 创造实现IValueConverter接口的类
2. 添加特性ValueConversion属性到类声明，指定目标数据类型。
3. 实现Convert()方法，此方法改变数据，从原始格式到显示格式。
4. 实现ConvertBack()方法，此方法反向改变值，从显示格式到本地格式。

这是处理价格的完整的值转换器类：

```
[ValueConversion(typeof(decimal), typeof(string))]
public class PriceConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var price = (decimal) value;
        return price.ToString("C", culture);
    }
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var price = value.ToString();
        
        decimal result;
        if (Decimal.TryParse(price, NumberStyles.Any, culture, out result))
        {
            return result;
        }
        return value;
    }
}
```

应用转换器，创建一个PriceConverter实例，将它赋值给绑定对象的转换器属性：

```
<TextBlock Margin="7" Grid.Row="2">Unit Cost:</TextBlock>
<TextBox Margin="5" Grid.Row="2" Grid.Column="1">
  <TextBox.Text>
    <Binding Path="UnitCost">
      <Binding.Converter>
        <local:PriceConverter></local:PriceConverter>
      </Binding.Converter>
    </Binding>
  </TextBox.Text>
</TextBox>
```

在许多情况下，相同的转换器被用于多个绑定。在这种情况下，为每个绑定创造转换器的一个实例毫无意义。而是，在Resources集合创造转换器对象，如下所示：

```
<Window.Resources>
  <local:PriceConverter x:Key="PriceConverter"></local:PriceConverter>
</Window.Resources>
```

然后引用资源：

```
<TextBox Margin="5" Grid.Row="2" Grid.Column="1"
 Text="{Binding Path=UnitCost, Converter={StaticResource PriceConverter}}">
</TextBox>
```

 

### 用值转换器创建对象

值转换器是连接数据类和窗口显示的桥梁。

这是实现文件路径到BitmapImage对象转换的类：

```
public class ImagePathConverter : IValueConverter
{
    private string imageDirectory = Directory.GetCurrentDirectory();

    public string ImageDirectory
    {
        get { return imageDirectory; }
        set { imageDirectory = value; }
    }

    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var imagePath = Path.Combine(ImageDirectory, (string)value);
        return new BitmapImage(new Uri(imagePath));
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotSupportedException("The method or operation is not implemented.");
    }
}
```

为了使用这个转换器，添加它到资源：

```
<Window.Resources>
  <local:ImagePathConverter x:Key="ImagePathConverter"></local:ImagePathConverter>
</Window.Resources>
```

创造一个使用此转换器的绑定表达式：

```
<Image Margin="5" Grid.Row="2" Grid.Column="1" Stretch="None"
 HorizontalAlignment="Left" Source=
 "{Binding Path=ProductImagePath, Converter={StaticResource ImagePathConverter}}">
</Image>
```

注意，Image.Source属性期待一个ImageSource对象，而BitmapImage类派生自ImageSource类。

### 应用条件格式

例如，你希望标记高价的项目，给他们一个不同的背景颜色：

```
public class PriceToBackgroundConverter : IValueConverter
{
    public decimal MinimumPriceToHighlight {get; set;}

    public Brush HighlightBrush{get; set;}

    public Brush DefaultBrush{get; set;}

    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var price = (decimal)value;
        if (price >= MinimumPriceToHighlight)
            return HighlightBrush;
        else
            return DefaultBrush;
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotSupportedException();
    }
}
```

定义资源对象：

```
<local:PriceToBackgroundConverter x:Key="PriceToBackgroundConverter"
  DefaultBrush="{x:Null}" HighlightBrush="Orange" MinimumPriceToHighlight="50">
</local:PriceToBackgroundConverter>
```

在元素上应用值转换器：

```
<Border Background=
 "{Binding Path=UnitCost, Converter={StaticResource PriceToBackgroundConverter}}"
 ... >
```

 

### 估值多个属性

这个过程是不可逆的，也就是只能将多个值合并为一个结果。而不能将结果分解为多个属性。

结合多个属性的第一个方法是使用MultiBinding。下面例子：

```
<TextBlock>
  <TextBlock.Text>
    <MultiBinding StringFormat="{1}, {0}">
      <Binding Path="FirstName"></Binding>
      <Binding Path="LastName"></Binding>
    </MultiBinding>
  </TextBlock.Text>
</TextBlock>
```

第二个方法是使用值转换器，实现的是IMultiValueConverter。

MultiBinding使用源对象的UnitCost和UnitsInStock属性，并且使用一个值转换器结合他们：

```
<TextBlock>Total Stock Value: </TextBlock>
<TextBox>
  <TextBox.Text>
    <MultiBinding Converter="{StaticResource ValueInStockConverter}">
      <Binding Path="UnitCost"></Binding>
      <Binding Path="UnitsInStock"></Binding>
    </MultiBinding>
  </TextBox.Text>
</TextBox>
```

注意Convert()方法的values是一个对象数组，按照他们在MultiBinding中的顺序放置这些值。在前一个示例中，首先出现UnitCost，然后是UnitsInStock。

```
public class ValueInStockConverter : IMultiValueConverter
{
    public object Convert(object[] values, Type targetType, object parameter, CultureInfo culture)
    {
        var unitCost = (decimal)values[0];
        var unitsInStock = (int)values[1];

        return unitCost * unitsInStock;
    }

    public object[] ConvertBack(object value, Type[] targetTypes, object parameter, CultureInfo culture)
    {
        throw new NotSupportedException();
    }
}
```

 

## 列表类控件

所有列表控件的基类是ItemsControl。

[![ItemsControl](http://images.cnitblog.com/blog/128579/201306/13162400-dc9be4a62868465eb4f3e0c22a328876.png)](http://images.cnitblog.com/blog/128579/201306/13162359-b0620de80d734d87aae4b6373a35520c.png)

ItemsControl类的格式化相关的属性

| 名称                       | 描述                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ItemsSource                | 绑定数据源（你希望显示在列表的集合或DataView）。             |
| DisplayMemberPath          | 你希望为每个数据项目显示的属性。对于一个更世故的表示法或为使用属性的一个联合，使用ItemTemplate代替。 |
| ItemStringFormat           | .NET格式字符串，如果设置，将被用于格式化每个项目的文本。通常，这技术被用来转换数字或日期值到一个合适的显示表示法，确切地为Binding.StringFormat属性。 |
| ItemContainerStyle         | 一个样式允许你设置包裹每个项目容器的属性。容器取决于列表的类型（例如，对于ListBox类它是ListBoxItem，对于ComboBox类ComboBoxItem）。当列表被填充时，这些包裹器对象被自动地创造。 |
| ItemContainerStyleSelector | StyleSelector。使用代码为每个列表项目的包裹器选择一个样式。这允许你为不同的列表项目给出不同的样式。你必须自己创造一个自定义StyleSelector。 |
| AlternationCount           | 在你的数据中交替集合的数目。例如，2交替2行样式，3交替3行样式，等等。 |
| ItemTemplate               | 一个模板，从你的绑定对象提取合适的数据，并且排列它到合适的控件联合里。 |
| ItemTemplateSelector       | DataTemplateSelector，使用代码为每个列表项目选择一个模板。这允许你为不同的项目给出不同的模板。你必须自己创造一个自定义DataTemplateSelector类。 |
| ItemsPanel                 | 定义被创造持有列表项目面板。所有的项目包装被添加到这容器。通常，VirtualizingStackPanel带有一个垂直的（自顶到底）方向被使用。 |
| GroupStyle                 | 如果你使用分组，这是一个样式那定义每个组应该如何被格式化。当使用分组时，项目包装（ListBoxItem，ComboBoxItem，等等）被添加在GroupItem包装那表示每个组，和这些组是然后添加到列表。 |
| GroupStyleSelector         | StyleSelector。使用代码为每个组选择一个样式。这允许你为不同的组给不同的样式。你必须自己创造一个自定义StyleSelector。 |

ItemsControl类的下一级是Selector类，它添加了一组属性描述被选择的项目。

Selector类添加的属性包括：SelectedItem，SelectedIndex，SelectedValue，SelectedValuePath。

注意，Selector类不支持多选。ListBox通过SelectionMode和SelectedItems属性支持多选。

## 列表样式

### ItemContainerStyle

如果ItemContainerStyle被设置，当项目被创造时，样式将向下传递到列表控件的每个项目。在列表框控件的情况，每个项目是一个ListBoxItem对象。（在组合框，它是ComboBoxItem，等等。）因而，你用ListBox.ItemContainerStyle属性应用任何样式被用来设置每个ListBoxItem对象的属性。

```
<ListBox Name="lstProducts" Margin="5" DisplayMemberPath="ModelName">
  <ListBox.ItemContainerStyle>
    <Style>
      <Setter Property="ListBoxItem.Background" Value="LightSteelBlue" />
      <Setter Property="ListBoxItem.Margin" Value="5" />
      <Setter Property="ListBoxItem.Padding" Value="5" />
    </Style>
  </ListBox.ItemContainerStyle>
</ListBox>
```

带触发器的样式，样式中的触发器是在满足某个前提条件的情况下，设置控件的属性：

```
<ListBox Name="lstProducts" Margin="5" DisplayMemberPath="ModelName">
  <ListBox.ItemContainerStyle>
    <Style TargetType="{x:Type ListBoxItem}">
      <Setter Property="Background" Value="LightSteelBlue" />
      <Setter Property="Margin" Value="5" />
      <Setter Property="Padding" Value="5" />

      <Style.Triggers>
        <Trigger Property="IsSelected" Value="True">
          <Setter Property="Background" Value="DarkRed" />
          <Setter Property="Foreground" Value="White" />
          <Setter Property="BorderBrush" Value="Black" />
          <Setter Property="BorderThickness" Value="1" />
        </Trigger>
      </Style.Triggers>
    </Style>
  </ListBox.ItemContainerStyle>
</ListBox>
```

 

### 带复选框或单选按钮的列表框

基本技术是修改代表每个列表项目容器的控件模板。不要修改ListBox.Template属性，因为它是列表框的模板。你需要修改ListBoxItem.Template属性。这里是带有单选按钮的模板：

```
<Window.Resources>
  <Style x:Key="RadioButtonListStyle" TargetType="{x:Type ListBox}">
    <Setter Property="ItemContainerStyle">
      <Setter.Value>
        <Style TargetType="{x:Type ListBoxItem}" >
          <Setter Property="Margin" Value="2" />
          <Setter Property="Template">
            <Setter.Value>
              <ControlTemplate TargetType="{x:Type ListBoxItem}">
                <RadioButton Focusable="False"
                 IsChecked="{Binding Path=IsSelected, Mode=TwoWay,
                             RelativeSource={RelativeSource TemplatedParent} }">
                  <ContentPresenter></ContentPresenter>
                </RadioButton>
              </ControlTemplate>
            </Setter.Value>
          </Setter>
        </Style>
      </Setter.Value>
    </Setter>
  </Style>
</Window.Resources>
```

直接设置列表框的样式：

```
<ListBox Style="{StaticResource RadioButtonListStyle}" Name="lstProducts"
 DisplayMemberPath="ModelName">
```

复选框样式的列表框建立方法与单选列表框基本相同。只是允许列表框多选：

```
<Style x:Key="CheckBoxListStyle" TargetType="{x:Type ListBox}">
  <Setter Property="SelectionMode" Value="Multiple"></Setter>
  <Setter Property="ItemContainerStyle">
    <Setter.Value>
      <Style TargetType="{x:Type ListBoxItem}" >
        <Setter Property="Margin" Value="2" />
        <Setter Property="Template">
          <Setter.Value>
            <ControlTemplate TargetType="{x:Type ListBoxItem}">
              <CheckBox Focusable="False"
               IsChecked="{Binding Path=IsSelected, Mode=TwoWay,
                          RelativeSource={RelativeSource TemplatedParent} }">
                <ContentPresenter></ContentPresenter>
              </CheckBox>
            </ControlTemplate>
          </Setter.Value>
        </Setter>
      </Style>
    </Setter.Value>
  </Setter>
</Style>
```

 

### 交替项目样式

AlternationCount是形成一个序列的项目数，在此数目以后样式交替。默认情况下，AlternationCount被设置为0，并且不使用交替格式化。如果你设置AlternationCount为1，列表将在每个项目以后交替，这允许你应用偶奇格式化模式。

给每个ListBoxItem一个AlternationIndex属性，允许你决定它在交替项目的序列如何编号。假设你设置AlternationCount为2，第一ListBoxItem获得一个AlternationIndex的0，第二获得一个AlternationIndex的1，第三获得一个AlternationIndex的0，第四获得一个AlternationIndex的1，等等。

```
<ListBox Name="lstProducts" Margin="5" DisplayMemberPath="ModelName"
 AlternationCount=”2”>
  <ListBox.ItemContainerStyle>
    <Style TargetType="{x:Type ListBoxItem}">
      <Setter Property="Background" Value="LightSteelBlue" />
      <Setter Property="Margin" Value="5" />
      <Setter Property="Padding" Value="5" />
        <Style.Triggers>
          <Trigger Property="ItemsControl.AlternationIndex" Value="1">
            <Setter Property="Background" Value="LightBlue" />
          </Trigger>
          <Trigger Property="IsSelected" Value="True">
            <Setter Property="Background" Value="DarkRed" />
            <Setter Property="Foreground" Value="White" />
            <Setter Property="BorderBrush" Value="Black" />
            <Setter Property="BorderThickness" Value="1" />
          </Trigger>
        </Style.Triggers>
      </Style>
    </ListBox.ItemContainerStyle>
</ListBox>
```

 

### 样式选择器

必须用代码方式实现，从StyleSelector派生一个专用的类，覆盖SelectStyle()方法，选择合适的样式。

这里是一个基本的选择器，二样式选择一个：

```
public class ProductByCategoryStyleSelector : StyleSelector
{
    public override Style SelectStyle(object item, DependencyObject container)
    {
        var product = (Product)item;
        var window = Application.Current.MainWindow;
        if (product.CategoryName == "Travel")
        {
            return (Style)window.FindResource("TravelProductStyle");
        }
        else
        {
            return (Style)window.FindResource("DefaultProductStyle");
        }
    }
}
```

下面代码使用了反射，更为通用的一个样式选择器：

```
public class SingleCriteriaHighlightStyleSelector : StyleSelector
{
    public Style DefaultStyle
    {
        get; set;
    }
    public Style HighlightStyle
    {
        get; set;
    }
    public string PropertyToEvaluate
    {
        get; set;
    }
    public string PropertyValueToHighlight
    {
        get; set;
    }
    public override Style SelectStyle(object item, DependencyObject container)
    {
        Product product = (Product)item;
        
        // 使用反射获得要检测的属性
        Type type = product.GetType();
        PropertyInfo property = type.GetProperty(PropertyToEvaluate);
        
        // 根据属性值决定是否这个产品应该被高亮
        if (property.GetValue(product, null).ToString() == PropertyValueToHighlight)
        {
            return HighlightStyle;
        }
        else
        {
            return DefaultStyle;
        }
    }
}
```

定义代码中用到的两个样式：

```
<Window.Resources>
  <Style x:Key="DefaultStyle" TargetType="{x:Type ListBoxItem}">
    <Setter Property="Background" Value="LightYellow" />
    <Setter Property="Padding" Value="2" />
  </Style>

  <Style x:Key="HighlightStyle" TargetType="{x:Type ListBoxItem}">
    <Setter Property="Background" Value="LightSteelBlue" />
    <Setter Property="FontWeight" Value="Bold" />
    <Setter Property="Padding" Value="2" />
  </Style>
</Window.Resources>
```

内联使用样式选择器：

```
<ListBox Name="lstProducts" HorizontalContentAlignment="Stretch">
  <ListBox.ItemContainerStyleSelector>
    <local:SingleCriteriaHighlightStyleSelector
      DefaultStyle="{StaticResource DefaultStyle}"
      HighlightStyle="{StaticResource HighlightStyle}"
      PropertyToEvaluate="CategoryName"
      PropertyValueToHighlight="Travel"
    >
    </local:SingleCriteriaHighlightStyleSelector>
  </ListBox.ItemContainerStyleSelector>
</ListBox>
```

样式选择过程，当第一次绑定列表时，被执行一次。如果，由于修改了决定样式的数据，样式不会自动改变。只能用暴力的方法强制样式更新：

```
var selector = lstProducts.ItemContainerStyleSelector;
lstProducts.ItemContainerStyleSelector = null;
lstProducts.ItemContainerStyleSelector = selector;
```

 

## 数据模板

每个ListBoxItem只能绑定一个字段，没有办法结合多个字段。如果要显示多个字段，这就需要用到数据模板。数据模板是一块XAML标记。它定义了应该如何显示一个绑定数据对象。二类型的控件支持数据模板：

- 内容控件通过ContentTemplate属性支持数据模板。内容模板被用来显示Content属性。
- 列表控件通过ItemTemplate属性支持数据模板。这模板被用来显示集合每个项目。集合通过ItemsSource提供。

列表项目是一个内容控件。

数据模板应该包括数据绑定表达式。

这是一个例子，用一个导圆角的边界包裹每个项目，显示二片信息，并且使用粗体的格式化高亮模型数：

```
<ListBox Name="lstProducts" HorizontalContentAlignment="Stretch">
  <ListBox.ItemTemplate>
    <DataTemplate>
      <Border Margin="5" BorderThickness="1" BorderBrush="SteelBlue" CornerRadius="4">
        <Grid Margin="3">
          <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition></RowDefinition>
          </Grid.RowDefinitions>

          <TextBlock FontWeight="Bold"
           Text="{Binding Path=ModelNumber}"></TextBlock>
          <TextBlock Grid.Row="1"
           Text="{Binding Path=ModelName}"></TextBlock>
        </Grid>
      </Border>
    </DataTemplate>
  </ListBox.ItemTemplate>
</ListBox>
```

 

### 分离和重用模板

提取前一个例子的模板：

```
<Window.Resources>
  <DataTemplate x:Key="ProductDataTemplate">
    <Border Margin="5" BorderThickness="1" BorderBrush="SteelBlue"
     CornerRadius="4">
      <Grid Margin="3">
        <Grid.RowDefinitions>
          <RowDefinition></RowDefinition>
          <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>

        <TextBlock FontWeight="Bold" Text="{Binding Path=ModelNumber}">
        </TextBlock>
        <TextBlock Grid.Row="1" Text="{Binding Path=ModelName}">
        </TextBlock>
      </Grid>
    </Border>
  </DataTemplate>
</Window.Resources>
```

现在，你能添加你的数据模板到列表，使用一个静态资源：

```
<ListBox Name="lstProducts" HorizontalContentAlignment="Stretch"
 ItemTemplate="{StaticResource ProductDataTemplate}"></ListBox>
```

自动地在不同的类型的控件重用同一个数据模板。设置DataTemplate.DataType属性为绑定数据的类型。例如，前一个例子，移除Key并指定绑定Product对象：

```
<Window.Resources>
  <DataTemplate DataType="{x:Type local:Product}">
  </DataTemplate>
</Window.Resources>
```

 

### 使用更高级的模板

第一个例子，

```
<Window.Resources>
  <local:ImagePathConverter x:Key="ImagePathConverter"></local:ImagePathConverter>
  <DataTemplate x:Key="ProductTemplate">
    <Border Margin="5" BorderThickness="1" BorderBrush="SteelBlue"
     CornerRadius="4">
      <Grid Margin="3">
        <Grid.RowDefinitions>
          <RowDefinition></RowDefinition>
          <RowDefinition></RowDefinition>
          <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>

        <TextBlock FontWeight="Bold" Text="{Binding Path=ModelNumber}"></TextBlock>
        <TextBlock Grid.Row="1" Text="{Binding Path=ModelName}"></TextBlock>
        <Image Grid.Row="2" Grid.RowSpan="2" Source=
"{Binding Path=ProductImagePath, Converter={StaticResource ImagePathConverter}}">
        </Image>
      </Grid>
    </Border>
  </DataTemplate>
</Window.Resources>
```

直接一个模板内部放置控件。例如，一列种类。挨着每个种类是一个View按钮。你能使用按钮发射另一个窗口恰好匹配在那种类产品。

处理按钮点击。明显地，所有的按钮将被链接到同样的事件处理器，你在模板内部定义它。但是，你需要决定哪一个列表项目被点击。一个解决方案是存储一些额外的识别信息在按钮的Tag属性，如下所示：

```
<DataTemplate>
  <Grid Margin="3">
    <Grid.ColumnDefinitions>
      <ColumnDefinition></ColumnDefinition>
      <ColumnDefinition Width="Auto"></ColumnDefinition>
    </Grid.ColumnDefinitions>
    <TextBlock Text="{Binding Path=CategoryName}"></TextBlock>
    <Button Grid.Column="2" HorizontalAlignment="Right" Padding="2"
      Click="cmdView_Clicked" Tag="{Binding Path=CategoryID}">View ...</Button>
  </Grid>
</DataTemplate>
```

你能取回Tag属性，在事件处理器中：

```
private void cmdView_Clicked(object sender, RoutedEventArgs e)
{
    var cmd = (Button)sender;
    var categoryID = (int)cmd.Tag;
    ...
}
```

当你定义绑定时，遗漏Path属性，你能抓取整个数据对象：

```
<Button HorizontalAlignment="Right" Padding="1"
  Click="cmdView_Clicked" Tag="{Binding}">View ...</Button>
```

传递整个的对象使更新列表选择更容易。当点击View按钮之前，移动选择到按钮被点击的列表项目。如下所示：

```
var cmd = (Button)sender;
var product = (Product)cmd.Tag;
lstCategories.SelectedItem = product;
```

 

### 不同的模板

以不同的方式呈现列表项目：

- 数据触发器
- 值转换器
- 模板选择器

数据触发器。基于数据项目的一个属性，设置模板元素的一个属性。例如，基于相应产品对象的CategoryName属性，你能改变包裹每个列表项目的自定义边界的背景。这是一个例子，用红字高亮在工具目录中的产品：

```
<DataTemplate x:Key="DefaultTemplate">
  <DataTemplate.Triggers>
    <DataTrigger Binding="{Binding Path=CategoryName}" Value="Tools">
     <Setter Property="ListBoxItem.Foreground" Value="Red"></Setter>
    </DataTrigger>
  </DataTemplate.Triggers>
  <Border Margin="5" BorderThickness="1" BorderBrush="SteelBlue"
   CornerRadius="4">
    <Grid Margin="3">
      <Grid.RowDefinitions>
        <RowDefinition></RowDefinition>
        <RowDefinition></RowDefinition>
      </Grid.RowDefinitions>
      <TextBlock FontWeight="Bold"
       Text="{Binding Path=ModelNumber}"></TextBlock>
      <TextBlock Grid.Row="1"
       Text="{Binding Path=ModelName}"></TextBlock>
    </Grid>
  </Border>
</DataTemplate>
```

值转换器方法：

```
<Border Margin="5" BorderThickness="1" BorderBrush="SteelBlue" CornerRadius="4"
 Background=
 "{Binding Path=CategoryName, Converter={StaticResource CategoryToColorConverter}">
```

 

### 模板选择器

你需要创造一个派生自DataTemplateSelector类。检查绑定对象并且使用你提供逻辑选择一个合适的模板。

```
public class SingleCriteriaHighlightTemplateSelector : DataTemplateSelector
{
    public DataTemplate DefaultTemplate
    {
        get; set;
    }
    public DataTemplate HighlightTemplate
    {
        get; set;
    }
    public string PropertyToEvaluate
    {
        get; set;
    }
    public string PropertyValueToHighlight
    {
        get; set;
    }
    public override DataTemplate SelectTemplate(object item,
      DependencyObject container)
    {
        Product product = (Product)item;
        // Use reflection to get the property to check.
        Type type = product.GetType();
        PropertyInfo property = type.GetProperty(PropertyToEvaluate);
        // Decide if this product should be highlighted
        // based on the property value.
        if (property.GetValue(product, null).ToString() == PropertyValueToHighlight)
        {
            return HighlightTemplate;
        }
        else
        {
            return DefaultTemplate;
        }
    }
}
```

 

这里是创建两个数据模板的标记：

```
<Window.Resources>
  <DataTemplate x:Key="DefaultTemplate">
    <Border Margin="5" BorderThickness="1" BorderBrush="SteelBlue"
      CornerRadius="4">
      <Grid Margin="3">
        <Grid.RowDefinitions>
          <RowDefinition></RowDefinition>
          <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>
        <TextBlock
         Text="{Binding Path=ModelNumber}"></TextBlock>
        <TextBlock Grid.Row="1"
         Text="{Binding Path=ModelName}"></TextBlock>
      </Grid>
    </Border>
  </DataTemplate>
  
  <DataTemplate x:Key="HighlightTemplate">
    <Border Margin="5" BorderThickness="1" BorderBrush="SteelBlue"
     Background="LightYellow" CornerRadius="4">
      <Grid Margin="3">
        <Grid.RowDefinitions>    
          <RowDefinition></RowDefinition>
          <RowDefinition></RowDefinition>
          <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>
        <TextBlock FontWeight="Bold"
         Text="{Binding Path=ModelNumber}"></TextBlock>
        <TextBlock Grid.Row="1" FontWeight="Bold"
         Text="{Binding Path=ModelName}"></TextBlock>
        <TextBlock Grid.Row="2" FontStyle="Italic" HorizontalAlignment="Right">
         *** Great for vacations ***</TextBlock>
      </Grid>
    </Border>
  </DataTemplate>
</Window.Resources>    
```

这里是应用模板选择器的标记：

```
<ListBox Name="lstProducts" HorizontalContentAlignment="Stretch">
  <ListBox.ItemTemplateSelector>
    <local:SingleCriteriaHighlightTemplateSelector
      DefaultTemplate="{StaticResource DefaultTemplate}"
      HighlightTemplate="{StaticResource HighlightTemplate}"
      PropertyToEvaluate="CategoryName"
      PropertyValueToHighlight="Travel"
    >
    </local:SingleCriteriaHighlightTemplateSelector>
  </ListBox.ItemTemplateSelector>
</ListBox>
```

###  模板与选择

例1，修改选中项目的背景色：

Foreground属性使用属性继承，所以你添加到模板任何元素自动地获得白颜色。Background属性不使用属性继承，但是默认背景颜色是透明。

首先，设置项目容器的样式：

```
<ListBox Name="lstProducts" HorizontalContentAlignment="Stretch">
  <ListBox.ItemContainerStyle>
    <Style>
      <Setter Property="Control.Padding" Value="0"></Setter>
      <Style.Triggers>
        <Trigger Property="ListBoxItem.IsSelected" Value="True">
          <Setter Property="ListBoxItem.Background" Value="DarkRed" />
        </Trigger>
      </Style.Triggers>
    </Style>
  </ListBox.ItemContainerStyle>
</ListBox>
```

然后，修改数据模板：

```
<DataTemplate>
  <Grid Margin="0" Background="White">
    <Border Margin="5" BorderThickness="1"
     BorderBrush="SteelBlue" CornerRadius="4"
     Background="{Binding Path=Background, RelativeSource={
                             RelativeSource
                             Mode=FindAncestor,
                             AncestorType={x:Type ListBoxItem}
                          }}" >
      <Grid Margin="3">
        <Grid.RowDefinitions>
          <RowDefinition></RowDefinition>
          <RowDefinition></RowDefinition>
        </Grid.RowDefinitions>
        <TextBlock FontWeight="Bold" Text="{Binding Path=ModelNumber}"></TextBlock>
        <TextBlock Grid.Row="1" Text="{Binding Path=ModelName}"></TextBlock>
      </Grid>
    </Border>
  </Grid>
</DataTemplate>
```

例2，选中项目后，显示此项目的细节：

在数据模板中，使用Binding的RelativeSource属性搜索目前的ListBoxItem。如果它没有被选择，设置额外信息的Visibility属性，你能隐藏它。

使用一个数据触发器，当ListBoxItem的IsSelected属性被改变，修改容器的Visibility属性。

数据触发器只需要放在要隐藏的容器内。

这是还没有自动扩展功能的简化版：

```
<DataTemplate>
  <Border Margin="5" BorderThickness="1" BorderBrush="SteelBlue"
   CornerRadius="4">
    <StackPanel Margin="3">
      <TextBlock Text="{Binding Path=ModelName}"></TextBlock>
      <StackPanel>
        <TextBlock Margin="3" Text="{Binding Path=Description}"
         TextWrapping="Wrap" MaxWidth="250" HorizontalAlignment="Left"></TextBlock>
        <Image Source="{Binding Path=ProductImagePath, Converter={StaticResource ImagePathConverter}}">
        </Image>
        <Button FontWeight="Regular" HorizontalAlignment="Right" Padding="1"
         Tag="{Binding}">View Details...</Button>
      </StackPanel>
    </StackPanel>
  </Border>
</DataTemplate>
```

最内层的StackPanel包含着只对选中项目显示的内容，所以设置这个容器的样式：

```
<StackPanel>
  <StackPanel.Style>
    <Style>
      <Style.Triggers>
        <DataTrigger
          Binding="{Binding Path=IsSelected, RelativeSource={
                             RelativeSource
                             Mode=FindAncestor,
                             AncestorType={x:Type ListBoxItem}
                          }}"
          Value="False">
          <Setter Property="StackPanel.Visibility" Value="Collapsed" />
        </DataTrigger>
      </Style.Triggers>
    </Style>
  </StackPanel.Style>
</StackPanel>
```

在这个例子中，你需要使用DataTrigger而不是一个普通的触发器，因为你需要的属性在祖先元素中（ListBoxItem），并且唯一访问它的办法是使用数据绑定表达式。

### 改变项目布局

使用任何从System.Windows.Controls.Panel派生的类，设置ListBox的ItemsPanelTemplate属性，可以改变项目布局。

下面例子，使用WrapPanel改变项目布局：

```
<ListBox Margin="7,3,7,10" Name="lstProducts"
 ItemTemplate="{StaticResource ItemTemplate}"
 ScrollViewer.HorizontalScrollBarVisibility="Disabled">
  <ListBox.ItemsPanel>
    <ItemsPanelTemplate>
      <WrapPanel></WrapPanel>
    </ItemsPanelTemplate>
  </ListBox.ItemsPanel>
</ListBox>
```

你必须也设置ScrollViewer.HorizontalScrollBarVisibility附加属性为Disabled。这确保ScrollViewer从不使用水平的scrollbar。

VirtualizingStackPanel对于大量数据项目有更好的性能。

## 组合框

默认，ComboBox是只读的。当设置IsReadOnly属性为假并且IsEditable属性为真，选择框变成文本框，你能键入任何文本。

组合框有自动完成的功能。要关闭它，设置ComboBox.IsTextSearchEnabled属性为假。此属性位于ItemsControl类。

如果IsEditable属性是假（这是默认值），选择框将显示一个项目的精确视觉副本。

重要的细节是组合框显示的内容是什么，而不是它的数据源是什么。例如，用Product对象填充一个组合框控件，并且设置DisplayMemberPath属性为ModelName。如此，组合框显示每个项目的ModelName属性。即组合框从一组Product对象获取信息，你的标记创造的是一个普通的文本列表。它将显示当前产品的ModelName，并且如果IsEditable是真并且IsReadOnly是假，它将允许你编辑那值。

如果IsEditable属性是真，选择框显示一个它的一个逐字的表示法。就是简单地调用ToString()到项目。一般情况下，显示一个类名的全称。

纠正这个问题最简单的方法是，设置TextSearch.TextPath附着属性指明组合框应该使用的内容。

```
<ComboBox IsEditable="True" IsReadOnly="True" TextSearch.TextPath="ModelName" ...>
```

 