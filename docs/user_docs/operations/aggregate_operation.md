# Aggregating data
## Operation name
aggregate

## Purpose
[Aggregating data as in SQL](https://www.w3schools.com/sql/sql_aggregate_functions.asp)


## Parameters
| Parameter | Description | Datatype | Required | Default |
|-|-|-|-|-|
|group_by| Column to group by| string | yes | /
|agg_column| Column to aggregate on| string | yes | /
|agg_method| Method by which to aggregate| string | yes | /


### agg_method
The API can handle following aggregation methods

| agg_method | Description |
|-|-|
| count | counts rows |
| avg | average |
| min | minimum |
| max | maximum |
| sum | Total sum of everything |


## Inputs
| Port | Description | type |
|-|-|-|
| 0 | Dataframe to aggregate on | dataframe |

## Outputs
| Port | Description | type |
|-|-|-|
| 0 | Dataframe |dataframe |