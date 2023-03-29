# 19 WPF 数据绑定

WPF数据绑定允许你创造从几乎任何对象的任何属性获取信息以及填充到几乎任何元素的任何属性里的绑定。

## 使用自定义对象绑定到数据库

### 建立数据访问组件

数据绑定是将数据对象绑定到界面元素，首先建立访问数据库的代码。在项目中添加一个用于访问数据数据库的类，下面是该类的结构：

```
public class StoreDB
{
    private string connectionString = @"Data Source=.";

    public Product GetProduct(int ID)
    {
        var product = new Product();
        return (product);
    }


}//class
```

在应用程序类中缓存数据库对象：

```
public partial class App : System.Windows.Application
{
    private static StoreDB storeDB = new StoreDB();
    public static StoreDB StoreDB
    {
        get { return storeDB; }
    }
}
```

 

本章的主题是数据对象与界面元素的数据绑定，数据库访问不是重点。数据访问类的完整代码可以查看560页。

### 建立数据对象

对象的公共属性绑定到元素属性。双路绑定要求属性是可读写的。下面是最基本的数据对象：

```
public class Product
{
    private string modelNumber;
    public string ModelNumber
    {
        get { return modelNumber; }
        set { modelNumber = value; }
    }

    private string modelName;
    public string ModelName
    {
        get { return modelName; }
        set { modelName = value; }
    }

    private decimal unitCost;
    public decimal UnitCost
    {
        get { return unitCost; }
        set { unitCost = value; }
    }

    private string description;
    public string Description
    {
        get { return description; }
        set { description = value; }
    }
}
```

 

### 显示绑定对象

使用StoreDB在运行时创造Product对象，然后将Product对象绑定到你的窗口。

考虑一个简单的窗口，用户可以提供产品编码来查询产品的详细信息。窗口第一层标记：

```
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"></RowDefinition>
            <RowDefinition Height="*"></RowDefinition>
        </Grid.RowDefinitions>

        <Grid>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"></ColumnDefinition>
                <ColumnDefinition></ColumnDefinition>
                <ColumnDefinition Width="Auto"></ColumnDefinition>
            </Grid.ColumnDefinitions>
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto"></RowDefinition>
            </Grid.RowDefinitions>

            <TextBlock Margin="7">Product ID:</TextBlock>
            <TextBox Name="txtID" Margin="5" Grid.Column="1">356</TextBox>
            <Button Click="cmdGetProduct_Click" Margin="5" Padding="2" Grid.Column="2">Get Product</Button>
        </Grid>

        <Border Grid.Row="1" Padding="7" Margin="7" Background="LightSteelBlue">
            <!--详见下面的标记-->
        </Border>
    </Grid>
```

当设计这个窗口时，你没有访问在运行时提供数据的Product对象。但是，在不指明数据源的情况下，仍创造了绑定。你只需要指明每个元素使用Product类的属性。下面是显示产品细节的全部标记：

```
<Grid  Name="gridProductDetails">
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="Auto"></ColumnDefinition>
        <ColumnDefinition></ColumnDefinition>
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"></RowDefinition>
        <RowDefinition Height="Auto"></RowDefinition>
        <RowDefinition Height="Auto"></RowDefinition>
        <RowDefinition Height="Auto"></RowDefinition>
        <RowDefinition Height="*"></RowDefinition>
    </Grid.RowDefinitions>

    <TextBlock Margin="7">Model Number:</TextBlock>
    <TextBox Margin="5" Grid.Column="1" 
             Text="{Binding Path=ModelNumber}"></TextBox>
    
    <TextBlock Margin="7" Grid.Row="1">Model Name:</TextBlock>
    <TextBox Margin="5" Grid.Row="1" Grid.Column="1" 
             Text="{Binding Path=ModelName}"></TextBox>
    
    <TextBlock Margin="7" Grid.Row="2">Unit Cost:</TextBlock>
    <TextBox Margin="5" Grid.Row="2" Grid.Column="1" 
             Text="{Binding Path=UnitCost}"></TextBox>
    
    <TextBlock Margin="7" Grid.Row="3">Description:</TextBlock>
    <TextBox Margin="7" Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2"
             VerticalScrollBarVisibility="Visible" TextWrapping="Wrap" 
             Text="{Binding Path=Description}"></TextBox>                
</Grid>
```

