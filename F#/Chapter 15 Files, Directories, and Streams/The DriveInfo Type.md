## The DriveInfo Type

**Version 2005 of VB or Version 2.0 of  .NET** Previous versions of the .NET Framework expose no classes for  retrieving information about existing drives, and thus you must use either  PInvoke calls to the Windows API or Windows Management Instrumentation (WMI)  classes. This gap has been filled in version 2.0 with the introduction of the  DriveInfo type.





You can create a DriveInfo object in two ways: by passing a drive  letter to its constructor or by means of the GetDrives static method, which  returns an array containing information about all the installed drives:

```
' Display the volume label of drive C.
Dim driveC As New DriveInfo("c:")
Console.WriteLine(driveC.VolumeLabel)
```

When enumerating drives, it's crucial that you don't attempt to  read any member before testing the IsReady property:

```
' Display name and total size of all available drives.
For Each di As DriveInfo In DriveInfo.GetDrives()
   If di.IsReady Then
      Console.WriteLine("{0} {1:N}", di.Name, di.TotalSize)
   End If
Next
```

The DriveInfo object exposes the following properties: Name,  VolumeLabel, RootDirectory (the DirectoryInfo object that represents the root  folder), DriveType (an enumerated value that can be Fixed, Removable, CDRom,  Ram, Network, and Unknown), DriveFormat (a string such as NTFS or FAT32),  TotalSize (the capacity of the drive in bytes), Total-FreeSpace (the total  number of free bytes), and AvailableFreeSpace (the number of available free  bytes; can be less than TotalFreeSpace if quotas are used). All the properties  are read-only, which is quite understandable (even though I'd surely like to  increase the amount of free space on a drive by simply setting a property!),  except for the VolumeLabel property:

```
' Change the volume label of drive D.
Dim driveD As New DriveInfo("d:")
driveD.VolumeLabel = "MyData"
```

The following loop displays in a tabular format information about  all the installed drives, while skipping over the drives that aren't ready:

```
Console.WriteLine("{0,-6}{1,-10}{2,-8}{3,-16}{4,18}{5,18}", _
   "Name", "Label", "Type", "Format", "TotalSize", "TotalFreeSpace")
Console.WriteLine(New String("-"c, 78))
For Each di As DriveInfo In DriveInfo.GetDrives()
   If di.IsReady Then
      Console.WriteLine("{0,-6}{1,-10}{2,-8}{3,-16}{4,18:N0}{5,18:N0}", _
         di.Name, di.VolumeLabel, di.DriveType.ToString, di.DriveFormat, _
         di.TotalSize, di.TotalFreeSpace)
   Else
      Console.WriteLine("{0,-6}(not ready)", di.Name)
   End If
Next
```

Here's an example of what you might see in the console window:

```
Name  Label     Type    Format                    TotalSize   TotalFreeSpace
------------------------------------------------------------------------------
C:\             Fixed   NTFS                 20,974,428,160    8,009,039,872
D:\   DATA      Fixed   NTFS                 39,028,953,088   10,244,005,888
E:\   (not ready)
```