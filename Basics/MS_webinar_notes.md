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
- arg_max

#### Examples

```
SecurityEvent
| where TimeGenerated > ago(2d)
| summarize Count=count() by EventID
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
