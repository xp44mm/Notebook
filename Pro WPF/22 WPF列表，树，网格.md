# 22 WPF列表，树，网格

## ListView

ListView从ListBox派生，只增加了View属性。如果你没有设置View属性，ListView行为正如ListBox。

从技术上，View属性指向任何ViewBase派生类的一个实例。ViewBase类是简单的；事实上，它是两个样式。一样式应用于ListView控件（DefaultStyleKey属性），和另一样式应用于ListView项目（ItemContainerDefaultStyleKey属性）。DefaultStyleKey和ItemContainerDefaultStyleKey属性实际上没有提供样式；代替，他们返回一个指向样式的ResourceKey对象。

### 用GridView创造多列

GridView是一个类。派生自ViewBase。代表带有多个列的一个列表视图。你定义那些列依靠添加GridViewColumn对象到GridView.Columns集合。

GridView和GridViewColumn都提供几个有用的方法。你能用于自定义列表的外观。为创造最简单的列表，你需要为每个GridViewColumn设置二个属性：Header和DisplayMemberBinding。Header属性提供被放置在列顶的文本。DisplayMemberBinding属性包含一个绑定，从每个数据项目提取你希望显示的信息。

这是标记，定义了三列被用在这个例子中。

```
<ListView Margin="5" Name="lstProducts">
  <ListView.View>
    <GridView>
      <GridView.Columns>
        <GridViewColumn Header="Name"
         DisplayMemberBinding="{Binding Path=ModelName}" />
        <GridViewColumn Header="Model"
         DisplayMemberBinding="{Binding Path=ModelNumber}" />
        <GridViewColumn Header="Price" DisplayMemberBinding=
"{Binding Path=UnitCost, StringFormat={}{0:C}}" />
      </GridView.Columns>
    </GridView>
  </ListView.View>
</ListView>
```

没有一列有硬编码的尺寸。

DisplayMemberBinding属性使用一个绑定表达式。

#### 改变列尺寸

当声明列时，可以指定列宽：

```
<GridViewColumn Width="300" ... />
```

这只是设定了列的最初尺寸，没有阻止用户修改列尺寸。

#### 使用单元格模板

CellTemplate属性取一个数据模板。它只应用于一列。每列可以拥有它自己的数据模板。

定义一个可以换行的列：

```
<GridViewColumn Header="Description" Width="300">
  <GridViewColumn.CellTemplate>
    <DataTemplate>
      <TextBlock Text="{Binding Path=Description}" TextWrapping="Wrap"></TextBlock>
    </DataTemplate>
  </GridViewColumn.CellTemplate>
</GridViewColumn>
```

注意，为了换行有效果，你需要约束列的宽度。

因为数据模板使用了数据绑定，所以只能为每个字段创建一个单独的模板。

下面的列使用一个数据模板显示一个图像。

```
<GridViewColumn Header="Picture" >
  <GridViewColumn.CellTemplate>
    <DataTemplate>
      <Image Source=
"{Binding Path=ProductImagePath,Converter={StaticResource ImagePathConverter}}">
      </Image>
    </DataTemplate>
  </GridViewColumn.CellTemplate>
</GridViewColumn>
```

当创造一个数据模板，你选择内联定义它或引用一个在别处定义的资源。因为列模板不能被重用于不同的字段，最清晰的通常是内联定义他们。

你能使用模板选择器，设置GridViewColumn.CellTemplateSelector属性。

自定义列标头

### 创造一个自定义视图

见670页。

## 树视图

WPF的树视图充分支持数据绑定。

树视图是一个特殊的ItemsControl，容纳TreeViewItem对象。但是不同于ListViewItem，TreeViewItem不是一个内容控件。代替，每个TreeViewItem是一个独立的ItemsControl，能够持有更多TreeViewItem对象。

从技术上，TreeViewItem派生自HeaderedItemsControl，这派生自ItemsControl。HeaderedItemsControl类添加一个Header属性，这持有你希望显示的树项目内容（通常文本）。WPF包含二另外的HeaderedItemsControl类：MenuItem和ToolBar。

