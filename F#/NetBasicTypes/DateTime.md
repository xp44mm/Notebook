## The DateTime Type

使用DateTimeOffset代替

`System.DateTime` is the main .NET class for working with date and time values. Not only does it offer a place to store data values, it also exposes many useful methods ~~that can replace all the Visual Basic—specific date and time functions. For backward compatibility with Visual Basic 6, Visual Basic 2005 lets you use the Date type as a synonym for the System.DateTime type. In this section, I use the Date class name most of the time, but keep in mind that you can always replace it with System.DateTime or just DateTime (because of the projectwide Imports System statement).~~

You can initialize a `DateTime` value in a number of ways:

```FSharp
// Create a Date value by providing year, month, and day values.
let dt: DateTime = new DateTime(2005, 1, 6)       // January 6, 2005

// Provide also hour, minute, and second values.
let dt = new DateTime(2005, 1, 6, 18, 30, 20)     // January 6, 2005 6:30:20 PM

// Add millisecond value (half a second in this example).
let dt = new DateTime(2005, 1, 6, 18, 30, 20, 500)

// Create a time value from ticks (10 million ticks = 1 second).
let ticks: int64 = 20000000L                   // 2 seconds
// This is considered the time elapsed from Jan. 1 of year 1.
let dt = new DateTime(ticks)                   // 1/1/0001 12:00:02 AM
```

~~Because `Date` and `System.DateTime` are synonyms, the following statements are perfectly equivalent:~~

```FSharp
dt = new DateTime(2005, 1, 6)
dt = new DateTime(2005, 1, 6)
```

You can use the `Now` and `Today` static properties:

```FSharp
// The Now property returns the system date and time.
let dt = DateTime.Now        // For example, October 17, 2005 3:54:20 PM
// The Today property returns the system date only.
let dt = DateTime.Today      // For example, October 17, 2005 12:00:00 AM
```

The `UtcNow` static property returns the current time expressed as a Universal Time Coordinate (UTC) value and enables you to compare time values originated in different time zones; this property ignores the Daylight Saving Time if currently active for the current time zone:

```FSharp
let dt = DateTime.UtcNow
```

Once you have an initialized `DateTime` value, you can retrieve individual portions by using one of its read-only properties, namely, `Date` (the date portion), `TimeOfDay` (the time portion), `Year`, `Month`, `Day`, `DayOfYear`, `DayOfWeek`, `Hour`, `Minute`, `Second`, `Millisecond`, and `Ticks`:

```FSharp
// Is today the first day of the current month?
if DateTime.Today.Day = 1 then Console.WriteLine("First day of month")
// How many days have passed since January 1?
Console.WriteLine(DateTime.Today.DayOfYear)
// Get current time – note that ticks are included.
Console.WriteLine(DateTime.Now.TimeOfDay)     // => 10:39:28.3063680
```

The `TimeOfDay` property is peculiar in that it returns a `TimeSpan` object, which represents a difference between dates. Although this class is distinct from the `DateTime` class, it shares many of the `DateTime` class properties and methods and nearly always works together with `DateTime` values, as you'll see shortly.

A note for the curious programmer: a `DateTime` value is stored as the number of ticks (1 tick = 100 nanoseconds) elapsed since January 1, 0001; this storage format can work for any date between 1/1/0001 and 12/12/9999. In .NET Framework 2.0, this tick value takes 62 bits, and the remaining 2 bits are used to preserve the information whether the date/time is in Daylight Saving Time and whether the date/time is relative to the current time zone (the default) or is expressed as a UTC value.

### Adding and Subtracting Dates

The `DateTime` class exposes several instance methods that enable you to add and subtract a number of years, months, days, hours, minutes, or seconds to or from a `DateTime` value. The names of these methods leave no doubt about their function: `AddYears`, `AddMonths`, `AddDays`, `AddHours`, `AddMinutes`, `AddSeconds`, `AddMilliseconds`, `AddTicks`. You can add an integer value when you're using `AddYears` and `AddMonths` and a decimal value in all other cases. In all cases, you can pass a negative argument to subtract rather than add a value:

