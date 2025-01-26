# DAX Expressions for Tokyo Olympics Data Analysis
The Following are the DAX Expressions Used to facilitate Dashboard 

## Control MAX Value in Trends
This DAX expression is used to control the maximum value in the trends for the medals data.

```dax
All Combin Max = 
MAXX(
    SUMMARIZE(
        ALL('Medals Data'),
        'Medals Data'[Winners | Medals], 
        "TeamCount", DISTINCTCOUNT('Medals Data'[Team/NOC])
    ),
    [TeamCount]
)
```

## COUNTS Folder
This section contains DAX expressions to count various medal combinations and winners.

```dax
All Medal Winners = CONCATENATE(
  CALCULATE(DISTINCTCOUNT('Medals Data'[Team/NOC]),'Medals Data'[Medals Combination]="All-Medals"),"")

Two Medal Winners = CONCATENATE(
  CALCULATE(DISTINCTCOUNT('Medals Data'[Team/NOC]),'Medals Data'[Medals Combination] 
  IN {"Gold-Silver", "Silver-Bronze", "Gold-Bronze"}),"")

Single Medal Winners = CONCATENATE(
  CALCULATE(DISTINCTCOUNT('Medals Data'[Team/NOC]),'Medals Data'[Medals Combination] 
  IN {"Gold", "Silver", "Bronze"}),"")

Total Winners = DISTINCTCOUNT('Medals Data'[Team/NOC])
```

## MEDAL% Folder
This section contains DAX expressions to calculate medal percentages.
To show only when a filter is applied on part of Medals. otherwise BLANK.

```dax
Medal % = if([Medals Selected]=[Medal Type Total],"",[Medals Selected]/[Medal Type Total])

Medal total % = [Medals Selected]/[All Medal Total]

Medal total % text = FORMAT([Medals Selected]/[All Medal Total],"0.0%")&"" 
```

## SUM Folder
This section contains DAX expressions for summing medal data.

```dax
All Medal Total = SUMX(ALL('Medals Data'),'Medals Data'[Total])

Medal Type Total = 
CALCULATE(
    SUM('Medals Data'[Total]),
    REMOVEFILTERS('Medals Data'),
    'Medals Data'[Medal Type] = SELECTEDVALUE('Medals Data'[Medal Type])
)

Medals Selected = SUM('Medals Data'[Total])
```

## TITLES Folder
This section contains DAX expressions to generate titles for regions, countries, and tables.

```dax
title-Region-Country = if(COUNT('Country-region'[Team/NOC])>1,
  CONCATENATE(SELECTEDVALUE('Country-region'[region])," Medals"),
  CONCATENATE(SELECTEDVALUE('Country-region'[region]) & " - " & 
  SELECTEDVALUE('Country-region'[Team/NOC])," Medals"))

title-Table-Country = 
if(CONCATENATE([Medal Type Total],SELECTEDVALUE('Medals Data'[Winners | Medals]))="", "All Medal Combinations", 
CONCATENATE(
    SELECTEDVALUE('Medals Data'[Winners | Medals]),
    if(DISTINCTCOUNT('Medals Data'[Medals Combination])>1,""," - " & 
    SELECTEDVALUE('Medals Data'[Medals Combination]))))
    & ", "&SELECTEDVALUE('Medals Ranking'[Ranking])
```

---

This `.md` file includes all DAX expressions for medal analysis and related transformations, organized by their respective folders.
