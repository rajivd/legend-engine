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

### Bijective Relationship Philosophy

The relationship between the grammar and the protocol model is designed to be **bijective**. As stated in the Pure Grammar README:

> "Ideally, we want the relation between the grammar and the protocol model to be __bijective__. This is needed to ensure that the grammar parser and transformer are symmetrical. As such, the round-trip tests are designed to enforce this."

This bijective relationship ensures that:
1. Any valid Pure grammar can be parsed into a protocol model
2. Any protocol model can be transformed back into Pure grammar
3. The round-trip transformation preserves semantic equivalence

### Parser Simplicity

The parser is intentionally designed to be simple:

> "The parser's job is solely to build the protocol models. It should not be 'smart' enough to do any context-aware inference - this will be handled by the compiler."

This separation of concerns ensures that:
- The parser only throws parser errors, not compilation errors
- Complex validation and inference are handled by the compiler
- The parser focuses on structure, not semantics

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

The Pure language goes through a multi-pass compilation process implemented in the `PureModel` class:

1. **Parsing**: Text is parsed into an abstract syntax tree using ANTLR
2. **Protocol Model Creation**: AST is transformed into protocol model objects
3. **First Pass Compilation**: Basic structure and references are resolved
   - Creates Pure (M3) objects
   - Registers elements in the model
   - Processes independent elements first (e.g., profiles)
4. **Second Pass Compilation**: Cross-references and dependencies are resolved
   - Resolves references between elements
   - Processes elements in dependency order
   - Validates structural integrity
5. **Third Pass Compilation**: Final validation and optimization
   - Performs semantic validation
   - Resolves complex dependencies
   - Prepares for execution plan generation

### Dependency Management

The compilation process uses a sophisticated dependency management system:
- Elements are grouped by their Java class type
- A dependency graph is constructed between element types
- Elements are processed in topological order of their dependencies
- Circular dependencies are detected and reported as errors

### Extension-Based Compilation

The compiler uses a flexible extension mechanism to process different types of elements:

```java
// From PureModel.java
this.extensions.getExtraProcessors().forEach(x -> 
    dependencyGraph.put(x.getElementClass(), 
        (Collection<java.lang.Class<?>>) x.getPrerequisiteClasses()));
```

Each processor implements the `Processor` interface:

```java
public interface Processor<T>
{
    void process(T element, CompileContext context);
    
    // Multi-pass processing methods
    void processFirstPass(T element, CompileContext context);
    void processSecondPass(T element, CompileContext context);
    void processThirdPass(T element, CompileContext context);
}
```

This extension-based approach allows:
- Modular processing of different element types
- Custom compilation logic for extensions
- Consistent multi-pass compilation across all elements

## Extension Mechanisms

Pure language can be extended through:

- **CompilerExtension**: Adds custom compilation logic
- **LegendLanguageExtension**: Extends language capabilities
- **LegendPlanExtension**: Extends plan generation and execution
- **Custom DSLs**: Domain-specific languages built on top of Pure

### CompilerExtension

The `CompilerExtension` interface allows extending the Pure compiler with custom processing logic:

```java
public interface CompilerExtension
{
    default MutableList<String> group()
    {
        return Lists.mutable.empty();
    }
    
    default Iterable<? extends Processor<?>> getExtraProcessors()
    {
        return Lists.immutable.empty();
    }
    
    // Additional methods for specific element types
}
```

CompilerExtensions are discovered using Java's ServiceLoader pattern and provide:
- Custom processors for different element types
- Multi-pass compilation support
- Extension-specific validation logic

### From Pure to Execution Plans

After compilation, Pure code is transformed into executable plans through:

1. **Plan Generation**: Pure models are transformed into execution plans
   - `PlanGenerator` creates platform-agnostic execution plans
   - Plans represent the operations to be performed on data
   - Plans are optimized using transformers

2. **Plan Transformation**: Plans are optimized and transformed
   - `PlanTransformer` applies optimizations and transformations
   - Database-specific optimizations are applied
   - Cost-based optimizations improve performance

3. **Code Generation**: Platform-specific code is generated
   - Java code is generated for execution
   - SQL queries are generated for database operations
   - Other target platforms can be supported through extensions

4. **Plan Execution**: Plans are executed against data sources
   - `PlanExecutor` orchestrates execution across different stores
   - `StoreExecutor` implementations handle store-specific execution
   - Results are transformed according to the defined model

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
