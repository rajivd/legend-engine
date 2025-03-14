# Legend Engine Multi-Module Structure

## Overview
Legend Engine is organized as a multi-module Maven project with a clear separation of concerns. This document outlines the module structure and the purpose of each module.

## Top-Level Modules

From the root `pom.xml`, Legend Engine is organized into several top-level modules:

### Core System
- `legend-engine-core`: Core functionality including Pure language processing, compilation, and execution

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

## Build System

Legend Engine uses Maven for build management with:
- Java 11 as the target JDK
- Extensive plugin configuration for testing, packaging, and deployment
- Support for parallel builds to improve performance
