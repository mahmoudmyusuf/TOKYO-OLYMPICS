# Power Query For Medals Data
Power Query (M) script used to transform data.

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



