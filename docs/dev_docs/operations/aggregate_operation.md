# Aggregate Operation

The aggregate_operation is used for aggregating data [as in SQL](https://www.w3schools.com/sql/sql_aggregate_functions.asp)

[Read the user documentation about the aggregate_operation](../../user_docs/operations/aggregate_operation.md)

```
class aggregateOperation(oi.operation_interface):
```
## init
`group_by_column` defines what column to group by.

`aggregate_column` defines what column to aggregate on

`agg_method` defines what [method](#aggregate_method) to aggregate with.

```
def __init__(self):
    self.group_by_column: str
    self.aggregate_column: str
    self.agg_method: aggregate_method
```

## insert_parameters
```
def insert_parameters(self, params:dict) -> Exception | None:
    if params is None:
        return Exception("parameters missing")
    gcol = params.get("group_by")  
    if gcol is None:
        return Exception("parameter 'group_by' missing")
    self.group_by_column = gcol
    agcol = params.get("agg_column")  
    if agcol is None:
        return Exception("parameter 'agg_column' missing")
    self.aggregate_column = agcol
    agmeth = self.to_aggregate_method(params.get("agg_method"))
    if isinstance(agmeth, Exception):
        return agmeth
    self.agg_method = agmeth
```

## call
```
def __call__(self, inputs):
    for con in inputs:
        if isinstance(con.payload, pd.DataFrame):
            df = con.payload.copy()
            aggregated_data = 
                df
                .groupby(self.group_by_column)
                .aggregate({self.aggregate_column:self.agg_method.value})
            return [{"port":0, "output":aggregated_data}]
```

## to_aggregate_method

This method converts a string to an aggregate_method, if possible.

```
def to_aggregate_method(self, param):
    for method in aggregate_method:
        if method.value == param:
            return method
    return Exception("Invalid or missing agg_method")
```


# aggregate_method

Enumerator used for  increased readability and error safety.
Defines the possible ways to aggregate.

```
class aggregate_method(Enum):
    COUNT = 'count'
    MIN = 'min'
    MAX = 'max'
    AVG = 'avg'
    SUM = 'sum'
```