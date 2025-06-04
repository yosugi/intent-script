# IntentScript Language Specification

## Basic Concept

IntentScript is a language concept designed to express "what to do" to AI.
While traditional programming languages focus on "how to implement," IntentScript focuses solely on describing intent and objectives.

Its natural-language-friendly expressions and comments are designed with interpretation by LLMs in mind.
Beyond static specification, IntentScript serves as a collaborative foundation for shared and evolving intent between humans and AI.

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
* Zero setup cost
  * Supports loading specs from external sources
  * Scripts can be used directly by pasting into an LLM prompt
* Flexible AI execution
  * Error handling and exceptional conditions are delegated to AI judgment

### File Format and Extension

IntentScript files use the `.is.yml` extension:

* `.is`: identifies the file as an IntentScript definition
* `.yml`: indicates YAML-based syntax

This double extension allows editors and tools to recognize the file as YAML while still enabling IntentScript-specific features like highlighting or linting.

Examples:

* `order-entity.is.yml`: entity definition for orders
* `shipping-function.is.yml`: function for shipping fee calculation
* `product-catalog.is.yml`: entity definition for a product catalog

The filename convention is flexible—only the extension `.is.yml` is required.

## Declaring Used Specifications: include

Use include at the top of a file to explicitly declare which specs or definitions are available.This helps LLMs and tools constrain completion and validation.

```
include:
  - raw.githubusercontent.com/yosugi/intent-script/refs/heads/main/intent-script-spec.md
```

- include must be declared at the top level of the file
- All types, functions, and operators defined in the included documents become usable in the current script

## Gradual Formality

"Gradual formality" refers to the spectrum between natural language and formal syntax.
IntentScript allows the mixing of different levels of abstraction depending on user preference and context.

These levels are guidelines, not hard categories.

### The 3 Levels of Gradual Formality

* Level 0 (Natural language): Natural language-centered expression, relying on AI intent analysis
  ```text
  A user has a name, email address, and age.
  The name is required and must be no more than 50 characters.
  The email address must be valid and is also required.
  The age must be at least 18.
  ```

* Level 1 (Mixed): Combination of structured format and natural language
  ```yaml
  User:
    name: text, required, max 50 characters
    email: valid email address, required
    age: number, at least 18
  ```

* Level 2 (Formal): Completely formal syntax, implemented as strict YAML structure
  ```yaml
  User:
    name: string{required, max_length: 50}
    email: string{required, format: "email"}
    age: int{min: 18}
  ```

Note that these levels are convenient classifications. In practice, there are no clear boundaries, and it forms a continuum from formal notation to natural language.

## Language Structure and Basic Syntax

### Element Definition Format

All definitions in IntentScript (values, entities, functions, etc.) follow a consistent format:

```yaml
# Value/constant definition examples
TAX_RATE: 0.1
DEFAULT_CURRENCY: "JPY"
SHIPPING_ZONES: [domestic, asia, worldwide]

# Basic entity definition example: User entity
User:
  name: string{required, max_length: 50}
  email: string{required, format: "email"}
  age: int{min: 18}

# Definition with def keyword and options: Product entity
def Product{kind: entity}:
  name: string{required, max_length: 100}
  price: int{currency: DEFAULT_CURRENCY}  # Using defined constant
```

Element definitions consist of a name followed by a colon (:).
The definition body consists of the value itself, or indented attribute-value pairs.

When necessary, you can specify the def keyword and options (using {} after the name).
These two are independent of each other, and either can be used alone.
Their meanings are as follows:

* def keyword: `def`
  * Explicitly indicates "this is a definition"
  * For formal documents or large-scale systems
* Attributes: `User{kind: entity}:`
  * Explicitly specify type or additional attributes
  * For complex systems or when clear type distinction is needed

### Value/Constant Definition

IntentScript allows defining reusable values and constants:

```yaml
# Simple value definitions
TAX_RATE: 0.1
DEFAULT_CURRENCY: "JPY"
MAX_ORDER_ITEMS: 100

# Complex value definitions
SHIPPING_ZONES: [domestic, asia, worldwide]
ERROR_MESSAGES:
  not_found: "Product not found"
  out_of_stock: "Out of stock"
```

Defined values can be referenced within entity and function definitions:

```yaml
Product:
  price: int{currency: DEFAULT_CURRENCY}
  tax_included_price: int{
    derive: price * (1 + TAX_RATE)
  }
```

### Entity Definition

Entity definitions in IntentScript concisely and structurally express the intent of target data structures, serving as the foundation for specification description and AI-generated processing.

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