```FSharp
// Tomorrow's date
let dt = DateTime.Today.AddDays(1.)
// Yesterday's date
let dt = DateTime.Today.AddDays(-1.)
// What time will it be 2 hours and 30 minutes from now?
let dt = DateTime.Now.AddHours(2.5)

// A CPU-intensive way to pause for 5 seconds.
let endTime: DateTime = DateTime.Now.AddSeconds(5.)
while DateTime.Now <= endTime do
    ()
```

The `Add` method takes a `TimeSpan` object as an argument. Before you can use it, you must learn to create a `TimeSpan` object, choosing one of its overloaded constructor methods:

```FSharp
// One Long value is interpreted as a Ticks value.
let ts: TimeSpan = new TimeSpan(13500000L)       // 1.35 seconds
// Three integer values are interpreted as hours, minutes, seconds.
let ts = new TimeSpan(0, 32, 20)                 // 32 minutes, 20 seconds

// Four integer values are interpreted as days, hours, minutes, seconds.
let ts = new TimeSpan(1, 12, 0, 0)               // 1 day and a half
// (Note that arguments aren't checked for out-of-range errors; therefore,
// the next statement delivers the same result as the previous one.)
let ts = new TimeSpan(0, 36, 0, 0)               // 1 day and a half
// A fifth argument is interpreted as a millisecond value.
let ts = new TimeSpan(0, 0, 1, 30, 500)          // 90 seconds and a half
```

Now you're ready to add an arbitrary date or time interval to a `DateTime` value:

```FSharp
// What will be the time 2 days, 10 hours, and 30 minutes from now?
let dt = DateTime.Now.Add(new TimeSpan(2, 10, 30, 0))
```

The `DateTime` class also exposes a `Subtract` instance method that works in a similar way:

```FSharp
// What was the time 1 day, 12 hours, and 20 minutes ago?
let dt = DateTime.Now.Subtract(new TimeSpan(1, 12, 20, 0))
```

The `Subtract` method is overloaded to take another `DateTime` object as an argument, in which case it returns the `TimeSpan` object that represents the difference between the two dates:

```FSharp
// How many days, hours, minutes, and seconds have elapsed
// since the beginning of the third millennium?
let startDate = new DateTime(2001, 1, 1)
let diff: TimeSpan = DateTime.Now.Subtract(startDate)
```

Once you have a `TimeSpan` object, you can extract the information buried in it by using one of its many properties, whose names are self-explanatory: `Days`, `Hours`, `Minutes`, `Seconds`, `Milliseconds`, `Ticks`, `TotalDays`, `TotalHours`, `TotalMinutes`, `TotalSeconds`, and `TotalMilliseconds`. The `TimeSpan` class also exposes methods such as `Add`, `Subtract`, `Negate`, and `CompareTo`.

The `CompareTo` method enables you to determine whether a `DateTime` value is greater or less than another `DateTime` value:

```FSharp
// Is current date later than October 30, 2005?
match DateTime.Today.CompareTo(new DateTime(2005, 10, 30)) with
| 1   -> () // Later than Oct. 30, 2005
| -1  -> () // Earlier than Oct. 30, 2005
| 0   -> () // Today is Oct. 30, 2005.
| _ -> ()
```

And you can use comparison operators if you don't need three-state logic:

```FSharp
if DateTime.Today > new DateTime(2005, 10, 30) then …
```

By default, `DateTime` values are relative to the current time zone and you should never compare values coming from different time zones, unless they are in UTC format (see the section titled "Working with Time Zones" later in this chapter). Also, when evaluating the difference between two dates in the same time zone, you might get a wrong result if a transition to or from Daylight Saving Time has occurred between the two dates. This is one more reason to use dates in UTC format.

The `IsDaylightSavingTime` method ~~(new in .NET Framework 2.0)~~ enables you to detect whether Daylight Saving Time is active for the current time zone:

