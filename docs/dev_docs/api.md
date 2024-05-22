# api.py

api.py is the 'head' of the api. This is where all requests come in and are translated from JSON to its internal logic.

The api uses [Flask](https://flask.palletsprojects.com/en/3.0.x/).

## The code
From top to bottom


### APIResult

The APIResult dataclass is used as a return value.
Functions may return an APIResult where where different return types are possible. 

```
@dataclass
class APIResult:
    payload: tuple[int, Operation] | connection | pipelineObject | str = ""
    succes: bool = True
    HTTPStatusCode:int = 200
```
**payload**: The payload is the actual returned value/object. <br>
**success**: Success will be set to False if an exception has occured. In this case the Payload will be a string containing an error message. <br>
**HTTPStatuscode**: Returns what [status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) the api should respond with. This is 200 by default (OK) and will be changed when an exception occurs.


### Startup code
This code defines the api and sets it up ready to be started.

```
app = Flask(__name__)
CORS(app)
swagger = Swagger(app)
use_cache = False
```

'CORS(app)' sets up [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)

'swagger = Swagger(app)' sets up [swagger](https://swagger.io)

'use_cache = False' makes it so be default the cache will not be used.


### request: status

This code defines a request that simply returns "api online".
This is useful for checking if the api is running and able to receive requests.

```
@app.route('/status', methods=['GET'])
def get_status():
    return jsonify("api online")
```

### request: datasets

This code defines a request that returns the files that can be loaded.

```
@app.route('/datasets', methods=['GET'])
def get_datasets():
    return jsonify(loadOperation().loadable_files)
```

It uses a preset variable in the load_operation.py file. This means it will likely need to be rewritten when the workings of loading data is changed from preset datasets to loading from a database. (This is a planned change.)

### request: Execute-pipeline

This code defines a request to execute a pipeline.

```
@app.route('/execute-pipeline', methods=['POST']) # type: ignore
def execute_pipeline():
    json = request.get_json(silent=True)
    request_id = None
    session_id = None
    if json is not None:
        request_id = json.get("request_id")
        session_id = json.get("session_id")
    api_res = json_to_pipeline(json)
    if not api_res.succes:
        return jsonify_result(api_res.payload, session_id, request_id), api_res.HTTPStatusCode
    pipobj:pipelineObject = api_res.payload #type: ignore

    executed = controller.execute_pipeline(pipobj, use_cache, controller.cacheMethod.PIPELINE)
    if not executed.succes:
        return jsonify_result(str(f"There was a problem with a '{executed.operation_of_failure.__class__.__name__}' operation:  '") + str(executed.payload) + "'", session_id, request_id), 500
    else:
        json = pipelinememory_to_json(executed.payload) #type: ignore
        return jsonify_result(json, session_id, request_id), 200
```
##### - Getting the JSON and extracting basic info
```
json = request.get_json(silent=True)
request_id = None
session_id = None
if json is not None:
    request_id = json.get("request_id")
    session_id = json.get("session_id")
```
##### - calling [json_to_pipeline](#json_to_pipeline)

```
api_res = json_to_pipeline(json)
if not api_res.succes:
        return jsonify_result(api_res.payload, session_id, request_id), api_res.HTTPStatusCode
pipobj:pipelineObject = api_res.payload #type: ignore
```

If json_to_pipeline(json) was not succesful the exception is returned as response.
If it was succesful its return value is saved as pipobj. This will be a pipelineObject.

##### - executing pipeline and sending response.

```
executed = controller.execute_pipeline(pipobj, use_cache, controller.
cacheMethod.PIPELINE)

if not executed.succes:
    return jsonify_result(str(f"There was a problem with a '{executed.operation_of_failure.__class__.__name__}' operation:  '") + str(executed.payload) + "'", session_id, request_id), 500
else:
    json = pipelinememory_to_json(executed.payload) #type: ignore
    return jsonify_result(json, session_id, request_id), 200
```

### json_to_pipeline

Translates JSON to internal logic.


```
def json_to_pipeline(json) -> APIResult:
    if (json is None):
        return APIResult("No json input was received by api.", False, 400)
    if "operations" not in json:
        return APIResult("Request does not define operations.", False, 400)
    if "connections" not in json:
        return APIResult("Request does not define connections.", False, 400)
    if "session_id" not in json:
        return APIResult("Request does not define sessionid.", False, 400)
    json_operations = json["operations"]
    json_connections = json["connections"]

    pipobj = pipelineObject()
    pipobj.session_id = json["session_id"]
    pipobj.request_id = json.get("request_id")

    api_res = json_to_operations(pipobj, json_operations)
    if not api_res.succes:
        return api_res
          
    api_res = json_to_connections(pipobj, json_connections)
    if not api_res.succes:
        return api_res
    return APIResult(pipobj)
```
Some simple checks:
```
if (json is None):
    return APIResult("No json input was received by api.", False, 400)
if "operations" not in json:
    return APIResult("Request does not define operations.", False, 400)
if "connections" not in json:
    return APIResult("Request does not define connections.", False, 400)
if "session_id" not in json:
    return APIResult("Request does not define sessionid.", False, 400)
```
Defining variables:
```
json_operations = json["operations"]
json_connections = json["connections"]

pipobj = pipelineObject()
pipobj.session_id = json["session_id"]
pipobj.request_id = json.get("request_id")
```

[Translating operations](#json-to-operations):
```
api_res = json_to_operations(pipobj, json_operations)
    if not api_res.succes:
        return api_res
```

[Translating connections](#json-to-connections):
```
api_res = json_to_connections(pipobj, json_connections)
    if not api_res.succes:
        return api_res
    return APIResult(pipobj)
```

### json to operations

It converts a list of JSON operations into a list of operations and adds these to the pipelineObject.

This uses [operation_to_result](#operation-to-result)

```
def json_to_operations(pipobj:pipelineObject, json_operations:list) -> APIResult:
    results: list[APIResult] = []
    operation_hashes_by_id = {}
    operations_by_id = {}
    pipeline:list[Operation] = []
    for item in json_operations:
        api_res = operation_to_result(item)
        if (api_res.succes):
            results.append(api_res)
        else:
            return api_res

    for res in results:
        if type(res.payload) == tuple:
            payload = res.payload
            operations_by_id[payload[0]] = payload[1]
            operation_hashes_by_id[payload[0]] = payload[1].hash() #type: ignore
            pipeline.append(payload[1])
        else:
            return APIResult(f"unworkable payload detected while preparing pipeline. Payload should have been tuple(int, Operation). Payload given: {api_res.payload} ", False, 500)
    pipobj.operations = pipeline
    pipobj.operations_by_id = operations_by_id
    pipobj.operation_hashes_by_id = operation_hashes_by_id
    return APIResult(pipobj)
```

### json to connections

It converts a list of JSON connections into a list of connections and adds these to the pipelineObject.


This uses [connection_to_result](#connection-to-result)

```
def json_to_connections(pipobj:pipelineObject, json_connections:list) -> APIResult:
    results = []
    for con in json_connections:
        api_res = connection_to_result(con, pipobj.operations_by_id)
        if (api_res.succes):
            results.append(api_res)
        else:
            return api_res
    
    connections:list[connection] = []
    for res in results:
        if type(res.payload) == connection:
            connections.append(res.payload)
        else:
            return APIResult(f"unworkable connection detected while preparing pipeline. Payload should have been connection. Payload given: {api_res.payload}", False, 500)
    pipobj.connections = connections
    return APIResult(pipobj)
```


### operation to result

It takes an operation defined in JSON and translates it to an operation object of the correct class. It may also perform some checks.

```
def operation_to_result(json_operation: dict[str, str]) -> APIResult:
    operation:Operation
    id:int = 0
    #get operation
    match json_operation:
        case {"operation": "load"}:
            params = json_operation.get("parameters")
            check = check_parameters(params, "load")
            if check.succes == False:
                return check
            operation = loadOperation()
            insertion_result = operation.insert_parameters(params) #type: ignore
            if type(insertion_result) == Exception:
                return APIResult("A load operation had a problem while inserting parameters. Gave following Exception: " + str(insertion_result), False, 400)

        case {"operation": "duplicate"}:
            operation = duplicateOperation()

        case {"operation":"select_columns"}: 
            operation = selectColumnsOperation()
            params = json_operation.get("parameters")
            check = check_parameters(params, "select_columns")
            if check.succes == False:
                return check
            insertion_result = operation.insert_parameters(params) #type: ignore
            if type(insertion_result) == Exception:
                return APIResult("A select_columns operation had a problem while inserting parameters. Gave following Exception: " + str(insertion_result), False, 400)
            
        case {"operation":"merge"}: 
            params = json_operation.get("parameters")
            check = check_parameters(params, "merge")
            if check.succes == False:
                return check
            operation = mergeOperation()
            insertion_result = operation.insert_parameters(params) #type: ignore
            if type(insertion_result) == Exception:
                return APIResult("A merge operation had a problem while inserting parameters. Gave following Exception: " + str(insertion_result), False, 400)
        
        case {"operation": "decision_tree"}:
            params = json_operation.get("parameters")
            check = check_parameters(params, "decision tree")
            if check.succes == False:
                return check
            operation = decisionTree()
            insertion_result = operation.insert_parameters(params) #type: ignore
            if type(insertion_result) == Exception:
                return APIResult("A decision tree operation had a problem while inserting parameters. Gave following Exception: " + str(insertion_result), False, 400)
    
        case {"operation":"filter_rows"}: 
            params = json_operation.get("parameters")
            operation = filterRowsOperation()
            insertion_result = operation.insert_parameters(params) # type: ignore
            if type(insertion_result) == Exception:
                return APIResult("A filter rows operation had a problem while inserting parameters. Gave following Exception: " + str(insertion_result), False, 400)

        case {"operation":"test_operation"}: 
            operation = controller.testOperation()

        case {"operation":"predict"}:
            params = json_operation.get("parameters")
            operation = predict()
            insertion_result = operation.insert_parameters(params)
            if type(insertion_result) == Exception:
                return APIResult("A predict operation had a problem while inserting parameters. Gave following Exception: " + str(insertion_result), False, 400)


        case {"operation": notImpl}:
            return APIResult(f"Operation {notImpl} is not implemented", False, 400)    
                 
        case _:
            return APIResult(f"A requested operation is not implemented. Could not find implemented operation regarding the following: {json_operation}", False, 404)
        
    # test id
    match json_operation:
        case {"id": _id} if type(_id) == int:
            id = _id
        case {"id": _id}:
            return APIResult(f"The following id was found not to be convertible to an integer. id: {_id}.", False, 400)
        case _:
            return APIResult("An operation was found not to have an id.", False, 400)
    return APIResult(tuple([id, operation])) #type: ignore
```
### connection to result

It takes a connection defined in JSON and translates it to a connection object.

**Connectiondict** should be the connection in JSON.<br>
**operations** should be a dictionary of operations with their id as the keys.


```
def connection_to_result(connectiondict: dict[str, int], operations:dict[int, Operation]) -> APIResult:
    return APIResult(connection(operations[connectiondict["from"]], 
                                operations[connectiondict["to"]], 
                                {cport.FROM:int(connectiondict["from_port"]), cport.TO:int(connectiondict["to_port"])}
                                ))
```

### pipelinememory to JSON

This converts a pipelineObject into JSON.

This is used to convert internal logic back to JSON so it can be given as response.

```
def pipelinememory_to_json(pipobj:pipelineObject) -> dict[str, str | dict]:
    pipmem:pipelineMemory = pipobj.pipeline_memory
    json: dict[str, str | dict] = {}
    ids_by_operation:dict[Operation, int] = invert_dictionary(pipobj.operations_by_id)
    for key, value in pipmem.operations_memory.items():
        operation:Operation = key[0]
        port:int = key[1]
        id:int= ids_by_operation[operation]
        jsonpipe:dict[str, str] = {}

        match value:
            case val if isinstance(val, pd.DataFrame):
                jsonpipe["output"] = dataframe_to_json(val)
                jsonpipe["output_type"] = "data"

            case val if isinstance(val, int):
                jsonpipe["output"] = str(value)
                jsonpipe["output_type"] = "integer"

            case val if isinstance(val, list):
                jsonpipe["output"] = str(value)
                jsonpipe["output_type"] = "array"
            
        pipe_output = json.get("id_" + str(id))
        if type(pipe_output) == dict:
            pipe_output.update({"port_" + str(port): jsonpipe})
        else:
            pipe_output = {"port_" + str(port): jsonpipe}
        json["id_" + str(id)] = pipe_output
    return json
```

```
def operation_to_json(operation:Operation):
    return str(type(operation))

def invert_dictionary(dict:dict) -> dict:
    return {v: k for k, v in dict.items()}
```

### The start function

This function starts the api.

```
if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Run the Flask application.')
    parser.add_argument('--host', type=str, default="127.0.0.1", help='The host')
    parser.add_argument('--port', type=int, default=5000, help='Port number to run the server on')
    parser.add_argument('--cache', default=False, action='store_true', help='If true, uses redis cache for better performance')
    parser.add_argument('--debug', default=False, action='store_true', help='Start in debug mode')
    args = parser.parse_args()

    use_cache = args.cache
    app.run(host=args.host,port=args.port, debug=args.debug)
```

#### Startup flags

When starting the api extra arguments can be given.

For example:
> python api.py --port 5050 --cache

The following argument flags exist:

|Argument| Followed by | Example|Description|
|-|-|-|-|
|--host| IP address | 127.0.0.1 | Sets the host IP address  |
|--port| Port number| 5050 | Sets the host port|
|--cache| Nothing | / | Enables cahing |
|--debug | Nothing | / | Makes Flask run in Debug mode |