### Function Definition

Function definitions in IntentScript explicitly describe the "purpose" and "structure" of processing, adopting a declarative style that assumes AI implementation support.

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

<<<<<<< HEAD
## Function Call Extensions

User-defined functions can be invoked in the following format:

```yaml
shipping_cost: int{
  derive: calculate_shipping_cost(weight: 10, distance: 20)
}
```

- Function calls use the format: function(arg1, arg2, ...)
- Arguments may include primitive values, structured types, and expressions such as if(...)
- Keyword arguments are supported
    - Keyword argument order is flexible
- Functions do not cause side effects

## Pipeline Syntax for Data Processing
=======
### Type System
>>>>>>> 368c8c5 (tmp)

IntentScript supports the following types:

* Primitive types: string, int, boolean, date, etc.
* Collection types: list<T>, map<K,V>
* User-defined types: references to other entities

Use `<>` notation for generic types:
```yaml
order_items: list<Product>
product_prices: map<string, int>
```

### Property Options

Properties can use `{}` to specify options:

```yaml
name: string{required, max_length: 100}
price: int{currency: JPY}
```

### YAML Notation: Comments and Special Characters

IntentScript uses YAML comment notation:

```yaml
# This is a full-line comment
entity:
  name: string  # End-of-line comments are also possible
  # Comments can be indented to match the context
```

Comments serve as an important complementary function for gradual formality.
Even when using formally strict descriptions (Level 2), you can add natural language intent explanations through comments:

```yaml
# This is a Level 2 formal description, but comments supplement the intent in natural language
User:
  name: string{required, max_length: 50} # Required, maximum 50 characters allowed
  email: string{required, format: "email"} # Required and must be in valid format
  age: int{min: 18} # User age must be 18 or older (adult age)
```

When using YAML special characters (>, |, *, &, -, ?, :, {, } etc.) in expressions or conditions, it's recommended to enclose them in quotes when necessary.

To prioritize YAML parser compatibility or prevent syntax errors, it's safe to write the entire expression as a string literal like "items |> filter(_ > 100)":

```yaml
filtered_items: list<int>{
  derive: "items |> filter(_ > 100)"  # '>' is a special character, so enclose in quotes when needed
}
```

### YAML Notation: Literal Notation

IntentScript uses standard YAML literal notation as-is:

```yaml
# String literals
simple_string: "Hello World"
multiline_string: |
  This is a multi-line
  string.
  Indentation is preserved.

# Numeric literals
int: 28
float: 3.14

# Boolean literals
active: true
disabled: false

# Date-time literals
created_at: 2006-01-02T15:04:05Z

# List literals - block notation
colors:
  - red
  - green
  - blue

# List literals - flow notation
colors: [red, green, blue]

# Map literals - block notation
point:
  x: 10
  y: 20

# Map literals - flow notation
point: {x: 5, y: 15}
```

## Data Processing and Pipelines

IntentScript adopts pipeline syntax to enable intuitive and declarative description of processing intent.
By describing data flow and transformation operations in sequence, it achieves both structure and intent visualization.

### Pipeline Basics

Use the pipeline operator `|>` to intuitively express data processing flow:

```yaml
discount_prices: list<int>{
  derive: items
          |> filter(_.in_stock)
          |> map(_.price * (1 - _.discount_rate))
          |> filter("_ > 0")
}
```

### Standard Pipeline Functions

IntentScript provides the following basic pipeline operation functions:

* map: transform each element
  ```yaml
  # Example: double the prices
  doubled_prices: list<int>{
    derive: prices |> map("_ * 2")
  }
  ```

* filter: select elements matching a condition
  ```yaml
  # Example: select only positive numbers
  positive_numbers: list<int>{
    derive: numbers |> filter("_ > 0")
  }
  ```

* sum: calculate the total of numbers
  ```yaml
  # Example: calculate total order amount
  total_amount: int{
    derive: order_items |> map(_.price * _.quantity) |> sum
  }
  ```

* sort: arrange elements in order
  ```yaml
  # Example: sort numbers in ascending order
  sorted_values: list<int>{
    derive: values |> sort
  }
  ```

* reduce: aggregate elements into a single value
  ```yaml
  # Example: multiply numbers (cumulative multiplication)
  multiplied_number: int{
    derive: numbers |> reduce("_1 * _2", 1)
  }
  ```

### Placeholder Syntax