用标记建立一个树：

```
<TreeView>
  <TreeViewItem Header="Fruit">
    <TreeViewItem Header="Orange"/>
    <TreeViewItem Header="Banana"/>
    <TreeViewItem Header="Grapefruit"/>
  </TreeViewItem>
 <TreeViewItem Header="Vegetables">
    <TreeViewItem Header="Aubergine"/>
    <TreeViewItem Header="Squash"/>
    <TreeViewItem Header="Spinach"/>
  </TreeViewItem>
</TreeView> 
```

树视图可以包括非TreeViewItem对象。Header属性是一个内容，可以包括一个其他元素。也可以通过HeaderTemplate，或HeaderTemplateSelector属性。

### 创造一个数据绑定树视图

你只是需要指定正确的数据模板。你的模板指明不同水平的数据之间的关系。

首先，定义Category类，Category类暴露Product对象的一个集合：

```
public class Category : INotifyPropertyChanged
{
    private string categoryName;
    public string CategoryName
    {
        get { return categoryName; }
        set { categoryName = value;
              OnPropertyChanged(new PropertyChangedEventArgs("CategoryName"));
            }
    }
    private ObservableCollection<Product> products;
    public ObservableCollection<Product> Products
    {
        get { return products; }
        set { products = value;
              OnPropertyChanged(new PropertyChangedEventArgs("Products"));
        }
    }
    public event PropertyChangedEventHandler PropertyChanged;
    public void OnPropertyChanged(PropertyChangedEventArgs e)
    {
        if (PropertyChanged != null)
            PropertyChanged(this, e);
    }
    public Category(string categoryName, ObservableCollection<Product> products)
    {
        CategoryName = categoryName;
        Products = products;
    }
}
```

创造一个集合通过一个属性暴露另一个集合是用WPF数据绑定父子关系导航的关键。

为显示目录，你需要提供一个TreeView.ItemTemplate，能处理绑定对象。在这个例子中，你需要显示每个Category对象的CategoryName属性。这里是数据模板：

```
<TreeView Name="treeCategories" Margin="5">
  <TreeView.ItemTemplate>
    <HierarchicalDataTemplate>
      <TextBlock Text="{Binding Path=CategoryName}" />
    </HierarchicalDataTemplate>
  </TreeView.ItemTemplate>
</TreeView>
```

使用一个HierarchicalDataTemplate对象而不是一个DataTemplate设置TreeView.ItemTemplate。HierarchicalDataTemplate能包裹第二层模板。然后，HierarchicalDataTemplate能从第一层提取一个项目集合并且提供集合到第二层模板。你只设置ItemsSource属性去识别有子项目的属性，以及设置ItemTemplate属性指明每个对象应该如何被格式化。

这是修改后的代码：

```
<TreeView Name="treeCategories" Margin="5">
  <TreeView.ItemTemplate>
    <HierarchicalDataTemplate ItemsSource="{Binding Path=Products}">
      <TextBlock Text="{Binding Path=CategoryName}" />
      <HierarchicalDataTemplate.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding Path=ModelName}" />
        </DataTemplate>
      </HierarchicalDataTemplate.ItemTemplate>
    </HierarchicalDataTemplate>
  </TreeView.ItemTemplate>
</TreeView>
```

本质上，你现在有二模板，一对于树的每个水平。第二模板从第一模板使用被选择项目作为它的数据源。

尽管这标记工作完美，普遍的是提取每个数据模板和应用它到你的数据对象依靠数据类型而不是依靠位置。为理解那意味着什么，有帮助的是考虑一个修订版本的标记对于数据绑定树视图：

```
<Window x:Class="DataBinding.BoundTreeView" ...
    xmlns:local="clr-namespace:DataBinding">
    
  <Window.Resources>
    <HierarchicalDataTemplate DataType="{x:Type local:Category}"
     ItemsSource="{Binding Path=Products}">
      <TextBlock Text="{Binding Path=CategoryName}"/>
    </HierarchicalDataTemplate>
    <HierarchicalDataTemplate DataType="{x:Type local:Product}">
      <TextBlock Text="{Binding Path=ModelName}" />
    </HierarchicalDataTemplate>
  </Window.Resources>
  
  <Grid>
    <TreeView Name="treeCategories" Margin="5">
    </TreeView>
  </Grid>
</Window>
```