```FSharp
if DateTime.Now.IsDaylightSavingTime() then Console.Write("Daylight Saving Time is active")
```

Finally, the `DateTime` class exposes two static methods that can be handy in many applications:

```FSharp
// Test for a leap year.
Console.WriteLine(DateTime.IsLeapYear(2000))           // => True
// Retrieve the number of days in a given month.
Console.WriteLine(DateTime.DaysInMonth(2000, 2))       // => 29
```

In spite of the abundance of date and time methods, the `DateTime` type doesn't offer a simple way to calculate the whole number of years or months elapsed between two dates. For example, you can't calculate the age of a person using this statement:

```FSharp
let age: Int32 = DateTime.Now.Year - aPerson.BirthDate.Year
```

because the result would be one unit greater than the correct value if the person hasn't celebrated her birthday in the current year. I have prepared a couple of reusable routines that provide the missing functionality:

```FSharp
// Return the whole number of years between two dates.
let YearDiff(startDate : DateTime, endDate : DateTime) : Int32 =
    let result: Int32 = endDate.Year - startDate.Year
    if endDate.Month < startDate.Month || (endDate.Month = startDate.Month && endDate.Day < startDate.Day) then 
        result - 1
    else
        result

// Return the whole number of months between two dates.
let MonthDiff(startDate : DateTime, endDate : DateTime) : Int32 =
    let result: Int32 = endDate.Year * 12 + endDate.Month - (startDate.Year * 12 + startDate.Month)
    if endDate.Month = startDate.Month && endDate.Day < startDate.Day then 
        result - 1
    else
        result
```

### Formatting Dates

The `DateTime` type overrides the `ToString` method to accept a standard date format among those specified in Table 12-2 or a user-defined format created by assembling the characters listed in Table 12-3:

```FSharp
// This is January 6, 2005 6:30:20.500 PM-U.S. Eastern Time.
let dt: DateTime = new DateTime(2005, 1, 6, 18, 30, 20, 500)

// Display a date using the LongDatePattern standard format.
let dateText: String = dt.ToString("D")     // => Thursday, January 06, 2005
// Display a date using a custom format.
let dateText = dt.ToString("d-MMM-yyyy")    // => 6-Jan-2005
```

You can format a `DateTime` value in other ways by using some peculiar methods that only this type exposes:

```FSharp
Console.WriteLine(dt.ToShortDateString())     // => 1/6/2005
Console.WriteLine(dt.ToLongDateString())      // => Thursday, January 06, 2005
Console.WriteLine(dt.ToShortTimeString())     // => 6:30 PM
Console.WriteLine(dt.ToLongTimeString())      // => 6:30:20 PM
Console.WriteLine(dt.ToFileTime())            // => 127495062205000000
Console.WriteLine(dt.ToOADate())              // => 38358.7710706019
// The next two results vary depending on the time zone you're in.
Console.WriteLine(dt.ToUniversalTime())       // => 1/7/2005 12:30:20 PM
Console.WriteLine(dt.ToLocalTime())           // => 1/6/2005 12:30:20 PM
```

A few of these formats might require additional explanation:

- The `ToFileTime` method returns an unsigned 8-byte value representing the date and time as the number of 100-nanosecond intervals that have elapsed since 1/1/1601 12:00 A.M. The `DateTime` type also supports the `ToFileTimeUtc` method, which ignores the local time zone.
- The `ToOADate` method converts to an OLE Automation–compatible value. (This is a `Double` value ~~similar to the Date values used in Visual Basic 6.~~)
- The `ToUniversalTime` method considers the `DateTime` value a local time and converts it to UTC format.
- The `ToLocalTime` method considers the `DateTime` value a UTC value and converts it to a local time.

The `DateTime` class exposes two static methods, `FromOADate` and `FromFileTime`, to parse an OLE Automation date value or a date formatted as a file time.

### Parsing Dates