* Single argument: `_` for the entire argument, `_.property` for property reference
  ```yaml
  # Single argument example - extract products priced 100 yen or more
  discounted_prices: list<int>{
    derive: products
            |> filter("_.price >= 100") # Extract only products priced 100 yen or more
  }
  ```

* Multiple arguments: `_1`, `_2`, `_3`... for each argument reference
  ```yaml
  # Multiple argument example - sort numbers in descending order
  sorted_scores: list<int>{
    derive: scores
            |> sort(_2 - _1)  # Sort in descending order (_1 and _2 are the two elements being compared)
  }
  ```

### Conditional Expressions: if(...)

IntentScript supports if(condition, then_value, else_value) as a pure expression that returns one of two values depending on a condit

```yaml
final_price: int{
  derive: if(onSale, price * 0.9, price)
}
```

- `then_value` and `else_value` are ideally of the same type, but the details are left to AI interpretation
- Standard comparison and logical operators are supported

## Use of Natural Language

IntentScript allows natural language to be used anywhere in structured definitions and pipeline processing.
Based on the concept of gradual formality, formal expressions and natural language can be mixed as needed:

This functionality enables using formal expressions for parts requiring strict structure, while using natural language for detailed explanations, complex rules, and processing steps.
AI interprets these natural language expressions, understands the intent, and converts them to implementation.

### Usage in Attribute Description

Attributes can be described in natural language as shown below:

```yaml
Product:
  name: string{required}
  price: int{currency: JPY}
  description: "Detailed product description. Can be written in Markdown format."

  availability_rule: |
    Normally available for purchase if in stock.
    However, for limited items, only members can purchase.
    Pre-orders are accepted from 2 weeks before release.

  discount_strategy: |
    - 10% discount for products priced 5000 yen or more
    - Additional 5% discount during sale periods
    - Gold members always get an additional 5% discount
```

### Usage in Pipelines

Pipeline processing content can also be described in natural language:

```yaml
top_products: list<Product>{
  derive: products
          |> "exclude out-of-stock products"
          |> "sort products by sales volume"
          |> "take only the top 5"
}
```

### Natural Language Definitions

IntentScript allows natural language or comment-based definitions in addition to structured definitions.
For undefined syntax or functions in particular, it's assumed that the implementation environment (such as LLMs) will complement them.

For example, new functions can be defined using comments like this:

```yaml
# average function calculates the average of numbers
```

Such descriptions are not formal syntax, but can be utilized as flexible expressions or designs anticipating future extensions.

### Notes on Input/Output

The IntentScript specification focuses on describing intent and excludes specific input/output formats (files, APIs, standard input/output, etc.).
However, when you want to specify input/output specifications that depend on the execution environment, supplementary description through comments is recommended:

```yaml
# Input: provided in CSV format from standard input
# Output: output one line at a time to standard output
```

<<<<<<< HEAD
## About This Document

* Version: 0.0.1
* Status: Minimal specification for proof-of-concept
* Guiding principle: Emphasizes flexibility and AI-driven interpretation over exhaustive definition. Ambiguity is intentional, allowing AI and implementers to make appropriate judgments.
* Authorship: This specification was created through dialogue with generative AI.
  We express our sincere gratitude for its significant contributions to shaping this document.
* License: MIT License
=======
Such comments clarify IntentScript's intent and assist in implementation and verification.

## Future Considerations

While this specification maintains a minimal configuration, the following are issues under consideration for future expansion:

* Error handling and validation
* Practical notations such as enumerated types, dependent types, and conditional constraints
* Support for composite types (Union / Intersection types)
  * Introduction of composite types like int | null and A & B
* Introduction of generic entity definitions (e.g., User<T>)
  * Enable entity reuse through type parameters, supporting flexible construction of complex domain models
* Introduction of package configuration (namespace and scope management) for large-scale development
* Consideration of `include` functionality enabling definition reuse and split description
* Templates and other grammar for reuse
* Consistency verification with natural language comments and AI-based completion/suggestion support for ambiguous expressions

## About This Document

* Version: 0.0.1
* Status: This specification represents the minimal configuration specification for the POC (Proof of Concept) stage of IntentScript.
  It is intended for initial design verification and confirmation of the language's effectiveness, positioned as a foundation for future expansion and formal specification.
* Co-authorship: This specification was created through dialogue with generative AI. We express our gratitude for the significant contributions of generative AI in forming this specification.
* License: This specification is published under the MIT License.
>>>>>>> 368c8c5 (tmp)

This document serves as a starting point for developing IntentScript as a "language for conveying intent to AI" and is intended to provide a foundation for future discussion, implementation, and utilization.
