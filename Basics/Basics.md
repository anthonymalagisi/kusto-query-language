# Basics of KQL

## Table Of Contents
- [Introduction](#introduction)
- [Data Types and Variables](#data-types-and-variables)
    - [Data Types](#data-types)
    - [Variables](#variables)
    - [Operators](#operators)
- [Working with Data Sets](#working-with-data-sets)
    - [Getting Data](#getting-data)
    - [Limiting Data](#limiting-data)
    - [Sorting Data](#sorting-data)
    - [Filtering Data](#filtering-data)
    - [Summarizing Data](#summarizing-data)
    - [Adding Columns](#adding-columns)
    - [Combining Multiple Data Sets](#combining-multiple-data-sets)
- [Useful Commands](#useful-commands)
## Introduction

KQL is like a hybrid of SQL and PowerShell. Like SQL, it is used to make queries on datasets. KQL is structured in a similar way to SQL where you specify what data you are querying, and then perform actions on that data. The structure of KQL, however, adopts the idea of passing data down a pipeline, like in PowerShell. You will notice that it uses pipes like PowerShell as a result. Here is a sample of some KQL:

```
SigninLogs
| evaluate bag_unpack(LocationDetails)
| where RiskLevelDuringSignIn == 'none'
    and TimeGenerated >= ago(7d)
| summarize Count = count() by city
| order by Count desc
| take 5
```

A good rule of thumb is to filter data early on to improve performance. 

Also note that if your script has multiple lines, you must use a semicolon (;) at the end of each individual line. In this case, a sequence of commands used in a query (as in the one above) counts as one line where the semicolon goes to the last line. The last line does not need a semicolon. Here is a poorly written query that illustrates this point:

```
let testTime = 60m;
let recentData = SecurityEvent 
| where TimeGenerated  >= ago(testTime)
| order by EventID asc
| take 10;
let testData = recentData;
testData
```

Comment your code with "//" like in c based languages. Note that you cannot use block comments with /**/.

DO NOT SEPARATE LINES WITH EMPTY LINES! The query won't work properly.

## Data Types and Variables

### Data Types

Here are the data types in KQL:

Type        | Aliases           | Equivalent .NET type
---         | ---               | ---
bool        | Boolean           | System.Boolean
datetime    | Date              | System.DateTime
dynamic     | N/A               | System.Object
guid        | uuid, uniqueid    | System.Guid
int         | N/A               | System.Int32
long        | N/A               | System.Int64
real        | Double            | System.Double
string      | N/A               | System.String
timespan    | Time              | System.TimeSpan
decimal     | N/A               | System.Data.SqlTypes.SqlDecimal

For timespan, we have the following suffixes:

Function    | Description   
---         | ---
D           | days
H           | hours
M           | minutes
S           | seconds
Ms          | milliseconds
Microsecond | microseconds
Tick        | nanoseconds

Guid is a data type that represents a 128-bit, globally-unique identifier. It follows the format of 8-4-4-4-12 where each number is a hexadecimal number.

### Variables

Declare variables using let keyword. Example:

```
// Binds numberOfDays to the timespan value of 10 days
let numberOfDays = 10d;
```

Variables can store basic scalar values as well as whole data sets. Here is an example of both being used:

```
let takeNumber = 20;
let timeRange = 8h;
let dataSet1 = SecurityEvent
| where TimeGenerated >= ago(timeRange)
| order by TimeGenerated desc
| take takeNumber;
let dataSet2 = SigninLogs
| where TimeGenerated  >= ago(timeRange)
| order by TimeGenerated desc
| take takeNumber;
dataSet1
| union dataSet2
```

### Operators

As with any other programming language, there are operators that can be used to perform calculations. Here are some numerical operators:

Operator    | Description
:---:       | :---
\+          | Addition
\-          | Subtraction
\*          | Multiplication
/           | Division
%           | Modulo

Here are some general logical operators that can be used with any datatype:

Operator        | Description
:---:           | :---
==              | Checks if the item on the LHS is equal to the RHS
!=              | Checks if the item on the LHS is not equal to the LHS
in              | Checks if the element on the LHS is included in the object on the RHS
!in             | Checks if the element on the LHS is not included in the object on the RHS

Here are some logical operators that can be used with numbers:

Operator    | Description
:---:       | :---
\>=         | Greater than or equal to
\>          | Greater than      
<=          | Less than or equal to 
<           | Greater than

Here are some operators that can be used specifically with strings (note RHS means right hand side and LHS means left hand side):

Operator        | Description
:---:           | :---
has             | Checks if LHS contains the RHS
!has            | Checks if a string doesn't contain a substring
=~              | Checks if two strings are equal (case insensitive)
!~              | Checks if two strings are not equal (case insensitive)
has_all         | Same as has, but for all element on the RHS (and)
has_any         | Same as has, but for any element on the RHS (or)
contains        | RHS occurs as a substring of LHS (case insensitive)
!contains       | RHS does not occur as a substring of LHS (case insensitive)
in~             | Same as in, but case insensitive
!in~            | Same as !in, but case insensitive

[Return to Top](#table-of-contents)

## Working with Data Sets

### Getting Data 

To get a specific data set, all you have to do is write its name. The name is bound to the data in the table. Take note that log names are case sensitive. Some data sets used in the above examples include SecurityEvent (data set containing data from Windows Security logs), and SigninLogs (data set containing information on sign ins).

### Limiting Data

To limit the number of results in a given query, you can use the "take" command. The following example limits the response to 100 data points.

```
SecurityEvent
| where TimeGenerated >= ago(30d)
    and EventID == 4688
| order by TimeGenerated desc
| take 100
```

You can also limit what columns are shown using the "project-keep" command. This is in case there are only certain columns you are concerned with. For example, the following script does the same thing as the previous, but will only display TimeGenerated, Computer, EventID, Process, ParentProcessName, and CommandLine.

```
SecurityEvent
| project-keep TimeGenerated, Computer, EventID, Process, ParentProcessName, CommandLine
| where TimeGenerated >= ago(30d)
    and EventID == 4688
| order by TimeGenerated desc
| take 100
```

### Sorting Data

You will notice that in the above example, the "order" command is used. This command sorts the data by the data field chosen specified in either ascending (asc) or descending (desc) order which is also specified. In the case that you are sorting by multiple columns (the values of a column may not be unique), you can separate each column name with a comma. The first column specified will be sorted first, then if multiple data points have the same value in the first column, they will be sorted by the next, and so on and so forth. As an example:

```
let takeNumber = 200;
let timeRange = 1d;
SecurityEvent
| project-keep TimeGenerated, Computer, EventID, Process, ParentProcessName, CommandLine
| where TimeGenerated >= ago(timeRange)
    and EventID == 4688
| order by ParentProcessName, Process
| take takeNumber 
```

### Filtering Data

You will also notice in the above examples that the "where" command is used. This command keeps data points that meet the conditional statement. Column names can be used to specify the value wanted for the statement. Examples of conditional statements are:

```
TimeGenerated >= ago(10d) and EventID == 4688
coreCount_d == 4 
```

Here is an example of the where command being used:

```
SecurityEvent
| where TimeGenerated >= ago(30d)
    and EventID == 4688
| take 100
```

Check the logical operators in the section above for a list of operators you can use for conditional statements.

### Summarizing Data

Some data can be summarized in different ways. For example, the Perf log has a property called InstanceName. If you wanted to see how many times each name in InstanceName appears in the log, you would run the following (note that I am limiting the number of entries to 1000):

```
Perf
| take 1000
| summarize  count() by InstanceName
```

The output is a new table with the columns InstanceName and count_. Each row in the table is for a unique InstanceName and count is how many times that unique InstanceName shows up in the original dataset. Note that this new table is created as soon as the command summarize is used. The following will throw an error:

```
Perf
| take 1000
| summarize  count() by InstanceName
| order by TimeGenerated desc
```

This is because the table generated by the summarize command doesn't have a column named TimeGenerated. If you want more columns to be included, you must add them after "count() by" and separate them with commas. If you do this, however, the count will now represent how many items in the data set have the values given in each column. I will refer to the set of these columns as the group. Run the following and notice the differences in the tables:

```
Perf
| take 1000
| summarize  count() by InstanceName, CounterName
```

Each group consists of a unique InstanceName AND CounterName

```
Perf
| take 1000
| summarize count() by InstanceName, CounterName, CounterPath
```

Each group produced above consists of a unique InstanceName, CounterName, and CounterPath.

There are other types of summaries that we can make other then count. Here is a list of some of those summaries:

Function            | Description
:---:               | :---
count()             | Returns the count of data points that have data values specified by the group
max(exp)            | Returns the maximum value across the group given by "exp"
min(exp)            | Returns the minimum value across the group given by "exp"
avg(exp)            | Returns the average value across the group given by "exp"
countif(predicate)  | Returns count with the predicate of the group
dcount(exp)         | Returns the number of unique points in the group given by "exp"
stdev(exp)          | Returns the standard deviation of the group given by "exp"
sum(exp)            | Returns the sum of the items in the group given by "exp"

[Return to Top](#table-of-contents)

### Adding Columns

The command "extend" is used to create a new column that can be calculated from the values of other columns for each row. The format of this command is as follows:

```
| extend NewColumnName = "Expression involving other column values"
```

For example:

```
Usage
| where TimeGenerated >= ago(12h)
| order by TimeGenerated asc
| extend TimeDifference = EndTime - StartTime
| project-keep TimeGenerated, ResourceUri, StartTime, EndTime, TimeDifference
```

Note that a new column "TimeDifference" was generated and the values in this column were calculated by taking the difference between EndTime and StartTime for each row. 

### Combining Multiple Data Sets

We can combine the data from multiple data sets using either the union or join command.  The union command simply joins multiple tables and returns all of the rows. Here is an example of the union command:

```
Usage
| union Perf
```

union has a few parameters that can be used to adjust the result of the command. One of those is withsource, which creates a new column with the name of the table to which the row belongs to. You specify what the column is named.

```
Usage
| union withsource = SourceTable Perf
```

Another parameter is kind, which allows you to specify whether to perform an inner union or outer union. An inner union is one where only the columns that are shared by both tables will be kept (kind of like an and, or set intersection). An outer union is one where all the columns from both tables are kept (kind of like an or, or set union / is the default option for union). You specify which type by either putting in kind = inner or kind = outer. You can use the different parameters on the same line

```
Usage 
| union withsource = SourceTable kind = inner Perf
```

The other combining command is join. This command combines rows together and outputs a new table. Thus if two tables have data where each row represents the same object, join can be used to make a table with all of the data from both. There are many ways to do this, and therefore join has a rather verbose syntax:

```
TableOne
| join kind = <join type>
(
    TableTwo
) on $left.<TableOneColumn> == $right.<TableTwoColumn>
```

Note that the left table is TableOne and the right table is TableTwo. Just like union, we can specify the join kind. A list will be provided below. In the parentheses, we specify which table is to be combined to the left table. You can include multiple lines here if need be. After the parentheses, we specify which column from the left table and which column from the right table are going to be used to specify a match in the rows from each table. For example, let us say that TableOne has a column ComputerId that indicates the ID of the computer and TableTwo has a column computer_id that indicates the same thing. Then, if the ComputerId from a row in TableOne matches that of the computer_id in TableTwo, we know that those rows represent the same object. Sometimes two different tables will have the same column name for the identifier we want to use. Still use the above syntax for that scenario. Here is an actual example of the join command from the book:

```
OfficeActivity
| where TimeGenerated >= ago(1d)
    and LogonUserSid != ''
| join kind = inner (
    SecurityEvent
    | where TimeGenerated >= ago(1d)
        and SubjectUserSid != ''
) on $left.LogonUserSid == $right.SubjectUserSid
```

Note here that both tables are filtered by TimeGenerated. Make sure to do this to make sure that you are drawing data from the same time range for both tables. For this example, LogonUserSid and SubjectUserSid represent the same thing for the objects of interest and are thus used after the parentheses to tell join what to join by. It is a best practice to have the smallest table on the left.

Here are the join types:

Join Type       | Description
:---:           | :---
inner           | One row returned for each combination of matching rows
innerunique     | Inner join with left side deduplication (default)
leftouter       | Returns matched records from left table and all records from right/ unmatched will be null
rightouter      | Same as leftouter but left and right switched
fullouter       | Returns all records from both left and right tables / unmatched values will be null
leftanti        | Returns records that did not have a match in the right table. Only columns from left will be returned
rightanti       | Same as leftanti but left and right are switched
leftsemi        | Returns records that had a match in the right table. Only columns from left will be returned
rightsemi       | Same as leftsemi but left and right are switched

## Useful Commands

Command         | Description
---             | ---
getschema       | Displays column names, numbers, and types, and data types
let             | Binds names to expressions (variables)
take            | Limits the number of results in a query
order           | Sorts the data by the specified column(s) in either ascending or descending order
where           | Returns data that causes the conditional statement to be true
summarize       | Returns summary statistics of table as defined by aggregate function
project-keep    | Selects what columns you want to keep from the table
project-away    | Selects what columns you don't want to keep from the table
extend          | Creates a new calculated column (e.g. unit conversions)
union           | Combines two or more tables with ALL of the content from both
join            | Combines rows together by the specified join type
evaluate        | Allows for plugins to be invoked by the query (like R or Python plugins)

[Return to Top](#table-of-contents)
