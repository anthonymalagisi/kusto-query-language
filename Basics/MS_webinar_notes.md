# Description

# Part 1: Learn the KQL You Need for Azure Sentinel

## Introduction

- KQL used when you build rules and workbooks
- KQL is pipe oriented 
    - Data | Condition to Filter | Evidence (prepare)

## Filter & Prepare

Choose input -> Filter on Condition <-> Parse & Prepare

###  Choose Input

- Multiple ways to get Data
    - Use the Standard Tables (SecurityEvent, SecurityAlert, AzureActivity, etc.)
    - Use Custom Tables ([blog post on how](https://techcommunity.microsoft.com/t5/azure-sentinel/azure-sentinel-creating-custom-connectors/ba-p/864060))
    - Use union to query multiple tables
    - Use externaldata to query a table in an external file
    - Use datatable to query a static table 
    - Use stored functions to pre-prepare and parse a virtual table
- Any query is a table

### Filter

### where 

- This filters a table to the subset of rows that satisfies a condition
- Can use "and", "or", and "not()"

#### Examples

```
SecurityEvent
| TimeGenerated > ago(1d)
```

```
SecurityEvent
| where EventID in (4624,4625)
```

(Note: "()" is the list operator. "Column in ()" will check if an item from the list shows up in the Column)

### search

- Inefficient and not recommended for use in content. Good for interactive 

#### Examples

```
SecurityEvent
| where TimeGenerated >= ago(1h)
| search "Guest"
```

### Parse and Prepare

### extend

- Used to create a calculated column and append to result set
- Used extensively for parsing
- Utilize built in functions for calculations like extract, parse, split, etc.

#### Examples

```
SecurityEvent
| where EventID in (4624,4625)
| extend rgroup1=extract("resourcegroups/(.*)/providers",1,_ResourceId)
| extend rgroup2=split(_ResourceId, "/",4)[0]
| parse _ResourceId with "/subscriptions/" sub "/resourcegroups/" rgroup3 "/providers" *
| project-keep _ResourceId, rgroup1, rgroup2, rgroup3, sub
```

## Analyze

### summarize

- Takes input stream and aggregates content
- Syntax: `T|summarize Aggregation [by Group Expression]`
- Simple Aggregation functions: count(), sum(), avg(), min(), max()
- Advanced functions: arg_min(), arg_max(), make_list(), countif()
- Can override the default column name produced by using typing the desired resulting column name followed by an equals sign and the aggregate function. For example, `summarize Count=count() by EventID`
- arg_max() and arg_min() is a filtering option and will return the top or bottom records of the field specified. 

#### Examples

```
SecurityEvent
| where TimeGenerated > ago(2d)
| summarize Count=count() by EventID
```

```
SecurityEvent
| where TimeGenerated >= ago(60d)
| summarize arg_max(TimeGenerated,AccountType,Computer,EventID,Activity) by Account
| limit 1000
```

```
SecurityEvent
| summarize arg_max(TimeGenerated,*) by Account
```

### order by

- sorts the table by a desired column ascending (asc) and descending (desc)
- case operator is similar in logic to a switch statement where the value returned is based on a list of defined conditions and the result if the specific condition is met. Here is some general syntax. Each condition must be a boolean statement:

```case(
    Condition_1, Result_1,
    Condition_2, Result_2,
    ...
    Condition_n, Result_n,
    Default_Value)
```

- The case operator can be used if you wish to sort a table by string values

#### Examples

```
AzureActivity 
| where TimeGenerated >= ago(1d) 
| extend Status=case(
    ActivityStatusValue == "Success" or ActivityStatusValue == "Succeeded",2,
    ActivityStatusValue == "Start" or ActivityStatusValue == "Started",1,
    ActivityStatusValue == "Failed",0,
    -1)
| order by Status
| project-away Status
| take 3000
```

```
SecurityEvent
| where TimeGenerated >= ago(14d)
| summarize Count=count() by EventID
| order by Count desc
```

The following example is used for password spray detection:

```
let timeframe = 1d;
let threshold = 3;
SigninLogs
| where TimeGenerated >= ago(timeframe)
| where ResultType == "50057"
| where ResultDescription =~ "User account is disabled. The account has been disabled by an administrator."
| summarize applicationCount = dcount(AppDisplayName) by UserPrincipalName, IPAddress
| where applicationCount >= threshold
```

## Prepare

### project

- Will select a number of fields to keep the results in. Can also remove specific fields (`project-remove`) and rename specific fields (`project-rename`)
- `iff()` function similar to if in excel. Syntax: `iff(Condition,ValueIfTrue,ValueIfFalse)`

#### Examples

```
SecurityEvent
| where TimeGenerated >= ago(2d)
| where EventID == 4688
| project-keep TimeGenerated, Account, Computer, NewProcessName, ParentProcessName, CommandLine
| order by TimeGenerated desc
| limit 1000
```

```
SecurityEvent
| where TimeGenerated >= ago(2d)
| where EventID == 4688
| project-keep TimeGenerated, Account, Computer, NewProcessName, ParentProcessName, CommandLine
| project-rename ProcessPath=NewProcessName, ParentProcessPath=ParentProcessName
| order by TimeGenerated desc
| limit 1000
```

```
SecurityEvent
| where TimeGenerated >= ago(2d)
| where EventID == 4688
| extend OfInterest = iff(ParentProcessName  has "LTSVC.exe","Yes","No")
| project Account, Computer, OfInterest, NewProcessName, CommandLine 
| limit 1000
```

### summarize with make_list() and make_set()

- make_list() and make_set() are used to keep values of a summary group as a list.
- This allows you to retain data lost normally through summarize
- make_list() will keep all values
- make_set() will keep a subset

#### Examples

This returns computers that have had a security event in the last ten minutes with a list of the associated accounts on that computer (who have had a security event)

```
SecurityEvent
| where TimeGenerated >= ago(10m)
| summarize make_set(Account) by Computer
```

## Visualize

### summarize, bin, and timechart

- bin is used to create bins which are ranges of numbers used to split a data set into groups, often used to create histograms
- Useful with TimeGenerated
- Syntax: `bin(Column,binSize)`

#### Examples

```
SecurityEvent
| where TimeGenerated >= ago(5d) and EventID == 4688
| summarize count() by bin(TimeGenerated,1h)
| render
```

### render

- Generates a visualization
- Syntax: `Table | render Visualization [with (PropertyName=PropertyValue [,...])]`
- Types of visualizations:
    - areachart     
    - barchart      
    - columnchart   
    - piechart      
    - scatterchart  
    - timechart     : Displays how a value changes over time

#### Examples

```
SecurityEvent
| where TimeGenerated >= ago(5d)
| summarize ProcessCount=countif(EventID == 4688), OtherCount=countif(EventID != 4688) by bin(TimeGenerated, 1h)
| render timechart 
```

