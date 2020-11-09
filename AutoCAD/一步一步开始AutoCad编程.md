一步一步开始AutoCad编程

1. 新建一个Dll工程，名称为..\\Visual Studio 2008\\Projects\\CuislCad

2. 添加程序集引用，浏览到AutoCAD根目录，即acad.exe所在目录，找到以下文件，添加：

   ```
   accoremgd.dll
   acdbmgd.dll
   acmgd.dll
   ```

   添加后，在属性中修改此三个文件的复制本地为【false】。

3. Hello World

在任意的实例类的sub中增加CommandMethod属性，就可以在AutoCAD中调用该过程了

```F#
open Autodesk.AutoCAD
open Autodesk.AutoCAD.DatabaseServices
open Autodesk.AutoCAD.EditorInput
open Autodesk.AutoCAD.Geometry

type AnyClass()=
    [<Runtime.CommandMethod("HelloWorld")>]
    member __.HelloWorld()= ()
```

4. 设置AutoCAD，使其启动时自动加载.Net程序集CuislCad

工具，选项，文件

1.添加搜索目录，添加信任目录：

> D:\Application Data\AutoCAD
> D:\Application Data\AutoCAD\CuislCad\CuislCad\bin\x64\Debug\net48


2.在csl.lsp文件中输入以下内容：

```lisp
(setvar \"filedia\" 0)
(command \"netload\" \"CuislCad.dll\")
(setvar \"filedia\" 1)
```

3.将csl.lsp加入到启动组。工具，AutoLisp，加载应用程序，启动组，内容，选择csl.lsp，确定。


5.  设置【导入的命名空间】

```vb

    -   Cad=Autodesk.AutoCAD

    -   CadAp=Autodesk.AutoCAD.ApplicationServices

    -   CadDb=Autodesk.AutoCAD.DatabaseServices

    -   CadEd=Autodesk.AutoCAD.EditorInput

    -   CadGe=Autodesk.AutoCAD.Geometry

    -   CadRx=Autodesk.AutoCAD.Runtime
```

6.  新增一个vb模块Starter，用于获取Drawing、Editor、Database等根变量

```vb

> Module Starter
>
> Public ReadOnly Property Drawing() As CadAp.Document
>
> Get
>
> Return CadAp.Application.DocumentManager.MdiActiveDocument
>
> End Get
>
> End Property
>
> Public ReadOnly Property Editor() As CadEd.Editor
>
> Get
>
> Return Drawing.Editor
>
> End Get
>
> End Property
>
> Public ReadOnly Property Database() As CadDb.Database
>
> Get
>
> Return Drawing.Database
>
> End Get
>
> End Property
>
> Public ReadOnly Property WorkingDatabase() As CadDb.Database
>
> Get
>
> Return CadDb.HostApplicationServices.WorkingDatabase
>
> End Get
>
> End Property
>
> End Module
> 
```

7.  在当前空间画图

```c#

public void DrawInCurrentSpace(Entity entity)

{

Database db = HostApplicationServices.WorkingDatabase;

using (Transaction trans= db.TransactionManager.StartTransaction())

{

BlockTableRecord btr = (BlockTableRecord)trans.GetObject(db.CurrentSpaceId, OpenMode.ForWrite);

btr.AppendEntity(entity);

trans.AddNewlyCreatedDBObject(entity, true);

trans.Commit();

}

}

public void DrawInCurrentSpace(IEnumerable\<Entity\> entities)

{

foreach (var entity in entities)

{

DrawInCurrentSpace(entity);

}

}
```
8.  添加图层
```C#
public ObjectId CreateLayer(LayerTableRecord ltr)

{

ObjectId layerId;

Database db = HostApplicationServices.WorkingDatabase;

using (Transaction trans = db.TransactionManager.StartTransaction())

{

LayerTable lt = (LayerTable)trans.GetObject(db.LayerTableId, OpenMode.ForWrite);

if (lt.Has(ltr.Name))

{

layerId = lt\[ltr.Name\];

}

else

{

layerId = lt.Add(ltr);

trans.AddNewlyCreatedDBObject(ltr, true);

}

trans.Commit();

}

return layerId;

}
```

提示：

> 当添加现有xaml窗体时，单个选中`xaml`文件，VS会自动带上附加的文件。