注意，包围所有细节的网格面板有名字，可以在代码中完成数据绑定。

Grid容器包括了所有细节控件。设置了他的DataContext属性，就批量设置了容器所有包括数据绑定子元素的数据源。在按钮中指定网格的DataContext属性：

```
private void cmdGetProduct_Click(object sender, RoutedEventArgs e)
{
    var ID = Int32.Parse(txtID.Text);
    gridProductDetails.DataContext = App.StoreDB.GetProduct(ID);
}
```

 

### 绑定空值：

对于值类型使用可空类型表示数据库的可空类型，当然，引用类型总是支持空值。如：decimal?表示可空的decimal。绑定一个空值的结果是控件根本不显示任何东西。在绑定表达式中设置TargetNullValue属性，能修改处理空值的方式。

```
Text="{Binding Path=Description, TargetNullValue=[No Description Provided]}"
```

### 更新数据库

因为文本框等可编辑元素到数据对象的绑定是双路绑定，WPF自动更新数据对象。

给StoreDB类增加一个UpdateProduct()方法，添加一个Update按钮到窗口。当点击时，首先要确保焦点从最后一个编辑的文本框上移开，以更新数据对象；然后从Grid的上下文获取当前的产品对象；最后调用StoreDB中的更新数据库代码。下面是提交更新完整的代码：

```
private void cmdUpdate_Click(object sender, RoutedEventArgs e)
{
    FocusManager.SetFocusedElement(this, (Button)sender);

    var product = (Product) gridProductDetails.DataContext;
    App.StoreDB.UpdateProduct(product);
}
```

 

### 改变通知

如果你直接修改数据对象的数据，那么数据对象不会自动更新WPF界面元素。一个解决方案是给数据对象添加改变通知。

你能实现System.ComponentModel.INotifyPropertyChanged接口，它需要单个的PropertyChanged事件。当一个属性改变时，提供这个属性的名字，引起PropertyChanged事件。

```
public class Product : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;
    public void OnPropertyChanged(PropertyChangedEventArgs e)
    {
        if (PropertyChanged != null)
          PropertyChanged(this, e);
    }
}
```

在属性设置器中引发事件：

```
private decimal unitCost;
public decimal UnitCost
{
    get { return unitCost; }
    set {
        unitCost = value;
        OnPropertyChanged(new PropertyChangedEventArgs("UnitCost"));
    }
}
```

实现改变通知接口的数据类代码比较固定，可以使用代码生成器自动生成代码。

**提示：**如果几个值改变，你能调用OnPropertyChanged()并且传入一个空字符串。这告诉WPF对绑定到此类任何属性的绑定表达式重新估值。

## 绑定到对象的集合

在WPF，ItemsControl类的派生类都能显示项目的整个列表。数据绑定包括ListBox、ComboBox，ListView，和DataGrid（和对于层次数据Menu和TreeView）。

为支持集合绑定，ItemsControl类定义三个关键属性：

| 名字              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| ItemsSource       | 指向将被显示在列表的对象集合                                 |
| DisplayMemberPath | 标识将被用于创造每个项目显示文本的属性。                     |
| ItemTemplate      | 接受一个数据模板。将被用于创造每个项目的视觉外观。这属性比DisplayMemberPath更强力。 |

ItemsSource属性接受所有实现IEnumerable接口的对象，但是，对于基本的IEnumerable接口获得的支持被限制为只读绑定。要想编辑集合（例如，插入和移除项目），你需要更多构架，一会将看到。

### 显示和编辑集合项目

考虑一个窗口，显示一个产品列表，当你从列表中选择一个产品时，该产品的信息出现在窗口的下面部分，在这里你能编辑它。

首先建立数据访问逻辑。在这里，要在StoreDB中定义一个GetProducts()方法，从数据库获取产品集合。

在窗口的后台代码中增加一个产品集合的products字段，并且为GetProducts按钮添加Click事件处理器。用于填充products字段，然后将其绑定到列表框。

