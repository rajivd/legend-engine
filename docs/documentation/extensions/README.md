# Legend Engine Extension Mechanisms

## Overview
Legend Engine employs a plugin architecture that allows for extending its functionality without modifying the core codebase. This document outlines the extension mechanisms available in Legend Engine and how to implement them.

## Core Extension Interfaces

### LegendExtension
The base interface for all extensions in the Legend Engine ecosystem. It provides:
- Group identification for categorizing extensions
- Type information for extension discovery
- Type grouping for hierarchical organization

```java
public interface LegendExtension
{
    default MutableList<String> group()
    {
        return Lists.mutable.empty();
    }

    default String type()
    {
        return "Unknown Type " + this.getClass().getName();
    }

    default MutableList<String> typeGroup()
    {
        return Lists.mutable.empty();
    }
}
```

### LegendLanguageExtension
Extends language capabilities with:
- Custom grammar parsing
- Serialization/deserialization
- Compiler extensions for language constructs

```java
public interface LegendLanguageExtension extends LegendExtension
{
    @Override
    default MutableList<String> typeGroup()
    {
        return Lists.mutable.with("Lang");
    }
}
```

### LegendPlanExtension
Extends plan generation and execution capabilities:
- Adds custom plan transformers
- Provides execution plan optimization
- Enables platform-specific code generation

```java
public interface LegendPlanExtension extends LegendExtension
{
    @Override
    default MutableList<String> typeGroup()
    {
        return Lists.mutable.with("Plan");
    }
}
```

### LegendConnectionExtension
Extends connection management capabilities:
- Adds support for new connection types
- Provides authentication mechanisms
- Enables connection pooling and management

### LegendExternalFormatExtension
Extends support for external data formats:
- Adds schema parsing and validation
- Provides serialization/deserialization
- Enables format-specific optimizations

### CompilerExtension
Extends the Pure language compiler with custom processing logic:
- Adds processors for different element types
- Provides custom compilation steps
- Supports multi-pass compilation (first, second, third passes)
- Enables extension of value specifications, class mappings, and test assertions

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

Extension loaders are responsible for discovering and loading extensions using Java's ServiceLoader mechanism. They typically follow a common pattern:

```java
public class DeploymentExtensionLoader
{
    public static List<DeploymentExtension> extensions()
    {
        List<DeploymentExtension> extensions = Lists.mutable.withAll(ServiceLoader.load(DeploymentExtension.class));
        Set<String> extensionKeys = Sets.mutable.empty();
        for (DeploymentExtension extension : extensions)
        {
            if (!extensionKeys.add(extension.getKey()))
            {
                String extensionsWithSameKey = ListIterate.collect(extensions.stream()
                    .filter(e -> e.getKey().equals(extension.getKey()))
                    .collect(Collectors.toList()), e -> e.getClass().getName())
                    .makeString(",");
                throw new EngineException("Deployment extension keys must be unique. Found duplicate key: '" 
                    + extension.getKey() + "' on extensions: " + extensionsWithSameKey);
            }
        }
        return extensions;
    }
}
```

Common extension loaders include:
- `CompilerExtensionLoader` - Loads extensions for the Pure compiler
- `DeploymentExtensionLoader` - Loads extensions for deployment models
- `ArtifactGenerationExtensionLoader` - Loads extensions for artifact generation
- `PureGrammarParserExtensionLoader` - Loads extensions for grammar parsing
- `PureGrammarComposerExtensionLoader` - Loads extensions for grammar composition
- `ContentPatternParserExtensionLoader` - Loads extensions for content pattern parsing
- `ExternalFormatExtensionLoader` - Loads extensions for external formats
- `ExecutionPlanJavaCompilerExtensionLoader` - Loads extensions for Java compilation of execution plans

Extension loaders typically provide:
1. A method to load and cache extensions
2. Validation of extension uniqueness
3. Helper methods to access extension capabilities
4. Methods to aggregate extension functionality

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
