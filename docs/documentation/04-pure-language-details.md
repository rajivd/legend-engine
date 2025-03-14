# Pure Language Details

## Overview
Pure is a domain-specific language (DSL) used in Legend Engine for metamodeling and code generation. It provides a declarative way to define models, mappings, and transformations with strong typing and functional programming paradigms.

## Key Characteristics

### Functional Programming Language
- Pure is a functional programming language with strong typing
- It supports higher-order functions, lambdas, and immutability
- Functions are first-class citizens in Pure

### Metamodeling Capabilities
- Pure allows defining metamodels (models of models)
- Supports class definitions, properties, and inheritance
- Enables complex type hierarchies and relationships

### Grammar and Parsing
As described in the Legend Engine documentation:

> "This module includes logic for parsing PURE grammar to protocol JSON as well as transforming protocol to Pure."

The Pure language is parsed using ANTLR with a focus on:
- Bijective relationship between grammar and protocol model
- Simple parsing that builds protocol models without context-aware inference
- Clear separation between parsing and compilation concerns

## Pure Language Components

### Class Definitions
```
Class my::domain::Person
{
  firstName: String[1];
  lastName: String[1];
  dateOfBirth: Date[0..1];
}
```

### Enumerations
```
Enum my::domain::Gender
{
  MALE,
  FEMALE,
  NON_BINARY
}
```

### Functions
```
function my::domain::getFullName(person: my::domain::Person[1]): String[1]
{
  $person.firstName + ' ' + $person.lastName
}
```

### Mappings
```
Mapping my::domain::PersonMapping
(
  my::domain::Person: Pure
  {
    ~src my::source::PersonSource
    firstName: $src.FIRST_NAME,
    lastName: $src.LAST_NAME,
    dateOfBirth: $src.DOB
  }
)
```

## Compilation Process

The Pure language goes through a multi-pass compilation process:

1. **Parsing**: Text is parsed into an abstract syntax tree using ANTLR
2. **Protocol Model Creation**: AST is transformed into protocol model objects
3. **First Pass Compilation**: Basic structure and references are resolved
4. **Second Pass Compilation**: Cross-references and dependencies are resolved
5. **Third Pass Compilation**: Final validation and optimization

## Extension Mechanisms

Pure language can be extended through:

- **CompilerExtension**: Adds custom compilation logic
- **LegendLanguageExtension**: Extends language capabilities
- **Custom DSLs**: Domain-specific languages built on top of Pure

## Pure IDE

Legend Engine provides a Pure IDE for development:
- Available at http://127.0.0.1:9200/ide when running locally
- Supports debugging with breakpoints using `meta::pure::ide::debug()`
- Provides a terminal for debugging actions

## Debugging Pure Code

As described in the Legend Engine documentation:

1. Use `meta::pure::ide::debug()` to create breakpoints
2. Execute with F9, which pauses at breakpoints
3. A summary is printed with the current stack and accessible variables
4. Debug commands available:
   - `debug` or `debug summary`: Print the debugging summary
   - `debug <pure expression>`: Evaluate an expression
   - `debug abort`: Stop the current execution

## Best Practices

When working with Pure:

1. Follow functional programming principles
2. Leverage the type system for safety
3. Use mappings for data transformations
4. Create reusable functions for common operations
5. Organize code in logical packages
6. Use the Pure IDE for development and debugging