```
private List<Product> products;
private void cmdGetProducts_Click(object sender, RoutedEventArgs e)
{
    products = App.StoreDB.GetProducts();
    lstProducts.ItemsSource = products;
}
```

你需要通过以下方法之一，告诉列表框如何显示数据对象：DisplayMemberPath、覆盖数据对象的ToString()方法、使用数据模板。本例中，通过为DisplayMemberPath属性提供属性名设置列表框的显示方式：

```
<ListBox Name="lstProducts" DisplayMemberPath="ModelName"/>
```

为了在窗口下部的细节网格中显示当前选择的项目细节。你只需为Grid.DataContext属性设置一个绑定表达式，从列表中提取被选择的Product对象，如下所示：

```
<Grid DataContext="{Binding ElementName=lstProducts, Path=SelectedItem}">
  ...
</Grid>
```

这时，代码已经是全功能的了。每当修改控件的内容，然后切换焦点后，绑定数据自动更新。甚至，列表的显示内容也自动更新了。

### 插入和移除集合项目

为了能跟踪集合改变，你需要使用实现INotifyCollectionChanged接口的集合。WPF包含一个使用INotifyCollectionChanged接口的集合：ObservableCollection类。

ObservableCollection类相当于Windows Forms世界中的BindingList。

```
var products = new ObservableCollection<Product>();
```

ObservableCollection类派生自List类。

### 绑定到ADO.NET对象

见572页。

### 绑定到Linq表达式

Linq查询结果是 IEnumerable<T>接口类型。如果你要使用ObservableCollection<T>来监视数据源的变化，则需要强制转化的代码：

```
var products = App.StoreDB.GetProducts();

var matches = (
    from product in products
    where product.UnitCost >= 100
    select product
    ).ToList();

var productMatchesTracked = new ObservableCollection<Product>(matches);
```

 

### 用Visual Studio设计数据对话框

通过“数据源”可以快速的设计常规的数据对话框。通过实体对象模型可以快速建立数据访问组件。详见575页。

## 改善长列表的性能

见576页。

## 验证

### 在数据对象内添加验证

在数据对象的属性设置器中增加验证规则：

```
public decimal UnitCost
{
    get { return unitCost; }
    set
    {
        if (value < 0)
            throw new ArgumentException("UnitCost cannot be negative.");
        else
        {
            unitCost = value;
            OnPropertyChanged(new PropertyChangedEventArgs("UnitCost"));
        }
    }
}
```

需要发送异常到相应的元素，要借助于ExceptionValidationRule类

ExceptionValidationRule

```
<TextBox Margin="5" Grid.Row="2" Grid.Column="1">
  <TextBox.Text>
    <Binding Path="UnitCost">
      <Binding.ValidationRules>
        <ExceptionValidationRule></ExceptionValidationRule>
      </Binding.ValidationRules>
    </Binding>
  </TextBox.Text>
</TextBox>
```

当一个错误发生时：

- Validation.HasError附加属性被设置为真。
- ValidationError被创建，说明错误细节。
- 如果Binding.NotifyOnValidationError属性为真，元素的Validation.Error附加事件发生。

Validation.ErrorTemplate属性提供当错误发生时元素使用的模板。默认，文本框的边框变成红色。

### INotifyDataErrorInfo接口

许多面向对象纯粹主义者更喜欢不引起异常去指出用户输入错误。可能有几个原因，包括下面的：用户输入错误不是异常的条件，错误条件可以依赖多个属性值之间相互作用，并且有时值得保持不正确的值为了进一步处理而不是拒绝它们彻底。WPF提供二接口允许你建立对象报告错误无需抛出异常。这些接口是IDataErrorInfo和INotifyDataErrorInfo。 

**注意：**IDataErrorInfo和INotifyDataErrorInfo接口有同样的目标—他们用更礼貌的错误通知系统替换侵略的未处理异常。IDataErrorInfo接口是原始的错误跟踪接口，WPF包含它为了向后兼容。INotifyDataErrorInfo接口类似但是更丰富的接口。它支持附加的特征，诸如多个错误每属性和异步验证。