The operation complementary to date formatting is date parsing. The `DateTime` class provides a `Parse` static method for parsing jobs of any degree of complexity:

```FSharp
let dt: DateTime = DateTime.Parse("2005/1/6 12:30:20")
```

The flexibility of this method becomes apparent when you pass an `IFormatProvider` object as a second argument to it—for example, a `CultureInfo` object or a `DateTimeFormatInfo` object. The `DateTimeFormatInfo` object is conceptually similar to the `NumberFormatInfo` object described earlier in this chapter (see the section titled "Formatting Numbers" earlier in this chapter), except it holds information about separators and formats allowed in date and time values:

```FSharp
// Get a writable copy of the current locale's DateTimeFormatInfo object.
let dtfi: DateTimeFormatInfo = DateTimeFormatInfo.CurrentInfo.Clone() :?> DateTimeFormatInfo
// Change date and time separators.
dtfi.DateSeparator <- "-"
dtfi.TimeSeparator <- "."
// Now we're ready to parse a date formatted in a nonstandard way.
let dt = DateTime.Parse("2005-1-6 12.30.20", dtfi)
```

Many non-U.S. developers will appreciate the ability to parse dates in formats other than month/day/year. In this case, you have to assign a correctly formatted pattern to the `DateTimeFormatInfo` object's `ShortDatePattern`, `LongDatePattern`, `ShortTimePattern`, `LongTimePattern`, or `FullDateTimePattern` property before doing the parse:

```FSharp
// Prepare to parse (dd/mm/yy) dates, in short or long format.
dtfi.ShortDatePattern <- "d/M/yyyy"
dtfi.LongDatePattern <- "dddd, dd MMMM, yyyy"

// Both these statements assign the date "January 6, 2005."
let dt = DateTime.Parse("6-1-2005 12.30.44", dtfi)
let dt = DateTime.Parse("Thursday, 6 January, 2005", dtfi)
```

You can use the `DateTimeFormatInfo` object to retrieve standard or abbreviated names for weekdays and months, according to the current locale or any locale:

```FSharp
// Display the abbreviated names of months.
for s: String in DateTimeFormatInfo.CurrentInfo.AbbreviatedMonthNames do
   Console.WriteLine(s)
```

Even more interesting, you can set weekday and month names with arbitrary strings if you have a writable `DateTimeFormatInfo` object, and then you can use the object to parse a date written in any language, including invented ones. (Yes, including Klingon!)

Another way to parse strings in formats other than month/day/year is to use the `ParseExact` static method. In this case, you pass the format string as the second argument, and you can pass `null` to the third argument if you don't need a `DateTimeFormatInfo` object to further qualify the string being parsed:

```FSharp
// This statements assigns the date "January 6, 2005."
dt = DateTime.ParseExact("6-1-2005", "d-M-yyyy", null)
```

The second argument can be any of the supported `DateTime` format listed in Table 12-2. In .NET Framework 2.0, the new format `F` has been added to support the `ParseExact` method when there are a variable number of fractional digits.

**Version 2005 of VB or Version 2.0 of .NET** Both the `Parse` and `ParseExact` methods throw an exception if the input string doesn't comply with the expected format. As you know, exceptions can add a lot of overhead to your applications and you should try to avoid them if possible. Version 2.0 of the .NET Framework extends the `DateTime` class with the `TryParse` and the `TryParseExact` methods, which return true if the parsing succeeds and store the result of the parsing in a `DateTime` variable passed a second argument:

```FSharp
let mutable aDate: DateTime = DateTime.Now
if DateTime.TryParse("January 6, 2005", &aDate) then
    // aDate contains the parsed date.
```

Another overload of the `TryParse` method takes an `IFormatProvider` object (for example, a `CultureInfo` instance) and a `DateTimeStyles` bit-coded value; the latter argument enables you to specify whether leading or trailing spaces are accepted and whether local or universal time is assumed:

