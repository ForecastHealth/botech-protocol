# Botech
Authors: Rory Watts, Director, Forecast Health Australia

## Overview
Botech (back of the envelope technology), is a protocol for the creation and simulation of state transition models.
Fundamentally, **all components of the model are nodes or edges**: people, resources, costs, constants, these should all be represented as nodes, or edges.
Botech is a superset of the functionality of traditional state-transition models, and can generate complex behaviours from a relatively simple set of rules.
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


## Protocol Specification
- Architecture: Details of the protocol's structure and components.
- Data Format: Description of data structures, encoding, etc.
- Communication: Explanation of how data is transmitted and received.

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