下面的例子显示如何使用INotifyDataErrorInfo接口去侦察Product对象问题。第一步是实现接口：

```
public class Product : INotifyPropertyChanged, INotifyDataErrorInfo
{ ... }
```

INotifyDataErrorInfo接口只要求三成员。ErrorsChanged事件当错误被添加或移除时发射。HasErrors属性返回真或假去指出是否数据对象有错误。最后，GetErrors()方法提供充分错误信息。在你可以实现这些方法之前，你需要设法去跟踪在你的代码中错误。最好是一私有集合，像这样：

```
private Dictionary<string, List<string>> errors = 
  new Dictionary<string, List<string>>();
```

乍一看，这集合看着有点怪。要理解为什么，你需要知道二事实。首先，INotifyDataErrorInfo接口预期你去链接你的错误到指定的属性。第二，每个属性可以有一个或多个错误。最容易的方法去跟踪这错误信息是用Dictionary<T,K>集合那由属性名字索引。在词典中每个入口自己是错误的集合。这例子使用简单的字符串列表。

但是，你可以使用成熟的错误对象去把关于那错误的多个片信息捆束起来，包括诸如文本消息，错误代码，严重等级，等等细节。

一旦集合就位，当错误发生时你只需要添加它（并且如果错误被修正移除错误信息）。为使这过程更容易，Product类在这个例子中添加一对私有方法命名SetErrors()和ClearErrors()：

```
public event EventHandler<DataErrorsChangedEventArgs> ErrorsChanged;
private void SetErrors(string propertyName, List<string> propertyErrors)
{
    // Clear any errors that already exist for this property.
    errors.Remove(propertyName);

    // Add the list collection for the specified property.
    errors.Add(propertyName, propertyErrors);

    // Raise the error-notification event.
    if (ErrorsChanged != null) 
        ErrorsChanged(this, new DataErrorsChangedEventArgs(propertyName));
}
                
private void ClearErrors(string propertyName)
{            
    // Remove the error list for this property.
    errors.Remove(propertyName);         
   
    // Raise the error-notification event.
    if (ErrorsChanged != null) 
        ErrorsChanged(this, new DataErrorsChangedEventArgs(propertyName));
}
```

 这里是错误处理逻辑，确保Product.ModelNumber属性受限于字母数字的字符串。

```
private string modelNumber;
public string ModelNumber
{
    get { return modelNumber; }
    set
    {
        modelNumber = value; 
                             
        bool valid = true;
        foreach (char c in modelNumber)
        {
            if (!Char.IsLetterOrDigit(c))
            {
                valid = false;
                break;
            }
        }
        if (!valid)
        {
            List<string> errors = new List<string>();
            errors.Add("The ModelNumber can only contain letters and numbers.");
            SetErrors("ModelNumber", errors);
        }
        else
        {
            ClearErrors("ModelNumber");
        }
                                
        OnPropertyChanged(new PropertyChangedEventArgs("ModelNumber"));
    }
}
```

最后一步是实现GetErrors()和HasErrors()方法。GetErrors()方法返回指定属性的错误列表（或所有属性所有错误）。HasErrors()属性返回真表示Product类有一个或多个错误。

```
public IEnumerable GetErrors(string propertyName)
{
    if (string.IsNullOrEmpty(propertyName))
    {
        // Provide all the error collections.
        return (errors.Values);
    }
    else
    {
        // Provice the error collection for the requested property
        // (if it has errors).
        if (errors.ContainsKey(propertyName))
        {
            return (errors[propertyName]);
        }
        else
        {
            return null;
        }
    }  
}
public bool HasErrors
{
    get
    {
        // Indicate whether the entire Product object is error-free.
        return (errors.Count > 0);
    }
}
```

为告诉WPF使用INotifyDataErrorInfo接口并且当属性被修改时使用它去核对错误，绑定的ValidatesOnNotifyDataErrors属性必须是真：

```
<TextBox Margin="5" Grid.Row="2" Grid.Column="1" x:Name="txtModelNumber"
  Text="{Binding Path=ModelNumber, Mode=TwoWay, ValidatesOnNotifyDataErrors=True,
NotifyOnValidationError=True}"></TextBox>
```

