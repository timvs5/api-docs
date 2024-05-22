# blueprint

This is the general structure that all operations follow.

Examples are from the [load_operation](load_operation.md)

## init
The init function defines the variables and their types.<br>
Default values may also be defined.

Example:
```
def __init__(self):
    self.base_sample_size = 25
    self.filename:str
    self.sample:bool | int
    self.randomize_sample:bool = True
    self.loadable_files = ['mtcars', 'titanic', 'iris','student-attitude']
```

## hash
This function is important for caching.

The hash function calculates a hash using set variables and a list of inputs if given.
The inputs are given as a list of [connections](../connections.md). The hash is calculated based on what is in the payloads of these connections.

Example:
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
This function inserts parameters into the variables of the operation. 
These are to be given as a dictionary with as key a string representing the variable and as value the value to be inserted.
The keys preferably have the same name as the variables they represent.

Example:
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
This is how the operations are executed.
It always accepts inputs but does not necessarily use them. 
It returns a list of outputs in the form of dictionaries with the port on which the output is given and the actual object or dataframe it outputs.

Example:
```
def __call__(self, inputs):
    return [{"port":0, "output":self.load_data(self.filename, self.sample)}]
```

In this case it calls upon another function in the object to determine the output.

## others 
Other functions can be implemented to make the code mroe readable.
These should be considered private functions.


Example:
```
def load_data(self, filename:str, sample_size:int) -> pd.DataFrame | ValueError:
    if filename in self.loadable_files:
        # change this later
        if filename == "student-attitude":
            data:pd.DataFrame = pd.read_excel(os.path.join(os.path.dirname(__file__), 'extra_datasets', 'attitudereformat.xlsx'))
        else:
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