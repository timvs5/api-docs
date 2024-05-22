# connections.py

## cport
cport is an enumerator used for increased readability and error safety.
It represents the 'in' and 'out' ports of a connection.

```
class cport(Enum):
    FROM = 0
    TO = 1
```

## Connection
[Read the user documentation about connections](../user_docs/connections_info.md)<br>
A payload can be added to store the output of the 'from_operation'.

```
class connection:
    from_operation: Operation
    to_operation: Operation
    ports: dict[cport, int]
    payload:input_output_object | None = None
    
    def __init__(self, from_op:Operation, to_op:Operation, ports:dict[cport, int]):
        self.from_operation = from_op
        self.to_operation = to_op
        self.ports = ports

    def add_payload(self, payload):
        self.payload = payload

```