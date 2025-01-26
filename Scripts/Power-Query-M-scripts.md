
# Power Query Scripts for Tokyo Olympics Data Analysis

## Power Query Medals Data
This Power Query (M) script is used to transform medals data.

```pq
let 
    // Primary Work: Open load, remove unnecessary rows, and promote the header
    Source = Excel.Workbook(File.Contents("C:\My Power BI Projects\TOKYO OLYMPICS\Dataset\Tokyo Olympic Data.xlsx"),
             null, true),
    #"Tokyo Olympic_Sheet" = Source{[Item="Tokyo Olympic",Kind="Sheet"]}[Data],
    #"Removed Top Rows" = Table.Skip(#"Tokyo Olympic_Sheet",4),
    #"Trimmed Text" = Table.TransformColumns(Table.TransformColumnTypes(#"Removed Top Rows", {{"Column3", type text}, 
        {"Column4", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type text}}, "en-US"),
        {{"Column1", Text.Trim, type text}, {"Column2", Text.Trim, type text}, {"Column3", Text.Trim, type text}, 
        {"Column4", Text.Trim, type text}, {"Column5", Text.Trim, type text}, {"Column6", Text.Trim, type text}, 
        {"Column7", Text.Trim, type text}}),
    #"Promoted Headers" = Table.PromoteHeaders(#"Trimmed Text", [PromoteAllScalars=true]),
    
    // Change data type
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Rank", Int64.Type}, {"Team/NOC", type text}, 
        {"Gold", Int64.Type}, {"Silver", Int64.Type}, {"Bronze", Int64.Type}, {"Total", Int64.Type}, {"Rank by Total",
             Int64.Type}}),
    
    // Keep only required columns: "Team/NOC", "Gold", "Silver", "Bronze"
    #"Removed Other Columns" = Table.SelectColumns(#"Changed Type",{"Team/NOC", "Gold", "Silver", "Bronze"}),
    
    // Created new column for medals combinations
    #"Added Medals Combination" = Table.AddColumn(#"Removed Other Columns", "Medals Combination", 
        each if [Silver]=0 and [Bronze]=0 then "Gold-Only"
        else if [Gold]=0 and [Bronze]=0 then "Silver-Only"
        else if [Gold]=0 and [Silver]=0 then "Bronze-Only"
        else if [Bronze]=0 then "Gold-Silver"
        else if [Silver]=0 then "Gold-Bronze"
        else if [Gold]=0   then "Silver-Bronze"
        else "All-Medals", type text),
    
    // Created new columns for winners depending on medal types they won
    #"Added Winners | Medals" = Table.AddColumn(#"Added Medals Combination", "Winners | Medals", 
        each if [Medals Combination] = "All-Medals" then "3 Types" 
        else if Text.Contains([Medals Combination], "Only") then "1 Type" 
        else "2 Types", type text),
    
    // Unpivot Gold, Silver, and Bronze columns for filtering
    #"Unpivoted Other Columns" = Table.UnpivotOtherColumns(#"Added Winners | Medals", 
        {"Team/NOC","Medals Combination","Winners | Medals"}, "Medal Type", "Total"),
    
    // Remove data when the number of medals is zero
    #"Filtered Rows" = Table.SelectRows(#"Unpivoted Other Columns", each ([Total] <> 0))
in
    #"Filtered Rows"
```

## Power Query Country-Region
This Power Query (M) script is used to transform the country-region data.

```pq
let
    // Created this Excel sheet using Country-region-all sheet in temporary PowerBI file and drew table by 
    // Team/NOC from Tokyo Olympic Data and Country & Region from this sheet.
    // Now all data is available and placed in this file.

    Source = Excel.Workbook(File.Contents("C:\My Power BI Projects\TOKYO OLYMPICS\Dataset\Country-region.xlsx"), null, true),
    #"Country-region_Sheet" = Source{[Item="Country-region",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(#"Country-region_Sheet", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers",{{"Team/NOC", type text}, {"region", type text}}),

    // Handling countries like Great Britain that required manual adjustments for map compatibility.
    #"Added Custom" = Table.AddColumn(#"Changed Type1", "Country", each if [#"Team/NOC"] = "Great Britain" then "United Kingdom" 
        else [#"Team/NOC"], type text)
in
    #"Added Custom"
```