在这个例子中树视图没有显式地设置它的ItemTemplate。代替，基于绑定对象的数据类型合适的ItemTemplate被使用。类似地，Category模板没有指定ItemTemplate那应该被用于处理Products集合。它也依靠数据类型被自动地选择。这树现在能显示一列产品或一列目录那包含组的产品。

## DataGrid

顾名思义，DataGrid是一个显示数据控件，从对象的一个集合取信息和呈现它在一个行和列的网格。每个行对应于一个独立的对象，以及每个列对应于那对象一个属性。

DataGrid选择模型允许你选择是否用户能选择一个行，多个行，或一些单元格的联合。

为创造一个DataGrid，你能使用自动的列生成。你需要设置AutoGenerateColumns属性为真（这是默认值）：

```
<DataGrid x:Name="gridProducts" AutoGenerateColumns="True">
</DataGrid>
```

现在你能填充DataGrid，像填充列表控件一样，依靠设置ItemsSource属性：

```
gridProducts.ItemsSource = products;
```

对于自动的列生成，DataGrid使用反射在绑定数据对象中查找每个公开的属性。它为每个属性创造一个列。

为显示非字符串属性，DataGrid调用ToString()。对于数字，日期，和另外的简单的数据类型工作良好。

DataGrid的基本显示属性：

| 名字                                                        | 描述 |
| ----------------------------------------------------------- | ---- |
| RowBackground 和 AlternatingRowBackground                   |      |
| ColumnHeaderHeight                                          |      |
| RowHeaderWidth                                              |      |
| ColumnWidth                                                 |      |
| RowHeight                                                   |      |
| GridLinesVisibility                                         |      |
| VerticalGridLinesBrush                                      |      |
| HorizontalGridLinesBrush                                    |      |
| HeadersVisibility                                           |      |
| HorizontalScrollBarVisibility和 VerticalScrollBarVisibility |      |

### 重新修改尺寸和重新布置列

为设置ColumnWidth属性，你提供一个DataGridLength对象。你的DataGridLength能指定一个精确的尺寸或一个特殊的尺寸方式。如果你选择使用一个精确的尺寸，只是设置ColumnWidth相等的为合适的数（在XAML）或提供数作为一个构造函数实参当创造DataGridLength（在代码）：

```
grid.ColumnWidth = new DataGridLength(150);
```

你通过DataGridLength类的静态的属性访问特殊的尺寸模式。这是一个例子，使用默认DataGridLength.SizeToHeader尺寸方式，这意味着列被制造足够宽容纳它们的标头文本：

```
grid.ColumnWidth = DataGridLength.SizeToHeader;
```

另一个流行的选项是DataGridLength.SizeToCells。列只能自动加宽，不能自动收缩。

DataGridLength.Auto，每个列的宽度适合最大的单元格或列标头文本。

DataGrid支持使用星号设置宽度。你需要显式地设置每个列对象的Width属性。

修改列尺寸以后，列就失去自动增加列宽的功能。

你能阻止用户修改DataGrid的列尺寸，依靠设置CanUserResizeColumns属性为假。你能阻止用户修改单独列尺寸，依靠设置那列的CanUserResize属性为假。你也能阻止用户使列极端地窄，依靠设置列的MinWidth属性。

如果你不希望用户拖拽列位置，设置DataGrid的CanUserReorderColumns属性或指定的列的CanUserReorder属性为假。

### 定义列

设置AutoGenerateColumns为假。然后显式地定义列，设置列配置，指定列顺序。用正确的列对象填充DataGrid.Columns集合。

目前，DataGrid支持几个类型的列，这表现为派生自DataGridColumn不同的类：

