# Purpose

This file serves to store practice queries for basic things like counting the number of rows in a given table.

# Contents

- [Where Practice](#where-practice)
- [Extend Practice](#extend-practice)
- [Math Operations Practice](#math-operations-practice)
# Practice Queries

## Where Practice

```
SigninLogs 
| where TimeGenerated > ago(2d)
| order by TimeGenerated asc
```

## Extend Practice

```
let role = "150f5e0c-0603-4f03-8c7f-cf70034c4e90"; // Data Purger
AzureActivity
| where Authorization has "Microsoft.Authorization/roleAssignments/write"
| extend RoleDefinitionId_ = tostring(parse_json(tostring(parse_json(tostring(parse_json(Properties).requestbody)).Properties)).RoleDefinitionId)
| extend scope_ = tostring(parse_json(Authorization).scope)
| where RoleDefinitionId_ has role
```

## Math Operations Practice

```
let days = 1d;
let days_l = tolong(days) / 864000000000;
print DaysLong=days_l, Days=days
```
