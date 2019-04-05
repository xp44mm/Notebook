## The FileSystemWatcher Type

The `FileSystemWatcher` component lets you monitor a directory or a directory tree so that you get a notification when something happens inside it—for example, when a file or a subdirectory is created, deleted, or renamed or when the folder's attributes are changed. This component can be useful in many circumstances. For example, say that you're creating an application that automatically encrypts all the files stored in a given directory. Without this component, you should poll the directory at regular time intervals (typically using a `Timer`), but the `FileSystemWatcher` component makes this task easier. Another good example of how this component can be useful is when you cache a data file in memory to access its contents quickly, but need to reload it when another application modifies the data.

This component works on Microsoft Windows `Millennium` `Edition` (Me), Windows NT, Windows 2000, Windows XP, and Windows Server 2003.

### Initializing a FileSystemWatcher Component

You can create a `FileSystemWatcher` component in either of two ways: by means of code or by dragging it from the `Components` tab of the `Toolbox` to the tray area of a Windows `Forms` class, a Web `Forms` page, or another Microsoft Visual Studio designer. There's no noticeable difference in performance or flexibility, so any method is fine. The demo application uses a component in a form's component tray area, which I have renamed fsw (see Figure 15-1), but creating it through code is equally simple:

``` FSharp
// Use WithEvents to be able to trap events from this object.
Dim WithEvents fsw As New FileSystemWatcher()
```

![`Image` from book](images/fig623_01.jpg)
Figure 15-1: The demo application that enables you to experiment with the `FileSystemWatcher` component

Before you use this component, you must initialize at least its `Path`, `IncludeSubdirectories`, `Filter`, and `NotifyFilter` properties. The `Path` property is the name of the directory that you want to watch; notice that you're notified of changes occurring inside the directory, but not of changes to the directory's attributes (such as its `Hidden` or `ReadOnly` attribute).

The `IncludeSubdirectories` property should be set to `False` if you want to be notified of any change inside the specified directory only, or to `True` if you want to watch for changes in the entire directory tree whose root is the folder specified by the `Path` property.

The `Filter` property lets you specify which files you're interested in; for example, use *.* to get notifications about all the files in the directory or *.txt to watch only files with the .txt extension. The default value for this property is a null string, which means all files (same as *.*).





The `NotifyFilter` property is a bit-coded value that specifies which kind of modifications are announced by means of the component's `Changed` event. This property can be a combination of one or more `NotifyFilters` enumerated values: `Attributes`, `CreationTime`, `DirectoryName`, `FileName`, `LastAccess`, `LastWrite`, `Security`, and `Size`. The initial value of this property is `LastWrite` or `FileName` or `DirectoryName`, so by default you don't get notifications when an attribute is changed.

Here's an example of how you can set up a `FileSystemWatcher` component to watch for events in the C:\Windows directory and its subdirectories:

``` FSharp
Dim WithEvents fsw As New FileSystemWatcher()
…
fsw.Path = @"c:\windows"
fsw.IncludeSubdirectories = true          // Watch subdirectories.
fsw.Filter = "*.dll"                      // Watch only DLL files.
// Add attribute changes to the list of changes that can fire events.
fsw.NotifyFilter = fsw.NotifyFilter ||| NotifyFilters.Attributes
// Enable event notification.
fsw.EnableRaisingEvents = true
```

### Getting Notifications

Once you've set up the component correctly, you can get a notification when something happens. You can achieve this by writing event handlers or using the `WaitForChanged` method.

#### Events

The simplest way to get a notification from the `FileSystemWatcher` component is by writing handlers for the component's events. However, events don't fire until you set `EnableRaisingEvents` to `True`. The `Created`, `Deleted`, and `Changed` events receive a `FileSystemEventArgs` object, which exposes two important properties: `Name` (the name of the file that has been created, deleted, or changed) and `FullPath` (its complete path):