```FSharp
let ci = new CultureInfo("en-US")
if DateTime.TryParse(" 6/1/2005 14:26 ", ci, DateTimeStyles.AllowWhiteSpaces ||| DateTimeStyles.AssumeUniversal, &aDate) then
    // aDate contains the parsed date.
```

If you specify the `DateTimesStyles.AssumeUniversal` enumerated value ~~(new in .NET Framework 2.0)~~, the parsed time is assumed to be in UTC format and is automatically converted to the local time zone. By default, date values are assumed to be relative to the current time zone.

### Working with Time Zones

TimeZone已经被TimeZoneInfo完全取代。https://docs.microsoft.com/en-us/dotnet/api/system.timezoneinfo?view=netframework-4.8

**Version 2005 of VB or Version 2.0 of .NET** `DateTime` values in version 1.1 of the .NET Framework have a serious limitation: they are always expected to store a local time, rather than a normalized UTC time. This assumption causes a few hard-to-solve issues, the most serious of which is a problem that manifests itself when a date value is serialized in one time zone and deserialized in a different zone, using either the `SoapFormatter` or the `XmlSerializer` object. These two objects, in fact, store information about the time zone together with the actual date value: when the object is deserialized in a different time zone, the time portion of the date is automatically adjusted to reflect the new geographical location.

In most cases, this behavior is correct, but at times it causes the application to malfunction. Let's say that a person is born in Italy on January 1, 1970, at 2 A.M.; if this date value is serialized as XML and sent to a computer in New York—for example, by using a Web service or by saving the information in a file that is later transferred using FTP or HTTP—the person would appear to be born on December 31, 1969, at 8 P.M. As you can see, the issue with dates in .NET Framework 1.1 originates from the fact that you can't specify whether a value stored in a `DateTime` variable is to be considered relative to the current time zone or an absolute UTC value.

This problem has been solved quite effectively in .NET Framework 2.0 by the addition of a new `Kind` property to the `DateTime` type. This property is a `DateTimeKind` enumerated value that can be `Local`, `Utc`, or `Unspecified`. For backward compatibility with .NET Framework 1.1 applications, by default a `DateTime` value has a `Kind` property set to `DateTimeKind.Local`, unless you specify a different value in the constructor:

```FSharp
// February 14, 2005 at 12:00 AM, UTC value
let aDate = new DateTime(2005, 2, 14, 12, 0, 0, DateTimeKind.Utc)
// Test the Kind property.
Console.WriteLine(aDate.Kind.ToString())          // Utc
```

The `Kind` property is read-only, but you can use the `SpecifyKind` static method to create a different `DateTime` value if you want to pass from local to UTC time or vice versa:

```FSharp
// Next statement changes the Kind property (but doesn't change the date/time value!).
let newDate: DateTime = DateTime.SpecifyKind(aDate, DateTimeKind.Utc)
```

An important note: the `Kind` property is accounted for only when serializing and deserializing a date value and is ignored when doing comparisons.

In .NET Framework 1.1, `DateTime` values are serialized as 64-bit numbers by means of the `Ticks` property. In .NET Framework 2.0, however, when saving a `DateTime` value to a file or a database field, you should save the new `Kind` property as well; otherwise, the deserialization mechanism would suffer from the same problems you see in .NET Framework 1.1. The simplest way to do so is by means of the new `ToBinary` instance method (which converts the `DateTime` object to a 64-bit value) and the new `FromBinary` static method (which converts a 64-bit value to a `DateTime` value):

```FSharp
// Convert to a Long value.
let lngValue: int64 = aDate.ToBinary()

// Convert back from a Long to a Date value.
let newDate: DateTime = DateTime.FromBinary(lngValue)
```

