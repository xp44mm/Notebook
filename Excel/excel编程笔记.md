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