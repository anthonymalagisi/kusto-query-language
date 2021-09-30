# Description

# Part 1: Learn the KQL You Need for Azure Sentinel

## Introduction

- KQL used when you build rules and workbooks
- KQL is pipe oriented 
    - Data | Condition to Filter | Evidence (prepare)
- Multiple ways to get a table of Data
    - Use the Standard Tables (SecurityEvent, SecurityAlert, AzureActivity, etc.)
    - Use Custom Tables ([blog post on how](https://techcommunity.microsoft.com/t5/azure-sentinel/azure-sentinel-creating-custom-connectors/ba-p/864060))
    - Use union to query multiple tables
    - Use externaldata to query a table in an external file
    - Use datatable to query a static table 
    - Use stored functions to pre-prepare and parse a virtual table
- Any query is a table

## where

- This filters a table to the subset of rows that satisfies a condition
- Can use "and", "or", and "not()"

### Examples

```
SecurityEvent
| TimeGenerated > ago(1d)
```

```
SecurityEvent
| where EventID in (4624,4625)
```

(Note: "()" is the list operator. "Column in ()" will check if an item from the list shows up in the Column)

## search

- Inefficient and not recommended for use in content. Good for interactive 

### Examples

```
SecurityEvent
| where TimeGenerated >= ago(1h)
| search "Guest"
```






