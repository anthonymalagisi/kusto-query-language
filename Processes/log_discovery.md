# Description

The following process can be used to do an initial discovery of an unknown log. Should give you an idea of what is in the log, a sample of entry points, and basic features of the log. Even though each log is different and will require further investigation to use properly, this will serve as a starting point for any new log.

These sample queries are general in nature, and thus use some variables. Each variable will be surrounded by {} and must be replaced depending on your situation. Here are some variables that will be used and will need to be changed when actually put in use:

- {TableName} is the desired log
- {TimePeriod} is the desired time period
- {ColumnName} is the desired column

# Overview of steps

## Surface Level

- [Get Schema](#get-schema)
- [Rough Daily Count](#rough-daily-count)
- [Sample Full Entries](#sample-full-entries)

## Deeper Dive

- [Summarize Count by Column](#summarize-count-by-column)

# Surface Level

## Get Schema

The following query gets the name and data type of each column for a given log and sorts by the column name. I would advise that you export the resulting table as a csv file.

```
{TableName}
| getschema 
| project-keep ColumnName, DataType
| sort by ColumnName asc
```

## Rough Daily Count

The following gives the average daily count and total count of entries in a given log. You can choose how many days you wish to take the average and total over by changing the first line from `7d` to how ever many days you want. Make sure to use `d` after the number of days

```
let days = 7d; // Change as needed
let days_l = tolong(days) / 864000000000;
let totalCount = toscalar({TableName}
| where TimeGenerated >= ago(days)
| count
);
let averageDaily = totalCount / days_l;
print Average=averageDaily, Total=totalCount
```

## Sample Full Entries

Based on the daily count, you can choose an appropriate time limit to get a good sample of what the entries are from the table. Then take only 100 of the entries with all of their columns. Compare the actual results to the table schema to get a better idea of what each column will result in. Use this to further investigate based on your needs.

```
{TableName}
| where TimeGenerated >= ago({TimeValue})
| take 100
```

# Deeper Dive

## Summarize Count by Column

Before filtering a table, it is helpful to know what sort of values show up the most in that table. You can find this out using the `summarize count() by column_name` command. Find a column you are interested in and use the following query to get a summary of values that show up in that column (it will also give the percentage of total count for each column):

```
let totalCount=toscalar({TableName}
| where TimeGenerated >= ago({TimePeriod})
| count);
{TableName}
| where TimeGenerated >= ago({TimePeriod})
| summarize count() by {ColumnName}
| extend Percentage=round(todecimal(count_) / totalCount * 100,2)
```