# Duplicating data

## operation name
duplicate

## Purpose
The purpose of the duplicate operation is to duplicate a dataframe.<br>
This will output the same dataframe on port 0 and port 1.<br>
This is quite useless and can be circumvented entirely by simply having two connections have their 'from' be the same operation and their 'from_port' be the same port. <br>
Choose your own way of doing it.

## Parameters
Duplicate operation has no parameters.

## Inputs
| Port | Description | type |
|-|-|-|
| 0 | Dataframe to duplicate | dataframe |


## Outputs
| Port | Description | type |
|-|-|-|
| 0 | The inputted dataframe | dataframe |
| 1 | The inputted dataframe | dataframe |