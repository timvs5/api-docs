# pipeline.py

## pipelineObject

The purpose of this object is to contain the information needed to execute a pipeline. Such as the operations, connections and a pipelineMemory.

```
class pipelineObject:
    
    def __init__(self):
        self.request_id:str | None= None
        self.session_id:str = ""
        self.operations:list[Operation] = []
        self.connections:list[connection] = []
        self.operations_by_id:dict[int, Operation] = {}
        self.operation_hashes_by_id:dict[int, str] = {}
        self.pipeline_memory:pipelineMemory = pipelineMemory()
```

## pipelineMemory

The purpose of this object is to retain and give the return values of executed operations.

Return values are saved in a dictionary with as key the operation it comes from and the 'port' through which it was released. 

```
class pipelineMemory:
   
    def __init__(self) -> None:
        self.operations_memory: dict[tuple[Operation, int], input_output_object] = {}

    def get_output(self, operation:Operation, port:int=0): 
        return self.operations_memory.get((operation, port))
    
    def get_outputs(self, operations:tuple[Operation] | None, ports:tuple[int] | None):
        if operations is None or ports is None:
            return None
        return tuple(self.get_output(operation, port) for operation, port in zip(operations, ports))
    
    def add_output(self, operation:Operation, port:int, iuo:input_output_object):
            self.operations_memory[operation, port] = iuo

    def get_full_memory(self):
        return self.operations_memory
```