## Power Query Medal Ranking
This Power Query (M) script is used to transform the medal ranking data.

```pq
let
    // This is a copy of the Medals Data Table but using columns removed before in this new table

    Source = Excel.Workbook(File.Contents("C:\My Power BI Projects\TOKYO OLYMPICS\Dataset\Tokyo Olympic Data.xlsx"), null, true),
    #"Tokyo Olympic_Sheet" = Source{[Item="Tokyo Olympic",Kind="Sheet"]}[Data],
    #"Removed Top Rows" = Table.Skip(#"Tokyo Olympic_Sheet",4),
    #"Trimmed Text" = Table.TransformColumns(Table.TransformColumnTypes(#"Removed Top Rows", {{"Column3", type text},
        {"Column4", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type text}}, "en-US"),
        {{"Column1", Text.Trim, type text}, {"Column2", Text.Trim, type text}, {"Column3", Text.Trim, type text},
        {"Column4", Text.Trim, type text}, {"Column5", Text.Trim, type text}, {"Column6", Text.Trim, type text},
        {"Column7", Text.Trim, type text}}),
    #"Promoted Headers" = Table.PromoteHeaders(#"Trimmed Text", [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Rank", Int64.Type}, {"Team/NOC", type text},
        {"Gold", Int64.Type}, {"Silver", Int64.Type}, {"Bronze", Int64.Type}, {"Total", Int64.Type}, {"Rank by Total", Int64.Type}}),
    
    // Keep columns "Rank", "Team/NOC", "Total", "Rank by Total"
    #"Removed Other Columns" = Table.SelectColumns(#"Changed Type",{"Rank", "Team/NOC", "Total", "Rank by Total"}),

    // Rename Rank to be Rank by Gold to avoid confusion, as rank depends primarily on Gold
    #"Renamed Columns" = Table.RenameColumns(#"Removed Other Columns",{{"Rank", "Rank by Gold"}}),

    // Unpivot the two ranking methods to be used as slicers in the dashboard
    #"Unpivoted Columns" = Table.UnpivotOtherColumns(#"Renamed Columns", {"Team/NOC", "Total"}, "Ranking", "Value")
in
    #"Unpivoted Columns"
```

## Power Query for Medal Type
This Power Query (M) script is used to transform data for medal types.

```pq
let
    // Created this manual table just to order medals by Gold, Silver, then Bronze
    // Depending on alphabetical order, it would have been incorrect

    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("i45Wcs/PSVHSUTJUitWJVgrOzClLLQJyjcBcp6L8vKpUINdYKTYWAA==", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [#"Medal Type" = _t, Rank = _t]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Rank", Int64.Type}})
in
    #"Changed Type"
```

## Power Query for Medal Combination
This Power Query (M) script is used to transform data for medal combinations.

```pq
let
    // Created this manual table just to order medals combinations by Gold, Silver, then Bronze in each group
    // All-Medal: 
    // Two-Medals: Gold-Silver, Gold-Bronze, then Silver-Bronze
    // One Medal: Gold, Silver, then Bronze
    // Depending on alphabetical order, it would have been incorrect

    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("i45WcszJ0fVNTUnMKVbSUTJUitWJVnLPz0nRDc7MKUstAooZIcScivLzqlKBYsZgMYgShKgJQqV/Xk4lUMQUWR1UzAwsBtEEEzNXio0FAA==", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [#"Medals Combination" = _t, Rank = _t]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Rank", Int64.Type}})
in
    #"Changed Type"
```

