# PowerBI Reference

## Common Dimension Tables

<details>
  <summary> dimRefreshDateTime </summary>
  
  ### dimRefreshDateTime
  
```dimRefreshDateTime
let
    UTC_DateTimeZone = DateTimeZone.UtcNow(),
    UTC_Date = Date.From(UTC_DateTimeZone),
    StartOfSummerTime = Date.StartOfWeek( #date( Date.Year( UTC_Date ), 3, 31 ), Day.Sunday ),
    StartOfWinterTime = Date.StartOfWeek( #date( Date.Year( UTC_Date ), 10, 31 ), Day.Sunday ),
    Offset = if UTC_Date >= StartOfSummerTime and UTC_Date <= StartOfWinterTime then -5 else -6,
    CST_DateTimeZone = DateTimeZone.SwitchZone(UTC_DateTimeZone, Offset),
    #"Convert to table" = Table.FromValue(CST_DateTimeZone),
    #"Renamed columns" = Table.RenameColumns(#"Convert to table", {{"Value", "Refresh"}})
in
    #"Renamed columns"
```
</details>
      
<details>
  <summary> StartingDate </summary>
  
  ### StartingDate
  
_Created as a Parameter_
```StartingDate
#date(2022, 1, 1) meta [IsParameterQuery = true, IsParameterQueryRequired = true, Type = type date]
```
</details>

<details>
  <summary> dimDate </summary>
  
### dimDate
  
```dimdate
let
  // Start Date pulling from parameter set after confirming earliest date in all tables.
  StartDate = StartingDate,
  EndDate = Today,
  Today = Date.EndOfYear(DateTime.Date(DateTime.LocalNow())),
  Duration = Duration.Days(Duration.From(Today-StartDate))+1,
  // Date Columns
  Dates = List.Dates(StartDate, Duration, #duration(1, 0, 0, 0)),
  #"Converted to Table" = Table.FromList(Dates, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
  #"Renamed Columns" = Table.RenameColumns(#"Converted to Table", {{"Column1", "Date"}}),
  #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns", {{"Date", type date}}),
  #"Inserted Year" = Table.AddColumn(#"Changed Type", "Year", each Date.Year([Date]), Int64.Type),
  #"Inserted Quarter" = Table.AddColumn(#"Inserted Year", "Quarter#", each Date.QuarterOfYear([Date]), Int64.Type),
  #"Format Quarter" = Table.AddColumn(#"Inserted Quarter", "Quarter", each "Q" & Text.From([#"Quarter#"])),
  #"Inserted Month" = Table.AddColumn(#"Format Quarter", "Month#", each Date.Month([Date]), Int64.Type),
  #"Format Month" = Table.AddColumn(#"Inserted Month", "Month Name", each Text.Start(Date.MonthName([Date]),3), type text),
  #"Inserted Week of Year" = Table.AddColumn(#"Format Month", "Week of Year", each Date.WeekOfYear([Date]), Int64.Type),
  #"Format Month Year" = Table.AddColumn(#"Inserted Week of Year", "MonthYear", each Date.ToText([Date],"MMM") & "-" & Text.From(Date.Year([Date]))),
  #"Duplicated column" = Table.DuplicateColumn(#"Format Month Year", "Date", "Day"),
  #"Extracted day" = Table.TransformColumns(#"Duplicated column", {{"Day", each Date.Day(_), type nullable number}}),
  #"Format Quarter Year" = Table.AddColumn(#"Extracted day", "QuarterYear", each "Q"&Text.From([#"Quarter#"]) & "-" & Text.From(Date.Year([Date]))),
  #"Transform columns" = Table.TransformColumnTypes(#"Format Quarter Year", {{"Quarter", type text}, {"MonthYear", type text}, {"QuarterYear", type text}}),
  #"Added custom" = Table.TransformColumnTypes(Table.AddColumn(#"Transform columns", "IsCurrentYear", each Date.IsInCurrentYear([Date])), {{"IsCurrentYear", type logical}}),
  #"Added custom 1" = Table.TransformColumnTypes(Table.AddColumn(#"Added custom", "YearMonthSort", each ([Year]*100)+[#"Month#"]), {{"YearMonthSort", Int64.Type}})
in
  #"Added custom 1"
```
</details>
      
