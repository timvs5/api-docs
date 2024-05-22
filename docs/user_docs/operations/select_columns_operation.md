# Selecting columns
## Operation name
select_columns

## Purpose
The purpose of the select_columns operation is to select columns from a dataframe and continue with a new dataframe with only these columns.

## Parameters
| Parameter | Description | Datatype | Required | Default |
|-|-|-|-|-|
|columns| The columns to be selected | list of strings | yes | /


## Inputs
| Port | Description | type |
|-|-|-|
| 0 | Dataframe to select columns from | dataframe |

## Outputs
| Port | Description | type |
|-|-|-|
| 0 | Inputted dataframe with only the columns to select on |dataframe |
| 1 | Inputted dataframe with columns to select on filtered out|dataframe |

Port 1 effectively removes the columns.