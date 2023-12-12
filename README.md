# Botech
Authors: Rory Watts, Director, Forecast Health Australia

## Overview
Botech (back of the envelope technology), is a protocol for the creation and simulation of state transition models.
State transition models are composed of nodes and edges. 
Nodes store values as a `balance`, edges transmit values using a `transition rate`.
Edges have a direction. 
Therefore, the representation of these models is a `DiGraph` or `Directional Graph`.
Fundamentally, **all components of the model are nodes or edges**: people, resources, costs, constants, these should all be represented as nodes, or edges.
Furthermore, a node's balance is an **array** of 202 values, from males aged 0, to females aged 100.

Botech is a superset of the functionality of traditional state-transition models.
That is, it contains all the expected behaviour of state-transition models, and adds features to them.
Botech itself is not a piece of software, but has implementations.
Botech is loosely designed around the unix philosophy of doing one thing well.

The implementation takes a `model configuration file` and produces an `observation file` based on simulating that model.
Therefore, an implementation requires:
- Code to interpret the model file, and create the runtime environment
- Code to simulate the model and its behaviours
- Code to log and export the observations
Botech *should be shareable*, and so both the model configuration file and observation file are intended to be plain text (e.g. a JSON configuration and a CSV of observations).

## Background
The Botech protocol arose from the need to do the following things:
- Produce complex models...
- that people understand...
- which are reproducible

Complex models mean complex behaviours can be captured (e.g. feedback loops, dynamic changes to the model).
Understandable models mean that different groups can assess and use models.
Reproducibility means that assumptions are transparent, and that groups can adapt and rebuild models.


## Clarifications
- The protocol is under development. It is primarily under development through trial and error via the `botech-python` implementation.
- We use `edges` and `links` interchangeably in this document
- When referring to programming conventions, we are typically using Python naming conventions e.g. dictionaries for `{}` and lists for `[]`


## Specification: The Model Configuration File
### Data Format
The intention is for the model configuration to be plain text.
We believe `json`, `yaml`, `xml` and similar formats to be suitable.

### Architecture
A typical configuration file will consist of the following:
```
{
  "nodes": [],
  "links": [],
  "runtime": {},
  "metadata": {},
  "subloops": []
}
```
NOTE - `links` are synonymous with `edges` here.

`nodes` is a list of dictionaries representing nodes. 
`links` is a list of dictionaries representing links.
`runtime` is optional, and is a dictionary of runtime variables.
`metadata` is optional, and is a dictionary of key value pairs to be considered metadata.
`subloops` is optional, and is a list of dictionaries representing subloop configurations.

### Nodes and Links
Nodes and links are a collection of properties which forms the model structure. 
For our purposes, we will deal with nodes and links collectively when we can, and call them `components`.

#### ID [required]
Components have an ID. An ID should be unique. We choose to use `ULID`. 

#### Label [nodes only, required]
Nodes have a plain-text, human-readable label.

#### Order [optional]
An integer.

Order for nodes specifies the runtime order.
E.g. if `foo`, `bar` and `baz` have orders 0, 2, and 1, then for any `subloop` where they are all invoked, they will run in order `foo`, `baz`, `bar`.
More than one node can have the same order, and this is useful if there is no meaningful difference when two nodes run.

Order for links specifies their `batch` order.
Batch order is best illustrated through an example.
Consider a node with balance 10 called `foo`, it has two links, to `bar` and to `baz` (`foo -> bar`, `foo -> baz`).
Both `foo -> bar` and `foo -> baz` have transition rates of 0.5, meaning they will take 50% of `foo`'s balance.

In the first scenario, `foo -> bar` has order 0, and `foo -> baz` has order 1. This means:
- `foo` will transmit 50% of its balance to `bar` (5)
- `foo` subtracts 5 from its balance
- `foo` will then transmit 50% of its remaining balance to `baz` (2.5)
- `foo` subtracts 2.5 from its balance
- the balance in `foo` is 2.5, the balance in `bar` is 5, and the balance in `baz` is 2.5

In the second scenario, `foo -> bar` and `foo -> baz` have order 0. This means:
- `foo` keeps track of its balance before it transmits anything
- `foo` will transmit 50% of that balance to `bar` (5)
- `foo` will transmit to% of that balance to `baz` (5)
- `foo` subtracts that balance from itself
- the balance in `foo` is 0, the balance in `bar` is 5, and the balance in `baz` is 5