- DataGridTextColumn：这列是大多数数据类型标准选择。值被转变为文本并且显示在一个TextBlock。当你编辑行，TextBlock被替换为一个标准文本框。
- DataGridCheckBoxColumn：这列显示一个复选框。这列类型自动被用于Boolean（或可空Boolean）值。通常，复选框是只读，但是当你编辑行，它变成一个正常的复选框。
- DataGridHyperlinkColumn：这列显示一个可点击的链接。如果协同诸如Frame或NavigationWindow等WPF导航容器使用，它允许用户导航到另一个URI。
- DataGridComboBox：最初这列看起来像一个DataGridTextColumn，但是在编辑模式改变为一个下拉组合框。当你希望约束编辑到几个允许值它是一个好选择。
- DataGridTemplateColumn：这列是最强力的选项。它允许你定义一个数据模板显示列值，带有所有的灵活性和力量你有当使用模板在一个列表控件。例如，你能使用一个DataGridTemplateColumn显示图像数据或使用一个专门的WPF控件（诸如一个下拉列表带有有效的值或一个DatePicker对于日期值）。

例如，这里是一个修订DataGrid那创造一个2列的显示带有产品名字和价格。它也应用更清楚的列标题和放宽产品列适合它的数据：

```
<DataGrid x:Name="gridProducts" Margin="5" AutoGenerateColumns="False">
  <DataGrid.Columns>
    <DataGridTextColumn Header="Product" Width="175"
     Binding="{Binding Path=ModelName}"></DataGridTextColumn>
    <DataGridTextColumn Header="Price"
     Binding="{Binding Path=UnitCost}"></DataGridTextColumn>
  </DataGrid.Columns>
</DataGrid>
```

当你定义一个列，你几乎总是设置三细节：标头文本，列的宽度，和绑定获得数据。

通常，你将使用一个简单字符串设置DataGridColumn.Header属性，但是不局限于普通文本。

DataGridColumn.Width属性支持硬编码的值和几个自动的尺寸模式，正如DataGrid.ColumnWidth属性。唯一不同是DataGridColumn.Width应用于单个的列，然而DataGrid.ColumnWidth设置全体表默认。当DataGridColumn.Width被设置，它覆盖DataGrid.ColumnWidth。

DataGridColumn.Binding属性中的绑定表达式为列提供正确的信息。Binding方法比DisplayMemberPath更灵活，它允许你使用字符串格式化和值转换器，无需使用复杂的列模板。

```
<DataGridTextColumn Header="Price" Binding=
 "{Binding Path=UnitCost, StringFormat={}{0:C}}">
</DataGridTextColumn>
```

列的另外两个属性包括：Visibility，DisplayIndex属性。

#### DataGridCheckBoxColumn

正如DataGridTextColumn，Binding属性提取数据—在这种情况下，真或假值，那被用于设置内部复选框元素的IsChecked属性。DataGridCheckBoxColumn也添加一个属性命名Content那让你显示可选的内容复选框旁边。最后，DataGridCheckBoxColumn包含一个IsThreeState属性，这决定是否复选框支持未定状态也更明显的选中和未选中状态。如果你使用DataGridCheckBoxColumn显示从一个可空Boolean值信息，你能设置IsThreeState属性为真。那样，用户能点击向后未定状态（这显示一个轻阴影复选框）返回绑定值为空。

#### DataGridHyperlinkColumn

见691页。

#### DataGridComboBoxColumn

DataGridComboBoxColumn最初显示普通文本，但是在编辑模式时，允许用户从组合框控件选取一个项目。（事实上，用户将被迫从列表选择，因为组合框不允许直接输入文本。）

为使用DataGridComboBoxColumn，你需要决定如何填充组合框。你只设置DataGridComboBoxColumn.ItemsSource集合。可以在标记中用手填充它。下面的例子添加一列字符串到组合框：