``` FSharp
member this.fsw_Created(sender : Object,  e : FileSystemEventArgs) Handles fsw.Created =
   Console.WriteLine("File created: {0}", e.FullPath)


member this.fsw_Deleted(sender : Object,  e : FileSystemEventArgs) Handles fsw.Deleted =
    Console.WriteLine ("File deleted: {0}", e.FullPath)


member this.fsw_Changed(sender : Object,  e : FileSystemEventArgs) Handles fsw.Changed =
    Console.WriteLine ("File changed: {0}", e.FullPath)
```

The `FileSystemEventArgs` object also exposes a `ChangeType` enumerated property, which tells whether the event is a create, delete, or change event. You can use this property to use a single handler to manage all three events, as in this code:

``` FSharp
member this.fsw_All(sender : Object, e : FileSystemEventArgs)  Handles fsw.Changed, fsw.Created, fsw.Deleted =
   Console.WriteLine("File changed: {0} ({1})", e.FullPath, e.ChangeType)
```

The `Changed` event receives no information about the type of change that fired the event, such as a change in the file's `LastWrite` date or attributes. Finally, the `Renamed` event receives a `RenamedEventArgs` object, which exposes two additional properties: `OldName` (the name of the file before being renamed) and `OldFullPath` (its complete path):

``` FSharp
member this.fsw_Renamed(sender : Object, e : RenamedEventArgs)  Handles fsw.Renamed =
   Console.WriteLine("File renamed: {0} => {1}", e.OldFullPath, e.FullPath)
```

You can also have multiple `FileSystemWatcher` components forward their events to the same event handler. In this case, use the first argument to detect which specific component raised the event.

The `FileSystemWatcher` component raises one event for each file and for each action on the file. For example, if you delete 10 files, you receive 10 distinct `Deleted` events. If you move 10 files from one directory to another, you receive 10 `Deleted` events from the source directory and 10 `Created` events from the destination directory.

#### The WaitForChanged Method

If your application doesn't perform any operation other than waiting for changes in the specified path, or if you monitor file operations from a secondary thread, you can write simpler and more efficient code by using the `WaitForChanged` method. This method is synchronous and doesn't return until a file change is detected or the (optional) timeout expires. On return from this method the application receives a `WaitForChangedResult` structure, whose fields enable you to determine whether the timeout elapsed, the type of event that occurred, and the name of the involved file:

``` FSharp
// Create a *new* FileSystemWatcher component with values from
// the txtPath and txtFilter controls.
let tmpFsw = new FileSystemWatcher(txtPath.Text, txtFilter.Text)
// Wait max 10 seconds for any file event.
let res: WaitForChangedResult = tmpFsw.WaitForChanged(WatcherChangeTypes.All, 10000)

// Check whether the operation timed out.
if res.TimedOut then
   Console.WriteLine("10 seconds have elapsed without an event")
else
    Console.WriteLine("Event: {0} ({1}), res.Name, res.ChangeType.ToString())
```

The `WaitForChanged` method traps changes only in the directory the `Path` property points to and ignores the `IncludeSubdirectories` property. For this reason, the `WaitForChangedResult` structure includes a `Name` field but not a `FullPath` field. The first argument you pass to the `WaitForChanged` method lets you further restrict the kind of file operation you want to intercept:

``` FSharp
// Pause the application until the c:\temp\temp.dat file is deleted.
tmpFsw = New FileSystemWatcher(@"c:\temp", "temp.dat")
tmpFsw.WaitForChanged(WatcherChangeTypes.Deleted)
```

#### Buffer Overflows

