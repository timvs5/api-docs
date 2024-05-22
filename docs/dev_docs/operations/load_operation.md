# load_operation

The load_operation is used for loading data and gevis them as pandas dataframes.

[Read the user documentation about the load_operation](../../user_docs/operations/load_operation.md)

```
class loadOperation(oi.operation_interface):
```
## init
`base_sample_size` is the default amount of rows given if *sample* is True.

`sample`: If sample is False, it does nothing. If it is True, load_operation will only return the amount of rows defined in *base_sample_size*.
If it is an integer, load_operation will return that amount of rows.

`filename`defines the name of the file to be loaded in. 

`randomize_sample`: If True will randomize what rows are returned if only a sample is returned. If False, load_operation will consistently return the first x rows.

`loadable_files` defines what files the load_operation has access to. Filenames not in this list will never be loaded.

```
    def __init__(self):
        self.base_sample_size = 25
        self.filename:str
        self.sample:bool | int
        self.randomize_sample:bool = True
        self.loadable_files = ['mtcars', 'titanic', 'iris','student-attitude']
```
## hash
```
    def hash(self, inputs: list[connection]=[], hash = None) -> str:
        if hash is None:
            hash = hashlib.md5()
        hash.update(bytes(self.filename, 'utf-8'))
        if type(self.sample) is bool:
            hash.update(bytes(str(int(self.sample == True)), 'utf-8'))
        else:
            hash.update(bytes(str(self.sample), 'utf-8'))
        return super().hash(inputs, hash)
```
## insert_parameters

```
    def insert_parameters(self, params:dict) -> Exception | None:
        if params is None:
            return Exception("parameters missing")
        if params.get("filename") is None:
            return Exception("parameter 'filename' missing")
        else:
            self.filename = params.get("filename") #type: ignore
        self.sample = string_to_bool_or_int(params.get("sample", False))
        if type(self.sample) == bool:
            if self.sample == True:
                self.sample = self.base_sample_size
            else:
                 self.sample = -1
        if params.get("randomize_sample") is not None:
            self.randomize_sample = bool(params.get("randomize_sample"))
```
## call
```
    def __call__(self, inputs):
        return [{"port":0, "output":self.load_data(self.filename, self.sample)}]
```

## load_data
This function loads in data as a dataframe. This function is used by 'call'.

A filename that can not be found will resutl in an Exception.

```
def load_data(self, filename:str, sample_size:int) -> pd.DataFrame | ValueError:
    if filename in self.loadable_files:
        data:pd.DataFrame = pd.DataFrame(pydata(filename))
        if sample_size > -1:
            if self.randomize_sample:
                data = data.sample(n=sample_size)
            else:
                data = data.head(sample_size)
        return data
    else:
        return ValueError(f"Invalid dataset name. Choose from {self.loadable_files}")
```