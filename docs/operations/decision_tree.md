# Training a decision tree

## operation name
decision_tree

## Purpose
The purpose of the decision_tree operation is to train a [decision tree](https://en.wikipedia.org/wiki/Decision_tree_learning)

Example of a decision tree:

![Image of decision tree](https://upload.wikimedia.org/wikipedia/commons/thumb/e/eb/Decision_Tree.jpg/330px-Decision_Tree.jpg)

## Parameters
| Parameter | Description | Datatype | Required | Default |
|-|-|-|-|-|
| target | The target value the decision tree <br> predicts on | string | yes | / |


## Inputs
| Port | Description | type |
|-|-|-|
| 0 | Dataframe with data to train on | dataframe |

It will attempt to train on all columns in the inputted dataframe.


## Outputs
| Port | Description | type |
|-|-|-|
| 0 | Trained decision tree model | model |

Note that a model can not be returned in JSON.