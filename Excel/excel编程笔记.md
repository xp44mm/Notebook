Name对象的RefersTo必须使用A1样式的锁定地址，$，无锁定表示相对地址。
Names集合索引是从1开始计数的。
ws.Names.[nickname]用不带前缀的名称索引。

```F#
///测试小名用法
let testNicknames (ws:Worksheet) =
    let nicknames = getNicknames ws

    nicknames
    |> List.map(fun nickname -> ws.Names.[nickname]) //使用nickname而非使用限定名称
    |> List.map listPropsOfName 
    |> String.concat System.Environment.NewLin
```

Excel中的集合为非泛型集合，用`Seq.cast<>`转换为标准集合：

```F#
///获取名称
let getNames(names:Names) = 
    names 
    |> Seq.cast<Name> 
    |> Seq.filter(fun nm -> nm.Visible)
    |> Seq.toList

///获取工作表
let getWorksheets(wb:Workbook) =
    wb.Worksheets
    |> Seq.cast<Worksheet>
    |> Seq.toList
```

CELL 函数

工作表的名称不可以从标准名称转换为公式中的名称，只能从公式中的书写名称转化为标准名称

### InnerObject

如何从Microsoft.Office.Tools.Excel中的对象获取Microsoft.Office.Interop.Excel相应的对象？
答：Excel的宿主项目与Excel的宿主控件位于Microsoft.Office.Tools.Excel命名空间，它们都有一个InnerObject属性。使用InnerObject属性，可以获得Microsoft.Office.Interop.Excel空间中相应的对象。


Excel 宿主项目:
Microsoft.Office.Tools.Excel.Workbook.InnerObject
Microsoft.Office.Tools.Excel.WorkbookBase.InnerObject

Microsoft.Office.Tools.Excel.Worksheet.InnerObject
Microsoft.Office.Tools.Excel.WorksheetBase.InnerObject

Microsoft.Office.Tools.Excel.ChartSheet.InnerObject
Microsoft.Office.Tools.Excel.ChartSheetBase.InnerObject

Excel 宿主控件：
Microsoft.Office.Tools.Excel.Chart.InnerObject
Microsoft.Office.Tools.Excel.ListObject.InnerObject
Microsoft.Office.Tools.Excel.NamedRange.InnerObject
Microsoft.Office.Tools.Excel.XmlMappedRange.InnerObject

Range.Row,Range.Column,Cells是基于1的，Offset是基于0的。

如何不改变单元格的内容，设置单元格的下拉列表同另一个单元格？复制源单元格，选择性粘贴，只粘贴验证。

在Excel工作表项目中，当引用FsharpExcel库的时候，在编译时会有警告，修改如下程序集的“嵌入互操作类型”为False:

Microsoft.Office.Interop.Excel
Microsoft.Vbe.Interop
office

如何设置上标下标？选中要修改的单元格，在公式编辑栏中，选中要上标的内容，然后右键选则设置单元格格式，在特殊效果中选择上标，点确定完成。

使用代码生成工具时,注意使用这两个函数:
转化为文本:=text(value,format)
转置:{=transpose(range)}

