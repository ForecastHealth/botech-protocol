# Botech Protocol
**Authors:** Rory Watts, Director, Forecast Health Australia

## Overview
Botech (back of the envelope technology) is a protocol for the simulation of state transition models. The protocol dictates model creation and execution. Models are composed of nodes and links: nodes store values (`balance`), and links transmit values using a `transition rate`. Both `balance` and `rate` are arrays of 202 elements, representing populations from Males aged 0 to Females aged 100. Links have a direction from source to target, thus models are `DiGraph`s. All model components are nodes or links: people, resources, costs, constants, etc.

Botech is a superset of traditional state-transition models, adding features beyond expected behaviors. Botech is not software itself but has implementations that take a `model file` and produce an `observation file` based on simulation. Implementation requires:
- Code to parse the model file and create the runtime environment.
- Code to simulate the model.
- Code to log and export observations.

Botech files should be shareable and are intended to be plain text (e.g., JSON for configuration and CSV for observations).

## Background
Botech aims to:
- Produce complex, understandable, and reproducible models.

Complex models capture intricate behaviors. Understandable models allow assessment without programming expertise. Reproducibility ensures transparent assumptions and adaptable models. The protocol and implementation are open-source to leverage distributed intelligence and ensure good modeling practices.

## Clarifications
- The protocol is under development, primarily through the `botech-python` implementation.
- Links in other contexts may be referred to as `edges` or `vertices`.
- Programming conventions typically follow Python naming conventions (e.g., dictionaries `{}`, lists `[]`).

## Specification: The model file
### Data Format
The model configuration should be plain text, suitable formats include `json`, `yaml`, and `xml`.

### Architecture
A typical configuration file:
```json
{
  "nodes": [],
  "links": [],
  "runtime": {},
  "metadata": {},
  "subroutines": []
}
```
- `nodes`: List of dictionaries where each element is the configuration of a node.
- `links`: List of dictionaries where each element is the configuration of a link.
- `subroutines`: List of dictionaries where each element is a subroutine.
- `runtime`: Optional dictionary of runtime variables.
- `metadata`: Optional dictionary of metadata key-value pairs.

## Specification: Node
Nodes hold a balance.

### ID [required]
Components have a unique ID, recommended to use `ULID`.

### Label [optional]
Plain-text, human-readable label.

### Description [optional]
Plain-text description of the node's purpose.

### Subtype [optional]
May change the behavior of the node or link.

### Generate Array [optional]
Nodes can generate arrays to store in their `balance`. 
Specifications of the `datafetcher` are detailed elsewhere.

### Flush [optional]
Boolean, whether to reset the balance of nodes to 0 at the end of each timepoint.

### Age [optional]
Boolean, whether to "age" the balance of nodes (shift values by 1 index) at each trigger.

## Specification: Link
Links take a balance from a `source` node, and pass it (or part of it) to a `target` node.

### ID [required]
Components have a unique ID, recommended to use `ULID`.

### Label [optional]
Plain-text, human-readable label.

### Description [optional]
Plain-text description of the node's purpose.

### Source and Target [required]
Links require `source` and `target` IDs of the nodes they connect.

### Generate Array [optional]
Links can generate arrays to determine their `transition rate`.
Specifications of the `datafetcher` are detailed elsewhere.

### Trigger [optional]
Links can occur after a certain `trigger`.
Specifications of the `trigger` are detailed elsewhere.

## Specification: Runtime
### Start and End [optional]
- `startYear`: Integer indicating the start index/year (e.g., 2024).
- `endYear`: Integer indicating the end index/year (exclusive, e.g., 2030).

## Specification: Metadata
Key-value pairs for metadata about the model, e.g.:
```json
{
  "metadata": {
    "author": "Botech",
    "title": "Metadata Example",
    "description": "An example metadata dictionary",
    "year": "2023"
  }
}
```

## Specification: Subroutine
Order of operations in a model, allowing specification of event sequences.
If not links or nodes are specified, the method is called on all nodes.

### method
The string corresponding to the method that the node or edge should perform e.g. `flush` or `age` or `generate_balance`.

### narration
A string description of what the subroutine is doing e.g. "distribute diagnosed cancers to stage I -> IV"

### batch
Applies only to subroutines where one node is running the method `push_balance_to_edges` to many edges. If `False`, then the balance is recalculated after each method (e.g. foo -> bar, recalculate balance, foo -> baz, recalculate balance, foo -> bux, recalculate balance). If `True`, then the balance is the same for all method calls, then the total is subtracted from the balance (e.g. foo -> bar, foo -> baz, foo -> bux, recalculate balance).

### included_edges [optional]
The set of link IDs to call the method on.

### excluded_edges [optional]
The set of link IDs not to call the method on.

### included_source_nodes [optional]
The set of source nodes to call the method on.

### excluded_source_nodes [optional]
The set of source nodes not to call the method on.

### included_target_nodes [optional]
The set of target nodes to call the method on.

### excluded_target_nodes [optional]
The set of target nodes not to call the method on.

## Specification: The Botech Runtime Environment
The environment instantiates methods and data sources required for model simulation, running the `simulate` method to produce the `observation file`.

### Model Configuration
The environment creates a Botech Graph from the model configuration, translating nodes and links into components that maintain state, adjust balances, transmit values, and generate arrays.

### DataFetchers and DataSources
Generate or retrieve data for nodes and links. Datafetchers are APIs returning values in a 2x101 array, registered with the environment.

### Observations
Attach an `observation file` to the environment for data retrieval instead of using datafetchers, ensuring replicable models without sharing databases.

## Using the Observations as a DataSource
Attaching an `observation file` allows model replication and assumption modification, requiring identical runtime order.

## Specification: The Simulation Method and Runtime Events
The `simulate` method runs from start to end time, iterating through subroutines and performing specified methods on nodes. Events are recorded in the `observation file`.

## Additional Features
### On-Graph Calculations
Links can use source node values for dynamic interactions with target nodes.

### Surrogates (Nodes as Links)
Surrogate nodes affect transition rates around them, providing transparency in complex processes.

### Triggers
Trigger actions based on conditions, enabling complex behaviors and resource allocation.

## Implementations
### Python
`botech-python` is in development by Forecast Health Australia.

## Contributing
Email info@forecasthealth.org to contribute to Botech development.

## License
Botech and its implementations are licensed under the Apache 2.0 license. [License](https://github.com/ForecastHealth/botech-protocol/blob/master/LICENSE)

## Similar Projects
- [The OneHealth Tool (OHT) and Integrated Health Tool (IHT)](https://www.avenirhealth.org/)
- [The C4P Implementation by Levin Morgan LLC](https://www.levinmorgan.com/publications)
- [The TLO Model from the Thanzi La Onse collaboration](https://www.tlomodel.org/)
- [Vivarium by IHME](https://github.com/ihmeuw/vivarium)
- [Squiggle](https://github.com/quantified-uncertainty/squiggle)
- [TreeAge Software](https://www.treeage.com/)
- [The Policy1 Modelling Platform](https://www.policy1.org/)
- [The Optima Model](https://optimamodel.com/)

## Acknowledgments
We acknowledge contributions from the World Health Organization, Cancer Research UK, Mr. Joseph Dunne, Mr. Timothy Knowles, and Mr. Angus Watkins.