从技术上，你不需要显式地设置ValidatesOnNotifyDataErrors，因为默认情况下它是真（不同于类似ValidatesOnDataErrors属性，被用于IDataErrorInfo接口）。但是，显式地设置它仍是好想法，这使你的意图在标记中清楚。

顺便，你可以结合两个方法依靠创造对于一些类型的错误抛出异常的数据对象，并且利用IDataErrorInfo或INotifyDataErrorInfo去报告其他。但是记住这些二方法完全不同。当异常被触发时，在数据对象中的属性不被更新。但是当你使用IDataErrorInfo或INotifyDataErrorInfo接口时，无效的值被允许但是被标记。数据对象被更新，但是你可以使用通知和BindingValidationFailed事件去通知用户。

### 自定义验证规则

 定义一个类，派生自ValidationRule(在System.Windows.Controls名字空间)。并且覆盖Validate()方法去执行验证。如果渴望，你可以添加接受其它细节的属性，可用于去影响你的验证(例如，检测文本的验证规则可能包含布尔CaseSensitive属性)。

这里是一个完整的验证规则，其限制十进制值落在最小和最大值之间。默认情况下，最小值被设为0，最大值是十进制数据类型最大的数，这验证规则打算使用货币值。但是，通过属性可以配置这些两个细节，实现了最大的灵活性。

```
public class PositivePriceRule : ValidationRule
{
    private decimal min = 0;
    private decimal max = Decimal.MaxValue;
    public decimal Min
    {
        get { return min; }
        set { min = value; }
    }
    public decimal Max
    {
        get { return max; }
        set { max = value; }
    }
    public override ValidationResult Validate(object value, CultureInfo cultureInfo)
    {
        decimal price = 0;
        try
        {
            if (((string)value).Length > 0)
                price = Decimal.Parse((string)value, NumberStyles.Any, culture);
        }
        catch
        {
            return new ValidationResult(false, "Illegal characters.");
        }
        if ((price < Min) || (price > Max))
        {
            return new ValidationResult(false,
              "Not in the range " + Min + " to " + Max + ".");
        }
        else
        {
            return new ValidationResult(true, null);
        }
    }
}
```

注意验证逻辑使用Decimal.Parse()接受NumberStyles枚举值的超载版本。那是因为验证总是在转化之前被执行。如果你应用验证器和转换器到同样的字段，你需要确信的你的验证在有货币符号出现时将成功。验证逻辑的成功或失败由返回ValidationResult对象指出。IsValid属性指出验证是否成功，如果不，ErrorContent属性提供一个对象描述问题。在这个例子中，错误内容被设置为字符串将被显示在用户界面中，这是最常见的方法。

一旦完成验证规则，你准备附着它到一个元素，依靠添加它到Binding.ValidationRules集合。这是一个例子，使用PositivePriceRule和设置Maximum在999.99：

```
<TextBlock Margin="7" Grid.Row="2">Unit Cost:</TextBlock>
  <TextBox Margin="5" Grid.Row="2" Grid.Column="1">
    <TextBox.Text>
      <Binding Path="UnitCost">
        <Binding.ValidationRules>
          <local:PositivePriceRule Max="999.99" />
        </Binding.ValidationRules>
      </Binding>
    </TextBox.Text>
</TextBox>
```

经常，你将定义一个独立的验证规则对象为每个元素使用同样的类型的规则。那是因为你可能希望调节验证属性(诸如在PositivePriceRule中最小和最大值)独立地。如果你知道你希望使用精确地同样的验证规则对于一个以上绑定，你可以定义验证规则作为一资源并且简单地指向它在每个绑定中使用StaticResource标记扩展。

你可能已经知道，Binding.ValidationRules集合可以取无数的规则。当值被提交到源时，WPF按照顺序核对每个验证规则。（记住，当失去焦点时文本框的值被提交到源，除非你用UpdateSourceTrigger属性指定。如果所有验证成功，那么WPF调用转换器（如果存在）并且应用值到源。

