# filter_rows_operation

[Read the user documentation about the filter_rows_operation](../../user_docs/operations/filter_rows_operation.md)

## accessory classes
### comparator

Used for comparisons.

```
class Comparator(Enum):
    EQUAL = '=='
    NOT_EQUAL = '!='
    GREATER_THAN = '>'
    LESS_THAN = '<'
    GREATER_EQUAL_THAN = '>='
    LESS_EQUAL_THAN = '<='

    def compare(self, x, y):
        return eval(f"{x} {self.value} {y}")
```

### comparee

*Compare_value* is used when comparing rows in a dataframe with a static value.

*Compare_column* is used when comparing one row with another.

```
class compare_column():
    def __init__(self):
        self.column: str

class compare_value():
    def __init__(self):
        self.value: str | int | float

Comparee = compare_value | compare_column
```

## filterRowsOperation

```
class filterRowsOperation(oi.operation_interface):
```

### init

```
def __init__(self):
        self.conditions:list[Condition]
        self.include:bool = True
        self.set:set_type = set_type.INTERSECT
```

### hash
```
def hash(self, inputs: list[connection]=[], hash = None) -> str:
    return super().hash(inputs, hash)
```

### insert_parameters
```
def insert_parameters(self, params:dict) -> Exception | None:
    self.conditions = []
    if params.get("conditions") is None:
        return Exception("No conditions were given.")
    paramconditions:list = params.get("conditions") # type: ignore
    for paramcond in paramconditions:
        column = paramcond.get("column")
        comparee_val = paramcond.get("compare_with")
        if paramcond.get("compare_with_type") == "value":
            comparee = compare_value()
            comparee.value = comparee_val
        elif paramcond.get("compare_with_type") == "column":
            if not isinstance(comparee_val, str):
                return Exception("'compare_with' should be a string to be used as a column.") 
            comparee = compare_column()
            comparee.column = comparee_val
        else:
            return Exception("Invalid value for 'compare with type'")
        comparator:Comparator
        result = self.get_comparator(paramcond.get("comparator"))
        if isinstance(result, Exception):
            return result
        comparator = result
        condition = Condition()
        condition.insert_vars(column, comparator, comparee)
        self.conditions.append(condition)

    if params.get("include") is not None:
        self.include = bool(params.get("include"))
        
    self.set = set_type.INTERSECT
```

### call
```
def __call__(self, inputs: list[connection]=[]):
    for con in inputs:
        if isinstance(con.payload, pd.DataFrame):
            df:pd.DataFrame = con.payload.copy()
            if self.set == set_type.INTERSECT:
                for condition in self.conditions:
                    df = condition(df) # type: ignore
            if self.set == set_type.UNION:
                conditioned_dataframes = []
                for condition in self.conditions:
                    conditioned_dataframes.append(condition(df)) # type: ignore
                df = pd.concat(conditioned_dataframes)
            if not self.include:
                df_exclude = con.payload.copy()
                merged_df = pd.merge(df_exclude, df, on=list(df.columns), how='left', indicator=True)
                df_exclude = merged_df[merged_df['_merge'] == 'left_only'].drop(columns='_merge')
                df = df_exclude
            return [{"port":0, "output": df}]
    return Exception("No dataframe was found in inputs")
```

### get_comparator

This takes a string and returns the corresponding [Comparator](#comparator)

```
def get_comparator(self, paramcomparator:str) -> Comparator | Exception:
    for comp in Comparator:
        if comp.value == paramcomparator:
            return comp
    return Exception("Invalid comparator")
```
