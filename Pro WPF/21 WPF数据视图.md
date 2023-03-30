# 21 WPF数据视图

## 视图对象

当你绑定集合到ItemsControl，在幕后数据视图被安静地创造。视图位于数据源和绑定控件之间。数据视图是通往数据源的一个窗口。它跟踪当前项目，它支持诸如排序，过滤，和分组特征。这些特征独立于数据对象本身，意味着你能以不同的方式、在窗口的不同部分（或应用的不同部分）绑定相同的数据。例如，你能绑定相同的产品集合到两个不同的列表但是过滤他们显示不同的记录。

视图对象依赖于数据对象的类型。所有的视图派生自CollectionView，但是两个特殊的实现派生自CollectionView：ListCollectionView和BindingListCollectionView。这是它如何工作：

- 如果数据源实现IBindingList，一个BindingListCollectionView被创造。当你绑定一个ADO.NET DataTable时发生。
- 如果数据源没有实现IBindingList但是它实现IList，一个ListCollectionView被创造。当你绑定一个ObservableCollection，如同产品的列表。
- 如果你的数据源没有实现IBindingList或IList但是它实现IEnumerable，你获得一个基本CollectionView。

### 取回一个视图对象

为获得一个目前使用的视图对象，你使用System.Windows.Data.CollectionViewSource类的GetDefaultView()静态方法。当你调用GetDefaultView()，并传递数据源，就是你正使用的集合。这是一个例子，获得绑定到列表的产品集合的视图：

```
ICollectionView view = CollectionViewSource.GetDefaultView(lstProducts.ItemsSource);
```

GetDefaultView()方法总返回一个ICollectionView引用。你需要根据数据源转换视图对象到合适的类，可能是ListCollectionView或BindingListCollectionView。

```
var view = (ListCollectionView)
    CollectionViewSource.GetDefaultView(lstProducts.ItemsSource);
```

###  用视图导航

视图对象决定列表项目的数目（Count属性）和获得当前的数据对象一个引用（CurrentItem）或当前的位置索引（CurrentPosition）。也能使用几个方法从一记录移动到另一个，诸如MoveCurrentToFirst()，MoveCurrentToLast()，MoveCurrentToNext()，MoveCurrentToPrevious()，和MoveCurrentToPosition()。

显示绑定产品数据的绑定文本框保持不变。他们只需要指明合适的属性，如下所示：

```
<TextBlock Margin="7">Model Number:</TextBlock>
<TextBox Margin="5" Grid.Column="1" Text="{Binding Path=ModelNumber}"></TextBox>
```

但是，这例子没有包括任何列表控件，所以你要控制导航。为简化生活，你能在你的窗口类添加一个成员变量，存储指向视图的一个引用：

```
private ListCollectionView view;
```

在这种情况下，代码转换视图到合适的视图类型（ListCollectionView）而不是使用ICollectionView接口。ICollectionView接口提供了大多数功能，但是它缺乏Count属性。

当窗口第一次加载，你能获得数据，放置它到窗口的DataContext，和存储一个引用指向视图：

```
var products = App.StoreDB.GetProducts();
this.DataContext = products;

view = (ListCollectionView)
    CollectionViewSource.GetDefaultView(this.DataContext);
view.CurrentChanged += new EventHandler(view_CurrentChanged);
```

第二行在DataContext中放置产品对象的完整集合。绑定控件将沿元素树向上搜索，直到他们发现这个对象。当然，你希望绑定表达式绑定到集合的当前项目，而不是绑定到集合本身，但是WPF足够聪明能自动地推算。它自动地提供他们当前项目，所以你不需要额外的代码的一个缝合。

前一个例子有一附加的代码语句。它连接一个事件处理器到视图的CurrentChanged事件。当事件发生，你能执行几个有用的行为，诸如前一个和下一个按钮依赖于当前位置可用或不可用，和在窗口底部的TextBlock显示当前位置。

```
private void view_CurrentChanged(object sender, EventArgs e)
{
    lblPosition.Text = "Record " + (view.CurrentPosition + 1).ToString() +
      " of " + view.Count.ToString();
    cmdPrev.IsEnabled = view.CurrentPosition > 0;
    cmdNext.IsEnabled = view.CurrentPosition < view.Count - 1;
}
```

最后一步是写前一个和下一个按钮的逻辑。因为当这些按钮不能应用时，自动地不可用。你不需要考虑可能会移动到第一个项目之前或最后一个项目之后。