**注意：**如果你跟在ExceptionValidationRule后面添加PositivePriceRule，PositivePriceRule将首先被估值，将捕获值不在范围内的错误。但是，ExceptionValidationRule将捕获类型转换错误。

当你用PositivePriceRule执行验证时，行为等同于当你使用ExceptionValidationRule时—红轮廓的文本框，HasError和Errors属性被设置，并且Error事件发射。为提供用户一些有帮助的反馈，你需要加一点代码或自定义ErrorTemplate。在下几节，你将学习如何处理两个方法。

**提示：**自定义验证规则能是极端地特殊的以便他们用于指定的属性的指定约束，或非常通用以便他们可以被重用在各种场景。例如，你可以容易地使用正则表达式创造自定义验证规则验证字符串，你可以使用验证规则验证各种基于模式的文本数据，诸如电子邮件地址，电话数字，IP地址，和ZIP编码。

### 对验证错误做出反应

 在前例中，惟一指示用户接收错误是冒犯文本框周围红轮廓。为提供更多信息，你可以处理错误事件，在错误被存储或清除时发射。但是，你必须首先确信的你设置Binding.NotifyOnValidationError属性为真：

```
<Binding Path="UnitCost" NotifyOnValidationError="True">
```

错误事件是冒泡路由事件，如此你可以附着父容器事件处理器，可以为多个控件处理Error事件，如下所示：

```
<Grid Name="gridProductDetails" Validation.Error="validationError">
```

这里是反应事件代码并且弹出消息框显示错误信息。（更少破坏性选项将是显示工具提示或显示错误信息在窗口别的某处。）

```
private void validationError(object sender, ValidationErrorEventArgs e)
{
    // Check that the error is being added (not cleared).
    if (e.Action == ValidationErrorEventAction.Added)
    {
        MessageBox.Show(e.Error.ErrorContent.ToString());
    }
}
```

ValidationErrorEventArgs.Error属性提供ValidationError对象把几个有用的细节捆在一起，包括引起问题异常（Exception），被违反的验证规则（ValidationRule），相关的Binding对象（BindingInError），和ValidationRule对象返回的任何自定义信息（ErrorContent）。

如果使用自定义验证规则，你将几乎确定地选择放置错误信息在ValidationError.ErrorContent属性中。如果使用ExceptionValidationRule，ErrorContent属性将返回Message属性对应的异常的。但是，有一个问题。如果因为数据类型不能被转换到合适的值发生异常，ErrorContent工作符合预期并且报告问题。但是，如果数据对象属性设置器抛出异常，这异常被包装在TargetInvocationException中，并且ErrorContent从TargetInvocationException提供文本。Message属性给出没有太多帮助的警告“调用的目标扔出异常。“

因此，如果你的属性设置器引起异常，你将需要添加代码核对TargetInvocationException的InnerException属性。如果它不是空，你可以取回原始的异常对象并且使用它的Message属性代替ValidationError.ErrorContent属性。

### 获取错误列表

 有时，你可能希望获取在当前的窗口中（或在窗口中一个给定容器）所有错误的列表。这任务相对简单—所有你需要做的是遍历元素树测试验证每个元素的Validation.HasError属性。

下列代码例程示范一个例子。专门找出在文本框对象中的无效数据。它使用递归的代码向下挖掘贯穿全体的元素层次结构。顺便，错误信息被聚合到单个的消息里，然后被显示给用户。

```
private void cmdOK_Click(object sender, RoutedEventArgs e)
{
    string message;
    if (FormHasErrors(message))
    {
        // Errors still exist.
        MessageBox.Show(message);
    }
    else
    {
        // There are no errors. You can continue on to complete the task
        // (for example, apply the edit to the data source.).
    }
}
private bool FormHasErrors(out string message)
{
    var sb = new StringBuilder();
    GetErrors(sb, gridProductDetails);
    message = sb.ToString();
    return message != "";
}
private void GetErrors(StringBuilder sb, DependencyObject obj)
{
    foreach (object child in LogicalTreeHelper.GetChildren(obj))
    {
        var element = child as TextBox;
        if (element == null) continue;
        if (Validation.GetHasError(element))
        {
            sb.Append(element.Text + " has errors:\r\n");
            foreach (ValidationError error in Validation.GetErrors(element))
            {
                sb.Append("  " + error.ErrorContent.ToString());
                sb.Append("\r\n");
                }
            }
            // Check the children of this object for errors.
            GetErrors(sb, element);
        }
    }
}
```

