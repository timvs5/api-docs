# Merging data

## operation name
merge

## Purpose
The purpose of the merge operation is to merge two dataframes.<br>
This will perform a join operation as in SQL.<br>

## Parameters
| Parameter | Description | Datatype | Required | Default |
|-|-|-|-||
|column_1| Column from inputted <br> 'left' dataframe to join with | string | yes| /|
|column_2| Column from inputted <br> 'right' dataframe to join with | string | yes| /|
|join_type| Type of join to perform | string | no | inner |

### Join types
join_type can be one of four values.

* inner
* outer
* left
* right

These work the same as [SQL joins](https://www.w3schools.com/sql/sql_join.asp)


## Inputs
| Port | Description | type |
|-|-|-|
| 0 | 'Left' dataframe to join on | dataframe |
| 1 | 'Right' dataframe to join on | dataframe |


## Outputs
| Port | Description | type |
|-|-|-|
| 0 | Merged dataframe | dataframe |