```
private void cmdNext_Click(object sender, RoutedEventArgs e)
{
    view.MoveCurrentToNext();
}
private void cmdPrev_Click(object sender, RoutedEventArgs e)
{
    view.MoveCurrentToPrevious();
}
```

你能添加一个组合框到窗口，用于直接跳到某一记录。

```
<ComboBox Name="lstProducts" DisplayMemberPath="ModelName"
 Text="{Binding Path=ModelName}"
 SelectionChanged="lstProducts_SelectionChanged"></ComboBox>
```

指定数据源：

```
lstProducts.ItemsSource = products;
```

默认情况下，ItemsControl的当前项目不与视图的当前项目同步。幸运地，有两个容易的方法解决问题。

第一个用传统的代码方式强制同步：

```
private void lstProducts_SelectionChanged(object sender, RoutedEventArgs e)
{
    view.MoveCurrentTo(lstProducts.SelectedItem);
}
```

一个更简单解决方案是设置ItemsControl.IsSynchronizedWithCurrentItem为真。那样，目前选择项目自动地同步匹配视图的当前位置。

#### 使用查询表帮助编辑

组合框能方便地编辑记录值。

例如，你可能有数据库一个字段接受几个预置值之一。在这种情况下，使用一个组合框，绑定它到合适的字段，在Text属性上使用一个绑定表达式。但是，填充组合框用容许的值，依靠设置它的ItemsSource属性指向你定义列表。并且如果你希望显示列表值一方式（例如，为文本）但是存储他们另一个方式（为数字编码），只要添加一个值转换器到你的Text属性绑定。

另一个情况是相关表。例如，你可能希望允许用户拾一个产品目录使用定义所有的目录列表。基本方法是相同的：设置Text属性绑定合适的字段，和用ItemsSource属性填充选项列表。如果你需要转换低层的IDs到更有意义的名字，使用一个值转换器。

### 用声明方式创造一个视图

你能在XAML标记以声明方式构造一个CollectionViewSource，和然后绑定CollectionViewSource到你的控件（诸如列表）。

从技术上，CollectionViewSource不是一个视图。它是一个帮助者类，允许你取回一个视图（使用GetDefaultView()方法）和一个工厂，能创造一个视图。

CollectionViewSource类的二最重要的属性是View，包裹视图对象，和Source，包裹数据源。CollectionViewSource也添加SortDescriptions和GroupDescriptions属性，这镜像同一地命名视图属性。当CollectionViewSource创造一个视图，它简单地传递这些属性的值到视图。

CollectionViewSource也包含一个Filter事件，你能处理执行过滤。这过滤工作方式等同于视图对象提供的过滤回调，除了它被定义为一个事件，所以你能容易地在XAML中挂钩上你的事件处理器。

例如，考虑前一个例子，使用价格范围这对产品分组。这是你如何以声明方式定义转换器和CollectionViewSource：

```
<local:PriceRangeProductGrouper x:Key="Price50Grouper" GroupInterval="50"/>
<CollectionViewSource x:Key="GroupByRangeView">
  <CollectionViewSource.SortDescriptions>
    <component:SortDescription PropertyName="UnitCost" Direction="Ascending"/>
  </CollectionViewSource.SortDescriptions>
  <CollectionViewSource.GroupDescriptions>
    <PropertyGroupDescription PropertyName="UnitCost"
       Converter="{StaticResource Price50Grouper}"/>
  </CollectionViewSource.GroupDescriptions>
</CollectionViewSource>
```

注意，SortDescription类不是WPF名字空间。为了使用它，你需要填加下面的名字空间别名：

```
xmlns:component="clr-namespace:System.ComponentModel;assembly=WindowsBase"
```

一旦你建立CollectionViewSource，你能绑定它到你的列表：

```
<ListBox ItemsSource="{Binding Source={StaticResource GroupByRangeView}}" ... >
```

似乎列表框控件绑定到CollectionViewSource，而不是CollectionViewSource暴露的视图（这被存储在CollectionViewSource.View属性）。但是，WPF数据绑定对于CollectionViewSource一个特殊的例外。当你使用它在一个绑定表达式，WPF请求CollectionViewSource创造它的视图，然后绑定视图到合适的元素。

声明式的方法没有真正地节省你任何工作。你仍然需要在运行时用代码取回数据。不同的是现在你的代码必须传递数据沿着到CollectionViewSource而不是直接提供它到列表：

```
var products = App.StoreDB.GetProducts();
var viewSource = (CollectionViewSource)
    this.FindResource("GroupByRangeView");
viewSource.Source = products;
```

