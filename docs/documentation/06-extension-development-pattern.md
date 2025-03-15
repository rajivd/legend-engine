# Extension Development Pattern

## Overview
Legend Engine is designed with extensibility as a core principle. This document outlines the patterns and best practices for developing extensions to the Legend Engine.

## Extension Types

### Store Extensions
Add support for new data stores:
- Database connectors
- NoSQL stores
- Service integrations
- Custom data sources

### External Format Extensions
Add support for data format schemas:
- JSON Schema
- XSD
- Avro
- Protobuf
- Custom formats

### Service Extensions
Extend service capabilities:
- Service execution
- Service testing
- Service validation
- Service generation

### Generation Extensions
Add code generation capabilities:
- Language-specific code generation
- Documentation generation
- Artifact generation

## Extension Development Process

### 1. Identify Extension Point
- Determine which aspect of Legend Engine to extend
- Identify the appropriate extension interface
- Understand the extension's responsibilities

### 2. Implement Extension Interface
- Implement the required interface methods
- Follow the extension pattern for the specific type
- Ensure proper error handling and validation

### 3. Register Extension
- Register the extension with ServiceLoader
- Create META-INF/services entries
- Ensure unique extension identifiers

### 4. Test Extension
- Write comprehensive tests
- Validate functionality
- Ensure compatibility with existing features

### 5. Document Extension
- Document extension capabilities
- Provide usage examples
- Include configuration details

## Module Structure Pattern

Extensions typically follow a consistent module structure pattern:

### Protocol
- Defines data models and interfaces
- Contains POJOs representing domain concepts
- Example: `<module>-protocol`

### Pure
- Contains Pure language implementations
- Defines metamodels and transformations
- Example: `<module>-pure`

### Grammar
- Provides parsing and generation of textual DSLs
- Converts between text and object representations
- Example: `<module>-grammar`

### Compiler
- Implements compilation logic
- Transforms protocol objects to Pure model
- Example: `<module>-compiler`

### Execution
- Implements runtime execution
- Handles data processing and transformation
- Example: `<module>-execution`

### Tests
- Contains unit and integration tests
- Validates functionality
- Example: `<module>-tests`

## Example: Relational Store Extension

As described in the Legend Engine documentation:

1. **\<dbType\>-protocol**: Defines POJOs for connector-specific data source and authentication strategy specifications
2. **\<dbType\>-pure**: Contains DB-specific SQL generation logic in Pure language
3. **\<dbType\>-grammar**: Contains ANTLR-based code for bi-directional conversion between textual DSLs and POJOs
4. **\<dbType\>-execution**: Defines driver and authentication logic
5. **\<dbType\>-execution-tests**: Contains integration tests for connectivity and execution

## Implementation Patterns

### CompilerExtension Implementation

```java
public class MyCompilerExtension implements CompilerExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("MyExtension");
    }

    @Override
    public Iterable<? extends Processor<?>> getExtraProcessors()
    {
        return Lists.immutable.with(
            Processor.newProcessor(
                MyElement.class,
                (element, context) -> processFirstPass(element, context),
                (element, context) -> processSecondPass(element, context),
                (element, context) -> processThirdPass(element, context)
            )
        );
    }
    
    // Additional implementation methods
}
```

### Extension Loader Implementation

```java
public class MyExtensionLoader
{
    public static List<MyExtension> extensions()
    {
        List<MyExtension> extensions = Lists.mutable.withAll(ServiceLoader.load(MyExtension.class));
        // Validate extensions
        return extensions;
    }
}
```

## Best Practices

### Design Principles
1. Follow the single responsibility principle
2. Ensure proper error handling and validation
3. Maintain backward compatibility
4. Follow existing patterns in the codebase
5. Use functional interfaces for flexibility

### Implementation Guidelines
1. Use immutable data structures where possible
2. Leverage existing utilities and helpers
3. Provide comprehensive error messages
4. Document public APIs
5. Include source information for error reporting

### Testing Guidelines
1. Write unit tests for individual components
2. Include integration tests for end-to-end functionality
3. Test both positive and negative cases
4. Validate error handling
5. Ensure compatibility with existing features