```
<DataGridComboBoxColumn Header="Category"
 SelectedItemBinding="{Binding Path=CategoryName}">
  <DataGridComboBoxColumn.ItemsSource>
    <col:ArrayList>
      <sys:String>General</sys:String> 
      <sys:String>Communications</sys:String>
      <sys:String>Deception</sys:String>
      <sys:String>Munitions</sys:String>
      <sys:String>Protection</sys:String>
      <sys:String>Tools</sys:String>
      <sys:String>Travel</sys:String> 
    </col:ArrayList>
  </DataGridComboBoxColumn.ItemsSource>
</DataGridComboBoxColumn>
```

在许多情况下，你显示在列表值不是你希望存储在数据对象值。

代替一列简单的字符串，你绑定一个整个的Category对象的列表到DataGridComboBoxColumn.ItemsSource属性：

```
categoryColumn.ItemsSource = App.StoreDB.GetCategories();
gridProducts.ItemsSource = App.StoreDB.GetProducts();
```

然后你配置DataGridComboBoxColumn。你必须设置三属性：

```
<DataGridComboBoxColumn Header="Category" x:Name="categoryColumn"  
 DisplayMemberPath="CategoryName" SelectedValuePath="CategoryID"
 SelectedValueBinding="{Binding Path=CategoryID}"></DataGridComboBoxColumn> 
```

DisplayMemberPath告诉列从Category对象抽取哪一个文本并在列表中显示。SelectedValuePath告诉列从Category对象抽取什么数据。SelectedValueBinding说明在Product对象的链接字段。

#### DataGridTemplateColumn

DataGridTemplateColumn使用一个数据模板，这工作方式等同于列表控件数据模板特征。DataGridTemplateColumn在唯一不同是它允许你定义二模板：一对于数据显示（CellTemplate）和一对于数据编辑（CellEditingTemplate）。这是一个例子那使用模板数据在网格列放置每个产品的图像：

```
<DataGridTemplateColumn>
  <DataGridTemplateColumn.CellTemplate>
    <DataTemplate>
      <Image Stretch="None" Source=
       "{Binding Path=ProductImagePath, Converter={StaticResource ImagePathConverter}}">
      </Image>
    </DataTemplate>
  </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>
```

添加ImagePathConverter值转换器到Window.Resources集合：

```
<Window.Resources>
  <local:ImagePathConverter x:Key="ImagePathConverter"></local:ImagePathConverter>
</Window.Resources>
```

### 格式化和样式化列

你能以格式一个TextBlock元素的方式格式一个DataGridTextColumn，依靠设置Foreground，FontFamily，FontSize，FontStyle，和FontWeight属性。但是，DataGridTextColumn没有暴露TextBlock所有的属性。例如，没办法设置Wrapping属性，如果你希望创造一个列显示多行的文本。在这种情况下，你需要使用ElementStyle属性代替。

本质上，ElementStyle属性让你创造一个样式，那被应用于DataGrid单元格的内部元素。在一个简单的DataGridTextColumn的情况，那是一个TextBlock。在一个DataGridCheckBoxColumn，它是一个复选框。在一个DataGridTemplateColumn，它是你在数据模板中创造的元素。

这里是一个简单的样式，那允许在一个列文本换行：

```
<DataGridTextColumn Header="Description" Width="400"
 Binding="{Binding Path=Description}">
  <DataGridTextColumn.ElementStyle>
    <Style TargetType="TextBlock">
      <Setter Property="TextWrapping" Value="Wrap"></Setter>
    </Style>
  </DataGridTextColumn.ElementStyle>
</DataGridTextColumn>
```

不幸地，DataGrid不能像WPF布局容器一样灵活地修改它自己的尺寸。代替，你被迫设置一个固定行高度，依靠使用DataGrid.RowHeight属性。这高度应用于所有的行，不管它包含内容的数量。

你能使用EditingElementStyle设置编辑时的被雇用元素样式。在DataGridTextColumn的情况，编辑元素是TextBox控件。

DataGrid的样式属性

| 属性               | 样式应用到                                                   |
| ------------------ | ------------------------------------------------------------ |
| ColumnHeaderStyle  |                                                              |
| RowHeaderStyle     |                                                              |
| DragIndicatorStyle |                                                              |
| RowStyle           | TextBlock那被用于普通的行（行在没有被明确地通过列的ElementStyle属性自定义的列） |

