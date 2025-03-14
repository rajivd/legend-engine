# Legend Engine Extension Mechanisms

## Overview
Legend Engine employs a plugin architecture that allows for extending its functionality without modifying the core codebase. This document outlines the extension mechanisms available in Legend Engine and how to implement them.

## Core Extension Interfaces

### LegendExtension
The base interface for all extensions in the Legend Engine ecosystem. It provides:
- Group identification for categorizing extensions
- Type information for extension discovery
- Type grouping for hierarchical organization

### CompilerExtension
Extends the Pure language compiler with custom processing logic:
- Adds processors for different element types
- Provides custom compilation steps
- Supports multi-pass compilation (first, second, third passes)
- Enables extension of value specifications, class mappings, and test assertions

### LegendLanguageExtension
Extends language capabilities with:
- Custom grammar parsing
- Serialization/deserialization
- Compiler extensions for language constructs

## Extension Discovery

Legend Engine uses Java's ServiceLoader pattern to discover extensions at runtime:
- Extensions are registered in META-INF/services
- Extension loaders scan the classpath for implementations
- Registry classes maintain collections of available extensions

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

## Implementing Extensions

To implement a new extension:

1. Identify the appropriate extension point
2. Implement the relevant interface
3. Register the extension with ServiceLoader
4. Provide necessary supporting classes

### Example: Implementing a CompilerExtension

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

### Extension Registration

Register your extension in `META-INF/services/org.finos.legend.engine.language.pure.compiler.toPureGraph.extension.CompilerExtension`:

```
com.example.MyCompilerExtension
```

## Extension Loaders

Extension loaders are responsible for discovering and loading extensions:
- `DeploymentExtensionLoader`
- `ArtifactGenerationExtensionLoader`
- `ContentPatternParserExtensionLoader`
- And many others specific to different extension types

## Extension Registry

Extension registries maintain collections of loaded extensions and provide access to them:
- Validate extension uniqueness
- Group extensions by type
- Provide lookup capabilities

## Best Practices

When implementing extensions:
1. Follow the single responsibility principle
2. Ensure proper error handling and validation
3. Provide comprehensive testing
4. Document extension points and behavior
5. Follow existing patterns in the codebase