You can also serialize a `DateTime` value as text. In this case, you should use the `ToString` method with the `o` format ~~(new in .NET Framework 2.0)~~. This format serializes all the information related to a date, including the `Kind` property and the time zone (if the date isn't in UTC format), and you can read it back by means of a `ParseExact` method if you specify the new `DateTimeStyles.RoundtripKind` enumerated value:

```FSharp
// Serialize a date in UTC format.
let text: String = aDate.ToString("o", CultureInfo.InvariantCulture)

// Deserialize it into a new DateTime value.
let newDate = DateTime.ParseExact(text, "o", CultureInfo.InvariantCulture, DateTimeStyles.RoundtripKind)
```

### The TimeZone Type

The .NET Framework supports time zone information through the `System.TimeZoneInfo` object, which you can use to retrieve information about the time zone set in Windows regional settings:

```FSharp
// Get the TimeZone object for the current time zone.
//let tz: TimeZone = TimeZone.CurrentTimeZone
let tz: TimeZoneInfo = TimeZoneInfo.Local
// Display name of time zone, without and with Daylight Saving Time.
// (I got these results by running this code in Italy.)
Console.WriteLine(tz.StandardName)  // => W. Europe Standard Time
Console.WriteLine(tz.DaylightName)  // => W. Europe Daylight Time
```

The most interesting piece of information here is the offset from UTC format, which you retrieve by means of the `GetUTCOffset` method. You must pass a date argument to this method because the offset depends on whether Daylight Saving Time is in effect. The returned value is in ticks:

```FSharp
// Display the time offset of W. Europe time zone in March 2005,
// when no Daylight Saving Time is active.
let o = tz.GetUtcOffset(new DateTime(2005, 3, 1))
Console.WriteLine(o)  // => 01:00:00
// Display the time offset of W. Europe time zone in July,
// when Daylight Saving Time is active.
let o = tz.GetUtcOffset(new DateTime(2005, 7, 1))
Console.WriteLine(o)  // => 02:00:00
```

The `IsDaylightSavingTime` method returns True if Daylight Saving Time is in effect:

```FSharp
// No Daylight Saving Time in March
Console.WriteLine(tz.IsDaylightSavingTime(new DateTime(2005, 3, 1)))
// => False
```

Finally, you can determine when Daylight Saving Time starts and ends in a given year by retrieving an array of `DaylightTime` objects with the `TimeZone`'s `GetDaylightChanges` method:

```FSharp
// Retrieve the DaylightTime object for year 2005.
//let dlc: DaylightTime = tz.GetDaylightChanges(2005)
let adjustments = tz.GetAdjustmentRules()
for dlc in adjustments do
    // Note that you might get different start and end dates if you
    // run this code in a country other than the United States.
    Console.WriteLine("Starts at {0}, Ends at {1}, Delta is {2} minutes", dlc.DateStart, dlc.DateEnd, dlc.DaylightDelta.TotalMinutes)
       // => Starts at 3/27/2005 2:00:00 A.M., ends at 10/30/2005 3:00:00 A.M., Delta is 60 minutes.
```

### The Guid Type

The `System.Guid` type exposes several static and instance methods that can help you work with globally unique identifiers (GUIDs), that is, those 128-bit numbers that serve to uniquely identify elements and that are ubiquitous in Windows programming. The `NewGuid` static method is useful for generating a new unique identifier:

```FSharp
// Create a new GUID.
let guid1: Guid = Guid.NewGuid()
```

If you already have a GUID—for example, a GUID you have read from a database field—you can initialize a `Guid` variable by passing the GUID representation as a string or as an array of bytes to the type's constructor:

```FSharp
// Initialize from a string.
let guid2 = new Guid("f16f990e-dd16-43af-a5ea-6340c47b32b0")
```

There are only two more things you can do with a `Guid` object: you can convert it to a Byte array with the `ToByteArray` method, and you can compare two `Guid` values for equality using the `Equals` method (inherited from `System.Object`):

```FSharp
// Convert to an array of bytes.
let bytes: byte[] = guid1.ToByteArray()
for b: byte in bytes do
   Console.Write("{0} ", b)

// Compare two GUIDs.
if guid1.Equals(guid2) then
   Console.WriteLine("GUIDs are same.")
else
   Console.WriteLine("GUIDs are different.")
```