### 格式化行

在许多情况下，需要标记包含指定的数据的行。你能以编程方式应用这类格式化，依靠处理DataGrid.LoadingRow事件。

LoadingRow事件对于行格式化是一个强力的工具。通过它能访问当前的行的数据对象，执行简单的范围核对，比较，和更复杂的操作。它也提供DataGridRow对象对于行，让你格式行带有不同的颜色或一个不同的字体。但是，你不能格式恰好一个单个的单元格在那行—对于那，你需要DataGridTemplateColumn和一个自定义值转换器。

对于每个行当它出现在屏幕上时，LoadingRow事件激发一次。

```
//因为大数据显示的效率，重用刷子对象
private SolidColorBrush highlightBrush = new SolidColorBrush(Colors.Orange);
private SolidColorBrush normalBrush = new SolidColorBrush(Colors.White);

private void gridProducts_LoadingRow(object sender, DataGridRowEventArgs e)
{
    //从这一行取出数据对象
    var product = (Product)e.Row.DataContext;

    //应用条件格式化
    if (product.UnitCost > 100)
    {
        e.Row.Background = highlightBrush;
    }
    else
    {
        //恢复默认的白背景。这确保被使用的，
        //格式化的DataGrid对象被重设为原始的外观
        e.Row.Background = normalBrush;
    }
}
```

记住，你能使用一个值转换器那检查绑定数据并且转换它到其他的某物。这技术当结合DataGridTemplateColumn尤其强力的。

### 显示行细节

首先你需要定义在行细节区域显示内容，依靠设置DataGrid.RowDetailsTemplate属性。在这种情况下，行细节区域使用一个基本模板，包含一个TextBlock显示完整产品文本和添加一个它周围边界：

```
<DataGrid.RowDetailsTemplate>
  <DataTemplate>
    <Border Margin="10" Padding="10" BorderBrush="SteelBlue" BorderThickness="3"
     CornerRadius="5">
      <TextBlock Text="{Binding Path=Description}" TextWrapping="Wrap"
       FontSize="10">
      </TextBlock>
    </Border>
  </DataTemplate>
</DataGrid.RowDetailsTemplate>
```

你能配置行细节区域的显示行为，依靠设置DataGrid.RowDetailsVisibilityMode属性。默认情况下，这属性被设置为VisibleWhenSelected，这意味着行细节区域被显示当行被选择。可选地，你能设置它为Visible，这意味着将立刻显示每行的细节区域。或你能使用Collapsed，这意味着细节区域将不被显示对于任何行—至少，不直到你改变RowDetailsVisibilityMode在代码（例如，当用户选择一个确信的类型的行）。

### 冻结列

为使用列冻结，你设置DataGrid.FrozenColumnCount属性为一个比0更大的数。例如，一个值1恰好冻结第一列：

```
<DataGrid x:Name="gridProducts" Margin="5" AutoGenerateColumns="False"
 FrozenColumnCount="1">
```

如果你冻结一列，它是最左列；如果你冻结二列，他们将是左边前二列；等等。

### 选择

如同一个普通的列表控件，DataGrid让用户选择单独的项目。你能反应SelectionChanged事件当这发生。为找出哪一个数据对象是目前选择，你能使用SelectedItem属性。如果你希望用户能选择多个行，设置SelectionMode属性为Extended。（Single是唯一另外的选项和默认。）为选择多个行，用户必须按下Shift或Ctrl键。你能从SelectedItems属性取回选择项目的集合。

你能以编程方式设置选择，依靠使用SelectedItem属性。如果你设置的选择项目当前看不到，随后调用DataGrid.ScrollIntoView()方法，滚动表格使被选择项可见。

### 排序

DataGrid内建排序特征只要你绑定的集合实现IList（诸如List<T>和ObservableCollection<T>集合）。

