# IntentScript Language Specification

## Basic Concept

IntentScript is a language concept designed to express "what to do" to AI.
While traditional programming languages focus on "how to implement," IntentScript focuses solely on describing intent and objectives.

Its natural-language-friendly expressions and comments are designed with interpretation by LLMs in mind.
Rather than serving as a static specification language, IntentScript aims to be a collaborative foundation that enables shared and evolving intent between humans and AI.

## Language Design Principles

* YAML-based syntax

  * No dedicated parser needed
  * Aims for a balance between formal and natural language

* Avoidance of Turing-completeness

  * Recursive and looping control structures are intentionally excluded
  * Focused on intent structuring, freeing users from implementation details

* Support for gradual formality

  * Balances formal rigor with natural-language flexibility
  * Allows coexistence of different abstraction levels within the same system

* Zero learning curve

  * Usable by anyone familiar with natural language
  * No prior programming knowledge required

## File Format and Extension

IntentScript files use the `.is.yml` extension:

* `.is`: identifies the file as an IntentScript definition
* `.yml`: indicates YAML-based syntax

This double extension allows editors and tools to recognize the file as YAML while still enabling IntentScript-specific features like highlighting or linting.

Examples:

* `order-entity.is.yml`: entity definition for orders
* `shipping-function.is.yml`: function for shipping fee calculation
* `product-catalog.is.yml`: entity definition for a product catalog

The filename convention is flexible—only the extension `.is.yml` is required.

## Gradual Formality

"Gradual formality" refers to the spectrum between natural language and strict formal syntax.
IntentScript allows the mixing of different levels of abstraction depending on user preference and context.

These levels are guidelines, not hard categories.

### Level 1 (Formal)

```yaml
User:
  name: string{required, max_length: 50}
  email: string{required, format: "email"}
  age: int{min: 18}
```

### Level 2 (Mixed)

```yaml
User:
  name: text, required, max 50 characters
  email: valid email address, required
  age: number, at least 18
```

### Level 3 (Natural language)

```text
A user has a name, email address, and age.
The name is required and must be no more than 50 characters.
The email address must be valid and is also required.
The age must be at least 18.
```

Users can freely move between these levels based on their needs.

## Language Structure and Basic Syntax

### Element Definition

All definitions in IntentScript (entities, functions, etc.) follow a consistent format:

```yaml
User:
  name: string{required, max_length: 50}
  email: string{required, format: "email"}
  age: int{min: 18}
```

You may also use the `def` keyword or append metadata as options:

```yaml
def Product{kind: entity}:
  name: string{required, max_length: 100}
  price: int{currency: JPY}
```

* `def` makes it explicit that the block is a definition (useful in formal documentation or large systems)
* Metadata can indicate the kind or additional attributes of the element

## Type System

Supported types:

* Primitive types: `string`, `int`, `boolean`, `date`, etc.
* Collection types: `list<T>`, `map<K, V>`
* User-defined types: references to other entities

Example of generics:

```yaml
order_items: list<Product>
product_prices: map<string, int>
```

## Property Options

Properties can use `{}` to specify constraints and annotations:

```yaml
name: string{required, max_length: 100}
price: int{currency: JPY}
```

## Comments and Special Characters (YAML)

IntentScript supports YAML-style comments:

```yaml
# This is a full-line comment
entity:
  name: string  # Inline comment
```

Comments are useful for supplementing intent even in formal-level (Level 1) definitions:

```yaml
User:
  name: string{required, max_length: 50}  # Must be filled, max 50 characters
  email: string{required, format: "email"}  # Required and must be valid format
  age: int{min: 18}  # Age must be 18 or older
```

If special YAML characters are used in expressions (such as `>`, `{`, `|`), quoting them is recommended:

```yaml
filtered_items: list<int>{
  derive: "items |> filter(_ > 100)"
}
```

## YAML Literals

Standard YAML literal forms are valid:

```yaml
simple_string: "Hello World"
multiline_string: |
  This is a
  multi-line string

int: 28
float: 3.14

active: true
disabled: false

created_at: 2006-01-02T15:04:05Z

colors:
  - red
  - green
  - blue

colors: [red, green, blue]

point:
  x: 10
  y: 20

point: {x: 5, y: 15}
```

## Entities and Data Models

### Entity Definition

An entity expresses structured intent about data models and serves as a foundation for AI-generated logic or validation.

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

## Function Definition

Functions describe the purpose and structure of processing in a declarative manner, intended for AI-assisted implementation.

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

## Pipeline Syntax for Data Processing

Pipeline syntax allows intuitive and declarative expression of data flow and transformation.
Use the `|>` operator to chain operations.

```yaml
discount_prices: list<int>{
  derive: items
          |> filter(_.in_stock)
          |> map(_.price * (1 - _.discount_rate))
          |> filter("_ > 0")
}
```

### Built-in Pipeline Functions

* `map`: transform elements

  ```yaml
  doubled_prices: list<int>{
    derive: prices |> map("_ * 2")
  }
  ```

* `filter`: select elements based on condition

  ```yaml
  positive_numbers: list<int>{
    derive: numbers |> filter("_ > 0")
  }
  ```

* `sum`: calculate total

  ```yaml
  total_amount: int{
    derive: order_items |> map(_.price * _.quantity) |> sum
  }
  ```

* `sort`: order elements

  ```yaml
  sorted_values: list<int>{
    derive: values |> sort
  }
  ```

* `reduce`: combine into a single value

  ```yaml
  multiplied_number: int{
    derive: numbers |> reduce("_1 * _2", 1)
  }
  ```

### Placeholder Syntax

* Single argument: `_` refers to the item, `_.property` refers to a field

```yaml
filtered_products: list<Product>{
  derive: products |> filter(_.price > 100)
}
```

* Multiple arguments: `_1`, `_2`, etc.

```yaml
sorted_scores: list<int>{
  derive: scores |> sort(_2 - _1)
}
```

## Use of Natural Language

Natural language can be used throughout structured definitions and pipeline expressions.
This enables expressive and flexible descriptions that AI can interpret.

### Attribute Description with Natural Language

```yaml
Product:
  name: string{required}
  price: int{currency: JPY}
  description: "Detailed product description. Supports Markdown."

  availability_rule: |
    Normally available if in stock.
    For limited items, only members can purchase.
    Pre-orders accepted two weeks before release.

  discount_strategy: |
    - 10% discount for products over 5000 yen
    - Extra 5% off during sales
    - Gold members get an additional 5% off at all times
```

### Natural Language in Pipelines

```yaml
top_products: list<Product>{
  derive: products
          |> "exclude out-of-stock items"
          |> "sort by sales volume descending"
          |> "take top 5"
}
```

### Purely Natural Language Definitions

IntentScript also allows purely natural language or comment-based definitions:

```yaml
# average: a function that returns the average of a list of numbers
```

Such expressions are not formal syntax, but help guide AI and support future extensibility.

## Input and Output Notes

IntentScript focuses on describing intent, not I/O specifics.
However, optional comments can document expected interfaces:

```yaml
# Input: received via stdin in CSV format
# Output: one line per result to stdout
```

## Future Considerations

While this is a minimal specification for POC purposes, the following extensions are under consideration:

* Error handling and validation mechanisms
* Enumerated, dependent, and conditional constraint types
* Composite types (union/intersection)
* Generic entities (e.g., `User<T>`) to support reuse
* Package systems for large-scale development
* `include` support for modular and reusable specs
* Templates for recurring patterns
* Semantic validation of comments and natural language
* AI-assisted clarification and suggestions

## About This Document

* Version: 0.0.1
* Status: Minimal specification for proof-of-concept
* Authorship: This specification was created through dialogue with generative AI.
  We express our sincere gratitude for its significant contributions to shaping this document.
* License: MIT License

This document is the starting point for evolving IntentScript into a foundational language for communicating intent to AI.
It is designed to support further experimentation, discussion, and adoption.