Therefore, batches for links are more complicated, but important to understand. One is more suitable if a user is thinking about rates. One is more suitable if a user is trying to divide a node in the traditional sense.


#### Source and Target [links only, required]
Links require two properties: `source` and `target`.
These are the ID of the source node, and the ID of the target node respectively.

#### Description [optional]
Components may have a plain text description, which informs users about the purpose of the element.

#### Generate Array [optional]
Nodes can generate arrays to store in their `balance`. 
Links can generate arrays to create their `transition rate`.
This property is a dictionary with the following features:
- `method`: a string which matches the name of the method to generate the array
- `parameters`: a dictionary containing key value pairs of the parameters required to generate the array
- `data_fetcher_label`: [optional] a string representing the label of the attached `datafetcher` [see below]
- `cache`: [optional] a boolean. True if the original value should be cached (useful for intensive requests e.g. from `datafetcher`s)
- `fetch`: [optional] instructions on when in the runtime to fetch the array (coming soon)

#### Flush [nodes only, optional]
A boolean representing whether, at the the end of each timepoint (e.g. one year), the balance of the nodes should be reset to 0s ([[0, 0, 0...], [...0]]).
This can be useful if the node is not a state, but a collection or constant of some sort.

#### Age [nodes only, optional]
A boolean representing whether, when triggered, the balance of the nodes should be "aged".
Here, this means the values of each element in an array are shifted by 1 index.
Therefore, the first array will contain 0, and the final array will be removed.
This simulated ageing: males aged 1 will now be aged 2, females aged 50 will now be aged 51 etc.
This also implies anyone aged 100 will be removed from the array.

### Runtime
#### start and end [optional]
start: An integer indicating the index at which a model should start e.g. 2023
end: An integer indicating the index at which a model should end (inclusive of that year) e.g. 2030
This is optional, and defaults to 0 and 1 respectively.

Using the above example:
A model with this specification should have eight timepoints: 2023, 2024, 2025, 2026, 2027, 2028, 2029, 2030

Note - the specification of a timepoint is arbitrary.

### Metadata
A series of key value pairs containing strings that can be used to hold metadata about the model.
For example:
```
{
  "metadata": {
    "author": "Botech",
    "title": "Metadata Example",
    "description": "An example metadata dictionary",
    "year": "2023"
  }
}
```


### Subloops
The order of operations in a model is important.
Subloops allow users to specify when things happen within runtime.
For example, each year you may wish for ageing to occur first, then births, then deaths etc.
These are behaviours which can be ordered using subloops.

Subloops are a list of dictionaries, with each dictionary providing information about the behaviour to occur. 
Importantly, the list will run from top to bottom, so the order matters.

The properties of subloops are as follows
```
subloops = [
  {
    "method": "age",
    "narration": "Age the population",
    "all_nodes": False,
    "parameters": {
      "nodes_to_include": [],
      "nodes_to_exclude": []
    }
  }
]
```
`method` is a string, corresponding to a property or method that a node can execute e.g. "age", "flush", "generate_balance".
`narration` is a string which will be written to the `observation file` for each `Event` that occurs during the subloop.
This is useful to distinguish between subloops when debugging or reading logs.
`all_nodes` is a boolean which, if True, will attempt to run the `method` on every node, regardless of whether or not it has an `order`.

## Specification: The Botech Runtime Environment

## Specification: The Botech Implementation

## Specification: The Observation File

## Additional Features: Triggers

## Additional Features: Surrogates

## Attaching Models to External Data Sources

## Implementations
### Python
`botech-python` is currently being developed by Forecast Health Australia.

## Examples
Practical examples or use cases.

## Security Considerations
Any security-related information or best practices.

## Contributing
Guidelines for contributing to the protocol's development.

## FAQs
Answers to common questions.

##  License
The full text of the Apache License 2.0 or a summary with a link to the full license.

## Similar Projects

## Acknowledgments
We acknowledge the great contributions and support made by the World Health Organization. In particular we acknowledge the contributions made by Dr Andre Ilbawi, Dr Filip Meheus, Dr Melanie Bertram, Dr Slim Slama and Dr Tessa Edejer.
We acknowledge contributions made by Annalisa Belloni of Cancer Research UK.
We acknowledge contributions from Mr Joseph Dunne, and Mr Timothy Knowles, who have both contributed to the `botech-python` implementation, as well as other related projects.
We acknowledge Mr Angus Watkins, for the design of the Botech logo.