可选地，你能使用XAML标记创造产品集合作为一个资源。然后你能以声明方式绑定CollectionViewSource到你的产品集合。但是，你仍然需要使用代码填充你的产品集合。

## 过滤、排序、和分组

视图跟踪数据对象集合的当前位置。这是一个重要的任务，和发现（或改变）当前项目是使用视图的最普遍原因。

视图也提供若干可选的特征那允许你管理项目的全体集合。在下几节中，你将会看到你能如何使用一个视图过滤你的数据项目（暂时地隐藏那些你不希望看见），你能如何使用它应用排序（改变数据项目顺序），和你能如何使用它应用分组（创造能被独立地导航子集合）。

### 过滤集合

过滤允许你显示满足特定条件的一个子集。当带有一个集合作为数据源工作时，你使用视图对象的Filter属性设置过滤。

Filter属性的实现有点笨拙。它接受一个Predicate委托指向一个自定义过滤方法（你创造）。这是一个例子，你能如何连接视图到方法FilterProduct()：

```
var view = (ListCollectionView) 
    CollectionViewSource.GetDefaultView(lstProducts.ItemsSource);
view.Filter = new Predicate<object>(FilterProduct);
```

笨拙之处在于你只能使用Predicate<object>类型，而不是Predicate<Product>类型。

这是一个简单的过滤器方法，只允许单价高于100的产品：

```
public bool FilterProduct(Object item)
{
    var product = (Product) item;
    return (product.UnitCost > 100);
}
```

使用匿名委托，定义内联的过滤方法：

```
var view = (ListCollectionView)
    CollectionViewSource.GetDefaultView(lstProducts.ItemsSource);
view.Filter = delegate(object item)
              {
                  Product product = (Product)item;
                  return (product.UnitCost > 100);
              };
```

尽管这是一个整洁的，优雅的方法，在更复杂的过滤器场景下，你更可能创造一个专用的过滤类。那是因为在这些情况下，你经常需要过滤使用几个不同的准则，和后来你可能希望能修改过滤准则。

过滤类包裹过滤准则和执行过滤的回调方法。这里是一个极端地简单的过滤类，过滤掉单价小于最小价格的产品：

```
public class ProductByPriceFilter
{
    public decimal MinimumPrice
    {
        get; set;
    }
    public ProductByPriceFilter(decimal minimumPrice)
    {
        MinimumPrice = minimumPrice;
    }
    public bool FilterItem(Object item)
    {
        var product = item as Product;
        if (product != null)
        {
            return (product.UnitCost > MinimumPrice);
        }
        return false;
    }
}
```

这里是创造ProductByPriceFilterer和使用它应用最小价格过滤的代码：

```
private void cmdFilter_Click(object sender, RoutedEventArgs e)
{
    decimal minimumPrice;
    if (Decimal.TryParse(txtMinPrice.Text, out minimumPrice))
    {
        var view =
            CollectionViewSource.GetDefaultView(lstProducts.ItemsSource)
            as ListCollectionView;
        if (view != null)
        {
            var filter =
                new ProductByPriceFilter(minimumPrice);
            view.Filter = new Predicate<object>(filter.FilterItem);
        }
    }
}
```

你可能想创造不同的过滤器对于过滤不同的数据的类型。例如，你可能计划创造（和重用）一个MinMaxFilter，一个StringFilter，等等。无论如何，通常更有帮助的是对于每个窗口创造一个单个的过滤类。那是因为你不能链一个以上过滤在一起。

如果你希望不重新创造ProductByPriceFilter对象情况下，修改过滤，你需要在你的窗口类存储一个成员变量引用过滤对象。然后你能修改过滤属性。但是，你也需要调用视图对象的Refresh()方法强迫列表被重新过滤。这里是一些代码，当包含最小价格的文本框的TextChanged事件发生时，调整过滤设置：

```
private void txtMinPrice_TextChanged(object sender, TextChangedEventArgs e)
{
    var view =
      CollectionViewSource.GetDefaultView(lstProducts.ItemsSource)
      as ListCollectionView;
    if (view != null)
    {
        decimal minimumPrice;
        if (Decimal.TryParse(txtMinPrice.Text, out minimumPrice) &&
          (filter != null))
        {
            filter.MinimumPrice = minimumPrice;
            view.Refresh();
        }
    }
}
```

最后，依靠设置Filter属性为空，你能完全清除过滤器。

```
view.Filter = null;
```

 

### 过滤DataTable

详见656页。

### 排序

最简单的方法是基于每个数据项目一个或多个属性值分类。每个SortDescription代表一个属性：

