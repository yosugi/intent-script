# IntentScript

In the age of AI-generated code, the challenge has shifted from writing code to clearly communicating what we want to do.
IntentScript is a domain-specific language (DSL) designed to bridge that gap—a language for describing intent, created for both humans and AI.

This document introduces the design philosophy and language specification of IntentScript.

## Overview

Unlike traditional programming languages that focus on how to implement something, IntentScript is optimized for expressing what to accomplish.

### Key Design Principles

* Designed for LLM interpretation and implementation
* YAML-based syntax (no special parser required)
* Avoidance of Turing-completeness
* Support for gradual formality (from natural to structured language)
* Zero learning curve
  * Natural language is acceptable; non-programmers can start using it right away
* Zero setup cost
  * Just feed the script to an LLM and it's ready to run
* AI-driven error handling and flexible exception management

## Features

* Structuring via types and attributes
* Declarative function definitions for logic and calculations
* Pipeline syntax for filtering and aggregating data
* Rich support for comments and natural language annotations
* Conditional branching for dynamic value switching
* Extensibility via external definitions

## Specification

Detailed specifications are provided in the file:

* [`intent-script-spec-en.md`](intent-script-spec-en.md)

Topics covered include:

* Design principles for intent description
* The 3 levels of gradual formality
* Syntax for defining entities and functions
* Attribute annotations
* Type system structure
* Usage of pipeline syntax and standard functions

## Examples

```yaml
Product:
  name: string{required, max_length: 100}
  price: int{min: 1}
  discount_rate: 0.8
  tag: list<string>{max_items: 5}
  discount_price: int{
    derive: price * (1 - discount_rate)
  }
```

Natural language definitions are also supported:

```yaml
calculate_shipping_cost:
  description: "Calculate shipping cost based on weight and distance"
  inputs:
    weight: int{unit: kg}
    distance: int{unit: km}
  output: int{currency: JPY}
  logic: |
    Base cost is 500 yen.
    Add 200 yen per kg.
    Add 100 yen for every 100 km.
```

## Project Structure

```
├── intent-script-spec-en.md # IntentScript language specification
├── intent-script-spec.md    # IntentScript language specification(Japanese)
├── LICENSE                  # MIT License
├── README-en.md             # This file
└── README.md                # README(Japanese)
```

## Future Work

Version 0.0.1 remains a minimal configuration.
The following items are under consideration for future extension.

* Error handling and validation expression
* Enumerated, dependent, and conditional constraint types
* Composite types (e.g., union/intersection)
* Generic entity definitions (e.g., User<T>) for reuse
* Modular package structure for large-scale systems (namespaces and scope management)
* Template definitions and reuse mechanisms
* Consistency checking of natural language comments and AI-powered suggestion support

## Acknowledgements

The idea and documentation of IntentScript emerged through conversations with AI.
We thank AI for its collaborative contribution.

## License

Released under the MIT License.