通常，DataGrid使用出现在列中的绑定数据排序算法。但是，你能选择一个不同的属性从绑定数据对象依靠设置一个列的SortMemberPath。和如果你有一个DataGridTemplateColumn，你需要使用SortMemberPath，因为不存在绑定属性提供绑定数据。如果你没有，你的列将不支持排序。

你也能使排序不可用依靠设置CanUserSortColumns属性为假（或对于指定的列关掉它依靠设置列的CanUserSort属性）。

### 编辑

DataGrid的最大便利之一是它对于编辑的支持。一个DataGrid单元格切换到编辑方式当用户双击它。但是DataGrid用几个方法让你限制这编辑能力：

- DataGrid.IsReadOnly
- DataGridColumn.IsReadOnly
- 只读属性：如果你的数据对象有一个没有设置器的属性，DataGrid足够智能注意这细节并且不可用列编辑，就好像你设置DataGridColumn.IsReadOnly为真。类似地，如果你的属性不是一个简单的文本，数字的，或日期类型，DataGrid使它只读。

依赖于列类型，当一个单元格切换到编辑模式，发生什么。DataGridTextColumn显示一个文本框。DataGridCheckBox列显示一个复选框你能选中或不选。但是DataGridTemplateColumn允许你用一个更专门的输入控件替换标准编辑文本框。

例如下面的列显示一个日期。当用户双击去编辑那值，它转到一个带有预选择当前值的下拉DatePicker：

```
<DataGridTemplateColumn Header="Date Added">
  <DataGridTemplateColumn.CellTemplate>
    <DataTemplate>
      <TextBlock Margin="4" Text=
 "{Binding Path=DateAdded, Converter={StaticResource DateOnlyConverter}}">
      </TextBlock>
    </DataTemplate>
  </DataGridTemplateColumn.CellTemplate>
  <DataGridTemplateColumn.CellEditingTemplate>
    <DataTemplate>
      <DatePicker SelectedDate="{Binding Path=DateAdded, Mode=TwoWay}">
      </DatePicker>
    </DataTemplate>
  </DataGridTemplateColumn.CellEditingTemplate>
</DataGridTemplateColumn>
```

DataGrid同样自动地支持基本验证系统。这是一个例子，使用一个自定义验证规则验证UnitCost字段：

```
<DataGridTextColumn Header="Price">
  <DataGridTextColumn.Binding>
    <Binding Path="UnitCost" StringFormat="{}{0:C}">
      <Binding.ValidationRules>
        <local:PositivePriceRule Max="999.99" />
      </Binding.ValidationRules>
    </Binding>
  </DataGridTextColumn.Binding> 
</DataGridTextColumn>
```

DataGrid的编辑事件。

| 名称                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| BeginningEdit        | 发生在单元格准备进入编辑时。你能检查目前被编辑的列和行，检查单元格值，和依靠使用DataGridBeginningEditEventArgs.Cancel属性取消操作。 |
| PreparingCellForEdit | 用于模板列。这时，你能执行任何编辑控件要求的最后初始化。使用DataGridPreparingCellForEditEventArgs.EditingElement访问在CellEditingTemplate中的元素。 |
| CellEditEnding       | 发生在单元格准备退出编辑模式时。DataGridCellEditEndingEventArgs.EditAction告诉你是否用户尝试接受编辑（例如，依靠按压Enter或点击另一个单元格）或取消它（依靠按压Esc键）。你能检查新的数据和设置Cancel属性回滚一个试图的改变。 |
| RowEditEnding        | 发生在用户编辑当前行之后导航到新行。正如CellEditEnding，你能使用这指向执行验证和取消改变。典型地，你将执行牵涉几个列的验证—例如，确保某列的值不比另一列的值更大。 |

如果你需要一个地方执行你页面的验证逻辑，你能写自定义验证逻辑响应CellEditEnding和RowEditEnding事件。检查列规则在CellEditEnding事件处理器，和验证整行的一致性在RowEditEnding事件。并且记住如果你取消一个编辑，你应该提供一个问题的解释（通常在页面别处的一个TextBlock中）。