You should be aware of potential problems when too many events fire in a short time. The `FileSystemWatcher` component uses an internal buffer to keep track of file system actions so that events can be raised for each one of them even if the application can't serve them fast enough. By default, this internal buffer is 8 KB long and can store about 160 events. Each event takes 16 bytes, plus 2 bytes for each character in the filename. (`Filenames` are stored as Unicode characters.) If you anticipate a lot of file activity, you should increase the size of the buffer by setting the `InternalBufferSize` property to a larger value. The size should be an integer multiple of the operating system's page size (4 KB under Microsoft Windows 2000 and later versions). Alternatively, you can use the `NotifyFilter` property to limit the number of change operations that fire the `Changed` event or set `IncludeSubdirectories` to `False` if you don't need to monitor an entire directory tree. (Use multiple `FileSystemWatcher` components to monitor individual subdirectories if you aren't interested in monitoring all the subdirectories under a given path.)

You can't use the `Filter` property to prevent the internal buffer from overflowing because this property filters out files only after they've been added to the buffer. When the internal buffer overflows, you get an `Error` event:

``` FSharp
member this.fsw_Error(sender : Object, e : ErrorEventArgs)  Handles fsw.Error =
   Console.WriteLine("FileSystemWatcher error: {0}", e.GetException().Message)
```

If you notice that your application receives this event, you should change your event handling strategy. For example, you might store all the events in a queue and have them served by another thread.

### Troubleshooting

By default, the `Created`, `Deleted`, `Renamed`, and `Changed` events run in a thread taken from the system thread pool. (See Chapter 20, "`Threads`," for more information about the thread pool.) Because Windows `Forms` controls aren't thread safe, you should avoid accessing any control or the form itself from inside the `FileSystemWatcher` component's event handlers. If you find this limitation unacceptable, you should assign a Windows `Forms` control to the component's `SynchronizingObject` property, as in this code:

``` FSharp
// Use the Form object as the synchronizing object.
fsw.SynchronizingObject = Me
```

The preceding code ensures that all event handlers run in the same thread that serves the form itself. When you create a `FileSystemWatcher` component using the Visual Studio 2005 designer, this property is automatically assigned the hosting form object.

Here are a few more tips about the `FileSystemWatcher` component and the problems you might need to solve when using it:

* The `FileSystemWatcher` component starts raising events when the `Path` property is nonempty and the `EnableRaisingEvents` property is `True`. You can also prevent the component from raising unwanted events during the initialization phase of a Windows `Forms` class by bracketing your setup statements between a call to the `BeginInit` method and a call to the `EndInit` method. (This is the approach used by the Visual Studio designer.)

* As I mentioned before, this component works only on Windows Me, Windows NT, Windows 2000, Windows XP, and Windows Server 2003. It raises an error when it points to a path on machines running earlier versions of the operating system. Remote machines must have one of these operating systems to work properly, but you can't monitor a remote Windows NT system from another Windows NT machine. You can use UNC-based directory names only on Windows 2000 or later systems. The `FileSystemWatcher` component doesn't work on CD-ROM and DVD drives because their contents can't change.

* In some cases, you might get multiple `Created` events, depending on how a file is created and on the application that creates it. For example, when you create a new file using Notepad, you see the following sequence of events: `Created`, `Deleted`, `Created`, and `Changed`. (The first event pair fires because Notepad checks whether the file exists by attempting to create it.)

* A change in a file can generate an extra event in its parent directory as well because the directory maintains information about the files it contains (their size, last write date, and so on).

  

  

* If the directory the `Path` property points to is renamed, the `FileSystemWatcher` component continues to work correctly. However, in this case, the `Path` property returns the old directory name, so you might get an error if you use it. (This happens because the component references the directory by its handle, which doesn't change if the directory is renamed.)

* If you create a directory inside the path being watched and the `IncludeSubdirectories` property is `True`, the new subdirectory is watched as well.

* When a large file is created in the directory, you might not be able to read the entire file immediately because it's still owned by the process that's writing data to it. You should protect any access to the original file with a `Try` block and, if an exception is thrown, attempt the operation again some milliseconds later.

* When the user deletes a file in a directory, a new file is created in the `Recycle` `Bin` directory.