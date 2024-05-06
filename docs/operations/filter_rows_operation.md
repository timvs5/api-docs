# Filtering rows
## Operation name
filter_rows

## Purpose
Filtering data based on a set of conditions.
For example: removing every row where the column "income" has a value above 1800.

## Parameters
| Parameter | Description | Datatype | Required | Default |
|-|-|-|-|-|
|conditions| A list of conditions | list | yes | /

### Condition
A condition is a dictionary with following parameters.

| Parameter | Description | Datatype | Required | Default |
|-|-|-|-|-|
|column| The column you want to filter the data on | string | yes | /
|comparator| The comparison you want to make | string | yes | /
|compare_with| the column or value you want to compare with | string, int, float | yes | /
|compare_with_type| Defines what compare_with is (column or value) | string | yes | /

### comparator
Possible values:

- '=='
- '!='
- '<'
- '>'
- '<='
- '>='

For example, "comparator":">"  will result in a dataset with only the rows where value in 'column' > 'compare_with'.


### compare_with_type
Can be either 'column' or 'value'



## Inputs
| Port | Description | type |
|-|-|-|
| 0 | Dataframe to filter on | dataframe |

## Outputs
| Port | Description | type |
|-|-|-|
| 0 | filtered dataframe |dataframe |