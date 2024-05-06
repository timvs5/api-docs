# Operations
## What is an operation?
<p>An operation is a command. Something you want to be done.
A load operation for example lets you load data from a database.</P>

<P>An operation may take parameters that define the variable parts of it's job.
The load operation for example takes a 'filename' parameter that tells it what datafile it needs to load.</P>

<P>An operation may also take inputs. This is data it receives from other operations. The 'duplicate' operation for example must get the data it duplicates from a previous operation.</P>

<p>An operation has ports, both for input as output. These allow some operations to take multiple inputs (like the merge operation) or give multiple outputs (like the duplicate operation).</p>

## Defining an operation
<p>An operation is defined in JSON as follows:</p>

```
{"id":ID,"operation":operation_name,"parameters":{"parameter_name_1":parameter_1, "parameter_name_2":parameter_2, ...}}

Example:
{"id":2,"operation":"load","parameters":{"filename":"titanic"}}
```

<p>It is a dictionary that defines an id, an operation and a dictionary of parameters</p>

### id
<p>The id is an integer of your choice. Its outputs will be defined by this id in the response so you can know which operation returned what. It is also used to define the connections.</p>

### operation
<p>This is a string with the name of the operation you want to be executed.</p>

### parameters
<p>This is a dictionary in which you define the parameters for the operation. Each operation has its own set of parmeters it takes.</p>
