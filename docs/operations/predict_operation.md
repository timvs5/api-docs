# Predicting
## operation name
predict

## Purpose
The purpose of the predict operation is to use an inputted decision tree to make a prediction.

## Parameters
| Parameter | Description | Datatype | Required | Default |
|-|-|-|-||
|values| list with lists of values to make predictions on | list of lists | yes | / |


## Inputs
| Port | Description | type |
|-|-|-|
| 0 | decision tree model | model |


## Outputs
| Port | Description | type |
|-|-|-|
| 0 | list of predicted outcomes | list|