```
var view = CollectionViewSource.GetDefaultView(lstProducts.ItemsSource);
view.SortDescriptions.Add(
  new SortDescription("ModelName", ListSortDirection.Ascending));
```

也可以自定义排序规则：

```
public class SortByModelNameLength : IComparer
{
    public int Compare(object x, object y)
    {
        var productX = (Product)x;
        var productY = (Product)y;
        return productX.ModelName.Length.CompareTo(productY.ModelName.Length);
    }
}
```

自定义规则连接到视图：

```
var view = (ListCollectionView)
    CollectionViewSource.GetDefaultView(lstProducts.ItemsSource);
view.CustomSort = new SortByModelNameLength();
```

### 分组

正如排序，你能用容易的办法分组（基于单个的属性值）或困难地办法（使用一个自定义回调）。

简单分组法：

```
var view = CollectionViewSource.GetDefaultView(lstProducts.ItemsSource);
view.GroupDescriptions.Add(new PropertyGroupDescription("CategoryName"));
```

当你使用分组，你的列表为每个分组创造一个独立的GroupItem对象，并且它添加这些GroupItem对象到列表。GroupItem是一个内容控件，所以每个GroupItem持有合适的容器（好像ListBoxItem对象），容器带有你的实际的数据。显示你的分组的关键是格式化GroupItem元素所以它脱颖而出。

你能使用一个样式，应用格式化到一个列表中所有的GroupItem对象。但是，你可能希望不仅仅格式化—例如，你可能希望显示组标头，这要求一个模板的帮助。幸运地，ItemsControl类使两个任务容易，通过它的ItemsControl.GroupStyle属性，这提供一个GroupStyle对象的集合。尽管名字，GroupStyle类不是一个样式。它只是一个方便的包那包裹几个有用的设置对于配置你的GroupItem对象。

GroupStyle属性：

| 名字                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| ContainerStyle         | 设置被应用于GroupItem样式，对于每个分组生成。                |
| ContainerStyleSelector | 代替使用ContainerStyle，你能使用ContainerStyleSelector提供一个类那选择正确样式使用，基于分组。 |
| HeaderTemplate         | 允许你创造一个模板显示每个分组的开始内容。                   |
| HeaderTemplateSelector | 代替使用HeaderTemplate，你能使用HeaderTemplateSelector提供一个类那选择正确标头模板使用，基于分组。 |
| Panel                  | 允许你改变被用于持有分组的模板。例如，你能使用一个WrapPanel代替标准StackPanel创造一个列表平铺分组从左到右和然后向下。 |

这个例子，只设置每个分组之前的标头。

为添加一个分组标头，你需要设置GroupStyle.HeaderTemplate。你能用一个普通的数据模板填充这属性。你能使用元素的任意组合和你模板内部的数据绑定表达式。

但是，存在一诀窍。当你写你的绑定表达式，你不是绑定你列表的数据对象（在这种情况下，Product对象）。而是，你绑定分组的PropertyGroupDescription对象。那意味着如果你希望为了那个分组显示字段值，你需要绑定PropertyGroupDescription.Name属性而不是Product.CategoryName。

这是完整的模板：

```
<ListBox Name="lstProducts" DisplayMemberPath="ModelName">
  <ListBox.GroupStyle>
    <GroupStyle>
      <GroupStyle.HeaderTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding Path=Name}" FontWeight="Bold"
           Foreground="White" Background="LightGreen"
           Margin="0,5,0,0" Padding="3"/>
        </DataTemplate>
      </GroupStyle.HeaderTemplate>
    </GroupStyle>
  </ListBox.GroupStyle>
</ListBox>
```

ListBox.GroupStyle属性实际上是一个GroupStyle对象的集合。这允许你添加多个分组的水平。为做如此，你需要添加一个以上PropertyGroupDescription（顺序你希望你的分组和子组应用）和然后添加一个匹配GroupStyle对象格式每个水平。

你可能希望使用分组协同排序。如果你希望分类你的组，只确保第一SortDescription你使用分类基于分组字段。下列代码分类目录按字母顺序依靠目录名字和然后分类每个产品依靠模型名字顺序。

```
view.SortDescriptions.Add(new SortDescription("CategoryName",
  ListSortDirection.Ascending));
view.SortDescriptions.Add(new SortDescription("ModelName",
  ListSortDirection.Ascending));
```

#### 分组范围

本节讲基于数值的范围分组，依靠的是值转换器，将不同值转换为一个相同值。详见661页。

#### 分组和虚拟化

见663页。

### Live Shaping

见663页。