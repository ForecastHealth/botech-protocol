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
  "metadata": {}
}
```
NOTE - `links` are synonymous with `edges` here.

`nodes` is a list of nodes. 
`links` is a list of links.
`runtime` is optional, and is a dictionary of runtime variables.
`metadata` is optional, and is a dictionary of key value pairs to be considered metadata.

### Nodes and Links
Nodes and links are a collection of properties which forms the model structure. 
For our purposes, we will deal with nodes and links collectively when we can, and call them `components`.

#### ID [required]
Components have an ID. An ID should be unique. We choose to use `ULID`. 

#### Label [nodes only, required]
Nodes have a plain-text, human-readable label.

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
- 

### Runtime

### Metadata


## Specification: The Botech Runtime Environment

## Specification: The Botech Implementation

## Specification: The Observation File

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
