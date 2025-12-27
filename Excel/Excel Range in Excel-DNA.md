# Excel Range in Excel-DNA

You should prefer to get the values directly from the input parameter, without getting the Range COM object. It's also much more efficient doing it that way.

Your simple function might then look like this:

```c#
public static object Concat2(object[,] values)
{
    string result = "";
    int rows = values.GetLength(0);
    int cols = values.GetLength(1);
    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            object value = values[i, j];
            result += value.ToString();
        }
    }
    return result;
}
```

Typically you'd want to check the type of the value object, and do something different based on that. The `object[,]` array passed from Excel-DNA could have items of the following types:

```
double
string
bool
ExcelDna.Integration.ExcelError
ExcelDna.Integration.ExcelEmpty
ExcelDna.Integration.ExcelMissing (if the function is called with no parameter, as `=Concat2()`).
```

If you change the signature to have a single parameter of type `object` (instead of `object[,]`), like this:
```cs
public static object Concat2(object value)
```
then, depending on how the function is called, you might get one of the above types as the value or you might get an `object[,]` array as the value, so your type checks would look a bit different before you do the iteration.

If you really want information about the calling range, like the address, you need to

* Apply a `[ExcelArgument(AllowReference=true)]` attribute to the `object input` parameter.

* Check whether the object you receive is of type `ExcelReference` (this is a thin wrapper over the C API reference information).

* Either use the C API calls to get information about the `ExcelReference` (like a sheet name), or create a `Range` COM object from the information in the `ExcelReference`. This `Range` object needs to be used with care when called from a function.

Here is an example that creates a `Range` from the `ExcelReference` and reads information about the `Range`:

```c#
[ExcelFunction(IsMacroType = true)]
public static double GetBackColor([ExcelArgument(AllowReference = true)] object cell)
{
    ExcelReference rng = (ExcelReference)cell; // With throw Exception if input is not a reference (returned as #VALUE)
    Range refrng = ReferenceToRange(rng);
    return refrng.Interior.Color;
}


static Range ReferenceToRange(object xlInput)
{
    ExcelReference reference = (ExcelReference)xlInput;
    Application app = (Application)ExcelDnaUtil.Application;

    string sheetName = (string)XlCall.Excel(XlCall.xlSheetNm, reference);
    int index = sheetName.LastIndexOf("]");
    sheetName = sheetName.Substring(index + 1);
    Worksheet ws = (Worksheet)app.Sheets[sheetName];
    Range target = app.Range[ws.Cells[reference.RowFirst + 1, reference.ColumnFirst + 1], ws.Cells[reference.RowLast + 1, reference.ColumnLast + 1]];

    for (int iInnerRef = 1; iInnerRef < reference.InnerReferences.Count; iInnerRef++)
    {
        ExcelReference innerRef = reference.InnerReferences[iInnerRef];
        Range innerTarget = app.Range[ws.Cells[innerRef.RowFirst + 1, innerRef.ColumnFirst + 1], ws.Cells[innerRef.RowLast + 1, innerRef.ColumnLast + 1]];
        target = app.Union(target, innerTarget);
    }

    return target;
}
```