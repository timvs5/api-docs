# Connections
## What is a connection?
<p>A connection describes a dataflow from one operation to another. For example: a connection from a load operation to a select_columns operation will send the data retrieved in the load operation to the select_columns operation so it can perform its action on that data.</p>

## Defining a connection

<p>A connection is defined in JSON as follows:</p>

```
{"from":operation_id, "to":operation_id, "from_port":port_nr, "to_port":port_nr}

Example:
{"from":1, "to":2, "from_port":0, "to_port":0}
```

<p>It is a dictionary that defines a 'from' operation where the data comes from and a 'to' operation where the data goes to. here you fill in the id's you used when defining the operations. a 'from_port'. It also defines a 'from_port' and a 'to_port' that describe from which port in the one operation to which port in the other operation the data needs to go.</p>
