# select_columns_operation

The select_columns_operation is used for filtering out unneeded columns.

[Read the user documentation about the select_columns_operation](../../user_docs/operations/select_columns_operation.md)

```
class selectColumnsOperation(oi.operation_interface):
```
## init
`columns` contains the list of columns (by column name) that are to be selected.

```
def __init__(self):
    self.columns:list = []
```
## hash
```
def hash(self, inputs: list[connection]=[], hash = None) -> str:
    if hash is None:
        hash = hashlib.md5()
    cols = sorted(self.columns)
    for col in cols:
        hash.update(bytes(col, 'utf-8'))
    return super().hash(inputs, hash)
```
## insert_parameters


```
def insert_parameters(self, params:dict) -> Exception | None:
    if params is None:
        return Exception("parameters missing")
    if params.get("columns") is None:
        return Exception("parameter 'columns' missing")
    elif not isinstance(params.get("columns"), list) :
        return Exception("parameter 'columns' must be array")
    else:
        self.columns = params.get("columns") #type: ignore
```
## call

Included columns go to port 0.<br> 
Excluded columns go to port 1. 

```
def __call__(self, inputs: list[connection]=[]):
    for con in inputs:
        if isinstance(con.payload, pd.DataFrame):
            return [
                {"port":0, "output": self.select_columns(con.payload, self.columns, include_columns=True)}, 
                {"port":1, "output": self.select_columns(con.payload, self.columns, include_columns=False)}
                ]
    return Exception("No dataframe was found in inputs")
```

## select_columns

```
def select_columns(self, dataset: pd.DataFrame, col_names_list: list[str], include_columns: bool = True) -> pd.DataFrame | Exception:
        dataset_cols = dataset.columns.tolist()
        for col in col_names_list:
            if col not in dataset_cols:
                return Exception("A column to select was not found in the dataset")

        # Trim whitespace from each column name
        col_names_list = [col.strip() for col in col_names_list]
        
        # Filter columns based on the Include_Exclude parameter
        if include_columns:
            filtered_dataset = dataset[col_names_list]
        else: #(exclude columns)
            filtered_dataset = dataset.drop(columns=col_names_list)

        return filtered_dataset

```