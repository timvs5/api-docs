# Pipelines
## What is a pipeline? 
A Pipeline is a list of things you want to do ([operations](operations_info.md)) and the order you want them done ([connections](connections_info.md)). For example, you might want to retrieve data from a database, select a few columns of data and then train a linear regression model on it. In this case you would use 3 operations and define connections between these operations.

## Execute Pipeline

You can execute a pipeline by sending a '/Execute-pipeline' POST request to the API.

### What goes in?

In order to execute a pipeline you will need to defien the pipeline in JSON.
A pipeline is defined as follows:

```
{
"session_id":session_id,
"request_id":request_id,
"connections":[
    connection_1, connection_2, ...
],
"operations":[
    operation_1, operation_2, ...
]}
```
Example:
```
{
"session_id":1,
"request_id":1,
"connections":[
    {"from":1, "to":2, "from_port":0, "to_port":0}
],
"operations":[
    {"id":1,"operation":"load","parameters":{"filename":"titanic", "sample":1}},
    {"id":2,"operation":"select_columns", "parameters":{"columns":["age", "class"]}}
]}
```



### What comes out?

The response will return a JSON dictionary With the session_id, the request_id and the results of the executed pipeline.<br>
The result will be a dictionary with the id's of the operations.<br>
The id's will have a dictionary with the ports they outputted to. <br> 
The ports will have a dictionary with output and output type.

Example:
```
{
    "request_id": 1,
    "result": {
        "id_1": {
            "port_0": {
                "output": "{\"columns\": [\"index\", \"class\", \"age\", \"sex\", \"survived\"], \"datatypes\": {\"class\": \"object\", \"age\": \"object\", \"sex\": \"object\", \"survived\": \"object\"}, \"data\": [[0, \"1st class\", \"adults\", \"man\", \"yes\", 1]]}",
                "output_type": "data"
            }
        },
        "id_2": {
            "port_0": {
                "output": "{\"columns\": [\"index\", \"age\", \"class\"], \"datatypes\": {\"age\": \"object\", \"class\": \"object\"}, \"data\": [[0, \"adults\", \"1st class\", 1]]}",
                "output_type": "data"
            },
            "port_1": {
                "output": "{\"columns\": [\"index\", \"sex\", \"survived\"], \"datatypes\": {\"sex\": \"object\", \"survived\": \"object\"}, \"data\": [[0, \"man\", \"yes\", 1]]}",
                "output_type": "data"
            }
        }
    },
    "session_id": 1
}
```

To get the output of operation with id 3 from port 0 you would call: json["result"]["id_3"]["port_0"]["output"]


### Errors