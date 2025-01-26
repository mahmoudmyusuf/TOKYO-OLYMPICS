# Power Query Medals Data
This Power Query (M) script used to transform For Medals Data.

```pq
let 
    // Primary Work, Open Load, Remove unnecessary Rows, and Promte the header
    Source = Excel.Workbook(File.Contents("C:\Users\opr1141\OneDrive\My Power BI Projects\TOKYO OLYMPICS\Dataset\Tokyo Olympic Data.xlsx"), null, true),
    #"Tokyo Olympic_Sheet" = Source{[Item="Tokyo Olympic",Kind="Sheet"]}[Data],
    #"Removed Top Rows" = Table.Skip(#"Tokyo Olympic_Sheet",4),
    #"Trimmed Text" = Table.TransformColumns(Table.TransformColumnTypes(#"Removed Top Rows", {{"Column3", type text}, 
        {"Column4", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type text}}, "en-US"),
        {{"Column1", Text.Trim, type text}, {"Column2", Text.Trim, type text}, {"Column3", Text.Trim, type text}, 
        {"Column4", Text.Trim, type text}, {"Column5", Text.Trim, type text}, {"Column6", Text.Trim, type text}, 
        {"Column7", Text.Trim, type text}}),
    #"Promoted Headers" = Table.PromoteHeaders(#"Trimmed Text", [PromoteAllScalars=true]),
    
    // Change data Type 
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Rank", Int64.Type}, {"Team/NOC", type text}, 
        {"Gold", Int64.Type}, {"Silver", Int64.Type}, {"Bronze", Int64.Type}, {"Total", Int64.Type}, {"Rank by Total", Int64.Type}}),
    
    // Keeps Only required Columns "Team/NOC", "Gold", "Silver", "Bronze"
    #"Removed Other Columns" = Table.SelectColumns(#"Changed Type",{"Team/NOC", "Gold", "Silver", "Bronze"}),
    
    // Created new Column for Medals Combinations
    #"Added Medals Combination" = Table.AddColumn(#"Removed Other Columns", "Medals Combination", 
        each if [Silver]=0 and [Bronze]=0 then "Gold-Only"
        else if [Gold]=0 and [Bronze]=0 then "Silver-Only"
        else if [Gold]=0 and [Silver]=0 then "Bronze-Only"
        else if [Bronze]=0 then "Gold-Silver"
        else if [Silver]=0 then "Gold-Bronze"
        else if [Gold]=0   then "Silver-Bronze"
        else "All-Medals", type text),
    
    // Created new columns for Winners depending on Medals Type they won
    #"Added Winners | Medals" = Table.AddColumn(#"Added Medals Combination", "Winners | Medals", 
        each if [Medals Combination] = "All-Medals" then "3 Types" 
        else if Text.Contains([Medals Combination], "Only") then "1 Types" 
        else "2 Type", type text),
    
    // Unpivot Gold, Silver, and Bronze Columns to Make them good for filtering
    #"Unpivoted Other Columns" = Table.UnpivotOtherColumns(#"Added Winners | Medals", 
        {"Team/NOC","Medals Combination","Winners | Medals"}, "Medal Type", "Total"),
    
    // Remove Data when number of Medals is Zero
    #"Filtered Rows" = Table.SelectRows(#"Unpivoted Other Columns", each ([Total] <> 0))
in
    #"Filtered Rows"

# Power Query Country-region
This Power Query (M) script used to transform For Country-region.

```pq
let
    // Created this Excel Sheet by using Country-region-all sheet in temporary PowerBI file and Draw table by 
    // Team/NOC from Tokyo Olympic Data and Country & Region From this Sheet
    // Get a table with all Region except about 6 coutries. They are done manually using ChatGPT
    // Now All Data are Avaiable and put them in this File

    Source = Excel.Workbook(File.Contents("C:\Users\opr1141\OneDrive\My Power BI Projects\TOKYO OLYMPICS\Dataset\Country-region.xlsx"), null, true),
    #"Country-region_Sheet" = Source{[Item="Country-region",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(#"Country-region_Sheet", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers",{{"Team/NOC", type text}, {"region", type text}}),

    // Using the data in Maps and do Some Manual check for countries. All of them is OK Except Great Britain So created this Column
    // To Keep original name but Draw with name the Map can understand
    
    #"Added Custom" = Table.AddColumn(#"Changed Type1", "Country", each if [#"Team/NOC"] = "Great Britain" then "United Kingdom" 
        else [#"Team/NOC"], type text)
in
    #"Added Custom"

# Power Query Medal Ranking
This Power Query (M) script used to transform For Medal Ranking.

```pq
let
    
    // This is a copy of Medals Data Table but using Columns Removed before in this new Table

    Source = Excel.Workbook(File.Contents("C:\Users\opr1141\OneDrive\My Power BI Projects\TOKYO OLYMPICS\Dataset\Tokyo Olympic Data.xlsx"), null, true),
    #"Tokyo Olympic_Sheet" = Source{[Item="Tokyo Olympic",Kind="Sheet"]}[Data],
    #"Removed Top Rows" = Table.Skip(#"Tokyo Olympic_Sheet",4),
    #"Trimmed Text" = Table.TransformColumns(Table.TransformColumnTypes(#"Removed Top Rows", {{"Column3", type text}, {"Column4", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type text}}, "en-US"),{{"Column1", Text.Trim, type text}, {"Column2", Text.Trim, type text}, {"Column3", Text.Trim, type text}, {"Column4", Text.Trim, type text}, {"Column5", Text.Trim, type text}, {"Column6", Text.Trim, type text}, {"Column7", Text.Trim, type text}}),
    #"Promoted Headers" = Table.PromoteHeaders(#"Trimmed Text", [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Rank", Int64.Type}, {"Team/NOC", type text}, {"Gold", Int64.Type}, {"Silver", Int64.Type}, {"Bronze", Int64.Type}, {"Total", Int64.Type}, {"Rank by Total", Int64.Type}}),
    
    // Keep Columns "Rank", "Team/NOC", "Total", "Rank by Total"

    #"Removed Other Columns" = Table.SelectColumns(#"Changed Type",{"Rank", "Team/NOC", "Total", "Rank by Total"}),

    // Renamed Rank to be Rank by Gold to remove missunderstanding,
    // By search, I know Rank depend primarily on Gold, if equal check Silver, if equal Check Bronze

    #"Renamed Columns" = Table.RenameColumns(#"Removed Other Columns",{{"Rank", "Rank by Gold"}}),

    // Unpivot the Two Ranking Ways to be used as Slicer in Dashboard
    
    #"Unpivoted Columns" = Table.UnpivotOtherColumns(#"Renamed Columns", {"Team/NOC", "Total"}, "Ranking", "Value")
in
    #"Unpivoted Columns"


# Power Query For Medal Type
This Power Query (M) script used to transform For For Medal Type.

```pq
let
    
    // Created This Manual Table just to Order Medals by Gold, Silver, then Bronze
    // Depending on Alphabitical Order will be a miss
    
    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("i45Wcs/PSVHSUTJUitWJVgrOzClLLQJyjcBcp6L8vKpUINdYKTYWAA==", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [#"Medal Type" = _t, Rank = _t]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Rank", Int64.Type}})
in
    #"Changed Type"

# Power Query For Medal Combination
This Power Query (M) script used to transform For For Medal Combination.

```pq
let
    
    // Created This Manual Table just to Order Medals Combination by Gold, Silver, then Bronze in Each Group
    // All-Medal: 
    // Two-Medals: Gold-Silver, Gold-Bronze,then Silver-Bronze
    // One Medal: Gold, Silver, then Bronze
    // Depending on Alphabitical Order will be a miss

    Source = Table.FromRows(Json.Document(Binary.Decompress(Binary.FromText("i45WcszJ0fVNTUnMKVbSUTJUitWJVnLPz0nRDc7MKUstAooZIcScivLzqlKBYsZgMYgShKgJQqV/Xk4lUMQUWR1UzAwsBtEEEzNXio0FAA==", BinaryEncoding.Base64), Compression.Deflate)), let _t = ((type nullable text) meta [Serialized.Text = true]) in type table [#"Medals Combination" = _t, Rank = _t]),
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Rank", Int64.Type}})
in
    #"Changed Type"