在一个更完整的实现中，FormHasErrors()方法将可能创造一个带有错误信息对象的集合。然后cmdOK_Click()事件处理器将担负构造一个合适的消息。

### 显示不同错误指示器

 为充分利用WPF验证，你将希望创造你自己的错误模板，用恰当的方式标记错误。乍一看，这是报告一个错误相当低层的方法—毕竟，标准控件模板详尽无遗使你能自定义控件的组合。但是，错误模板不是就像平常的控件模板。

错误模板使用装饰器层，这是一个绘制层，只存在在平常的窗口内容上方。使用装饰器层，你可以添加一个视觉的装饰指出错误无需替换下面的控件的控件模板或改变你的窗口布局。对于文本框的标准错误模板依靠在对应的文本框正上面添加红色边界元素浮动（这保持下面的元素不变）。你可以使用一个错误模板添加另外的细节诸如图像，文本，或一些其它的种类的图形的细节，吸引对问题的注意。

下面的标记显示一个例子。它定义一个错误模板，使用一个绿色边界并且添加一个星号挨着带有无效输入的控件。模板被包装在样式中以便它自动地应用于在当前窗口中的所有文本框：

```
<Style TargetType="{x:Type TextBox}">
    <Setter Property="Validation.ErrorTemplate">
        <Setter.Value>
            <ControlTemplate>
                <DockPanel LastChildFill="True">
                    <TextBlock DockPanel.Dock="Right"
                   Foreground="Red" FontSize="14" FontWeight="Bold"
                   ToolTip="{Binding ElementName=adornerPlaceholder, Path=AdornedElement.(Validation.Errors)[0].ErrorContent}"
                   >*</TextBlock>
                    <Border BorderBrush="Green" BorderThickness="1">
                        <AdornedElementPlaceholder Name="adornerPlaceholder"></AdornedElementPlaceholder>
                    </Border>
                </DockPanel>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

 

AdornedElementPlaceholder是使这技术工作的胶水。它代表控件本身，存在于元素层。依靠使用AdornedElementPlaceholder，你可以布置你关联到下面文本框的内容。

结果，在这个例子中边界被直接放置在文本框的正上方，星号被放置在他的右侧。最好的是，新的错误模板内容被叠加在现存内容的顶上而无需改变原始的窗口中的布局。（事实上，如果你粗心大意并且包含太许多内容在装饰器层中，你将覆写窗口。）

上述代码TextBlock.ToolTip用于显示错误的附加信息。这里使用数据绑定抽取错误附加的信息。一个好的方法是取出第一个错误的错误内容，使用它用于你的错误指示器的工具提示文本。

绑定表达式的路径稍微有些费解，需要更仔细的研究。绑定表达式的源是AdornedElementPlaceholder，它被定义在控件模板中。

AdornedElementPlaceholder类提供到在下面元素的引用（在这种情况下，带有错误的TextBox对象）通过AdornedElement属性。

为取回实际的错误，你需要核对这元素的Validation.Errors属性。但是，你需要用括弧括起Validation.Errors属性指出它是一个附着属性，而不是文本框类的属性。

可选地，你可能希望对于边界或文本框本身在工具提示中显示错误消息。你可以执行这诀窍无需自定义错误模板的帮助—全部要做的是文本框控件当Validation.HasError变成真时触发器反应，并且显示带有错误消息的工具提示。这是一个例子：

```
<Style TargetType="{x:Type TextBox}">

    <Style.Triggers>
        <Trigger Property="Validation.HasError" Value="True">
            <Setter Property="ToolTip"
                    Value="{Binding RelativeSource={RelativeSource Self}, Path=(Validation.Errors)[0].ErrorContent}" />
        </Trigger>
    </Style.Triggers>
</Style>
```

 

### 验证多重值

 

## 数据提供者

见595页。