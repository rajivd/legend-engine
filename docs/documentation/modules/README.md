# Legend Engine Multi-Module Structure

## Overview
Legend Engine is organized as a multi-module Maven project with a clear separation of concerns. This document outlines the module structure and the purpose of each module.

## Top-Level Modules

From the root `pom.xml`, Legend Engine is organized into several top-level modules:

### Core System
- `legend-engine-core`: Core functionality including Pure language processing, compilation, and execution

The core module is further divided into several key submodules:

```xml
<!-- From legend-engine-core/pom.xml -->
<modules>
    <module>legend-engine-core-base</module>
    <module>legend-engine-core-shared</module>
    <module>legend-engine-core-pure</module>
    <module>legend-engine-core-testable</module>
    <module>legend-engine-core-external-format</module>
    <module>legend-engine-core-query-pure-http-api</module>
    <module>legend-engine-core-identity</module>
</modules>
```

#### Core Base (`legend-engine-core-base`)
Contains the fundamental components for execution plan generation and execution, as well as Pure language processing:

```xml
<!-- From legend-engine-core-base/pom.xml -->
<modules>
    <module>legend-engine-core-executionPlan-execution</module>
    <module>legend-engine-core-executionPlan-generation</module>
    <module>legend-engine-core-language-pure</module>
</modules>
```

- **legend-engine-core-executionPlan-execution**: Handles the execution of plans against data sources
- **legend-engine-core-executionPlan-generation**: Generates execution plans from Pure models
- **legend-engine-core-language-pure**: Contains the Pure language parser, compiler, and protocol

#### Core Shared (`legend-engine-core-shared`)
Contains shared utilities and extension mechanisms used across the codebase.

#### Core Pure (`legend-engine-core-pure`)
Contains Pure language implementations and core extensions.

#### Core Testable (`legend-engine-core-testable`)
Provides the testing framework for Legend Engine components.

#### Core External Format (`legend-engine-core-external-format`)
Supports external data formats like JSON, XML, etc.

#### Core Query Pure HTTP API (`legend-engine-core-query-pure-http-api`)
Provides HTTP API for Pure language queries.

#### Core Identity (`legend-engine-core-identity`)
Handles identity and authentication management.

### Stores
- `legend-engine-xts-serviceStore`: Service store for integrating with REST and other services
- `legend-engine-xts-relationalStore`: Relational database store for SQL databases
- `legend-engine-xts-mongodb`: MongoDB store integration
- `legend-engine-xts-elasticsearch`: Elasticsearch store integration

### External Formats
- `legend-engine-xts-xml`: XML schema support
- `legend-engine-xts-protobuf`: Protocol Buffers support
- `legend-engine-xts-flatdata`: Flat data format support
- `legend-engine-xts-json`: JSON schema support
- `legend-engine-xts-avro`: Avro schema support
- `legend-engine-xts-arrow`: Apache Arrow support

### Languages
- `legend-engine-xts-rosetta`: Rosetta language support
- `legend-engine-xts-haskell`: Haskell integration
- `legend-engine-xts-daml`: DAML integration
- `legend-engine-xts-morphir`: Morphir integration
- `legend-engine-xts-java`: Java code generation

### Query Protocol
- `legend-engine-xts-sql`: SQL query support
- `legend-engine-xts-graphQL`: GraphQL support

### Function Activators
- `legend-engine-xts-functionActivator`: Function activation framework
- `legend-engine-xts-snowflakeApp`: Snowflake application integration
- `legend-engine-xts-bigqueryFunction`: BigQuery function integration
- `legend-engine-xts-memsqlFunction`: MemSQL function integration
- `legend-engine-xts-service`: Service framework
- `legend-engine-xts-persistence`: Persistence framework
- `legend-engine-xts-hostedService`: Hosted service support

### New Packageable Elements
- `legend-engine-xts-text`: Text element support
- `legend-engine-xts-diagram`: Diagram support
- `legend-engine-xts-data-space`: Data space support
- `legend-engine-xts-changetoken`: Change token support
- `legend-engine-xts-generation`: Generation framework
- `legend-engine-xts-dataquality`: Data quality framework

### Translator
- `legend-engine-xts-openapi`: OpenAPI integration

### Misc
- `legend-engine-xts-authentication`: Authentication framework
- `legend-engine-xts-protocol-java-generation`: Protocol Java generation
- `legend-engine-config`: Configuration framework

### Analytics
- `legend-engine-xts-analytics`: Analytics framework

### Application
- `legend-engine-application-query`: Query application
- `legend-engine-xts-identity`: Identity management
- `legend-engine-xts-ingest`: Data ingestion framework

## Module Structure Pattern

Each functional area typically follows a consistent module structure pattern:

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

## Example: Relational Store Module Structure

As described in the documentation:

1. **\<dbType\>-protocol**: Defines POJOs for connector-specific data source and authentication strategy specifications
2. **\<dbType\>-pure**: Contains DB-specific SQL generation logic in Pure language
3. **\<dbType\>-grammar**: Contains ANTLR-based code for bi-directional conversion between textual DSLs and POJOs
4. **\<dbType\>-execution**: Defines driver and authentication logic
5. **\<dbType\>-execution-tests**: Contains integration tests for connectivity and execution

## Dependencies

Legend Engine has dependencies on:
- `legend.pure.version`: Pure language core
- `legend.shared.version`: Shared utilities
- Various database drivers, libraries, and frameworks

## Dependencies Between Modules

The Legend Engine modules follow a hierarchical dependency structure:

1. **Core Dependencies**:
   - Most modules depend on `legend-engine-core-shared` for common utilities
   - Modules that need Pure language processing depend on `legend-engine-core-language-pure`
   - Modules that need execution capabilities depend on `legend-engine-core-executionPlan-execution`

2. **Extension Dependencies**:
   - Store extensions (relational, MongoDB, etc.) depend on core execution modules
   - Language extensions depend on core language modules
   - Format extensions depend on core protocol modules

3. **Cross-Module Dependencies**:
   - Some modules have cross-dependencies, particularly for testing
   - The dependency graph is managed through Maven to avoid circular dependencies

## Navigating the Codebase

To effectively navigate the Legend Engine codebase:

1. **Start with the Core**:
   - Begin with `legend-engine-core` to understand the fundamental components
   - Examine the Pure language modules to understand the domain-specific language
   - Look at the execution plan modules to understand data processing

2. **Follow the Extension Pattern**:
   - Each extension follows a similar pattern (protocol, pure, grammar, compiler, execution)
   - Understanding one extension makes it easier to understand others

3. **Use Maven Structure**:
   - The Maven module hierarchy reflects the logical organization
   - Use `find . -name "pom.xml" | grep -v target` to identify modules
   - Examine parent-child relationships in pom.xml files

4. **Look for Patterns**:
   - Extension interfaces in `legend-engine-shared-extensions`
   - Extension loaders using ServiceLoader
   - Test classes following naming conventions like `*Test.java` or `Test*.java`

5. **Key Entry Points**:
   - `Server.java` - Main server entry point
   - `PlanExecutor.java` - Execution entry point
   - `CompilerExtension.java` - Compiler extension point
   - `ServicePlanGenerator.java` - Service plan generation

## Build System

Legend Engine uses Maven for build management with:
- Java 11 as the target JDK
- Extensive plugin configuration for testing, packaging, and deployment
- Support for parallel builds to improve performance
