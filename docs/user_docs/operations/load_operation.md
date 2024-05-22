# Loading data
## operation name
load

## Purpose
The purpose of the load operation is to load in data.

## Parameters
| Parameter | Description | Datatype | Required | Default |
|-|-|-|-||
|filename| The datafile to load| string| yes| /|
|sample| sample size| bool or int| no| False|
|randomize_sample| If true, randomizes  what data <br> is used for the sample | bool | no | False|


## Inputs
The load operation does not take any inputs.


## Outputs
| Port | Description | type |
|-|-|-|
| 0 | Loaded dataframe |dataframe |