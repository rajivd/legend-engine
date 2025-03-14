# Legend Engine Architecture Overview

## Introduction
Legend Engine is a data transformation and execution engine developed by FINOS. It provides the core execution capabilities for the Legend platform, enabling data transformation, model execution, and integration with various data sources. It's important to note that Legend Engine is specifically designed for data transformation and not a machine learning or AI project.

## Core Components

### Pure Parser and Compiler
- Parses and compiles code written in the Pure language
- Transforms Pure language constructs into executable plans
- Supports extensible compilation through compiler extensions

### Execution Engine
- Generates and executes execution plans when provided with:
  - A Pure function
  - A Mapping
  - A Runtime
- Handles data transformation across different data sources
- Supports various execution modes and optimization strategies

### Model Transformation
- Enables transformation between different data models
- Supports model-to-model mappings
- Provides access points for model transformers written in Pure

### Extension Mechanisms
- Plugin architecture for extending functionality
- Extension points for adding new data sources, formats, and capabilities
- Service loader pattern for discovering extensions

## High-Level Architecture

Legend Engine follows a modular architecture with clear separation of concerns:

1. **Core Engine** - Fundamental components for parsing, compiling, and executing Pure code
2. **Extensions** - Pluggable components for specific functionality (stores, formats, etc.)
3. **Protocol** - Data models and interfaces for communication
4. **Grammar** - Language parsing and generation
5. **Execution** - Runtime execution of compiled plans

## Key Concepts

### Pure Language
Pure is a domain-specific language for metamodeling and code generation. It provides:
- Strong typing system
- Functional programming paradigm
- Declarative syntax for defining models, mappings, and transformations

### Stores
Stores represent physical data sources like:
- Relational databases
- MongoDB
- Service endpoints
- Elasticsearch

### External Formats
External formats define schemas for data interchange:
- JSON Schema
- XSD
- Avro
- Protobuf

### Services
Services expose functionality through well-defined interfaces:
- REST endpoints
- GraphQL APIs
- Function execution

## Key Interfaces and Their Relationships

Legend Engine is built around a set of core interfaces that define its extensibility model:

### Extension Interfaces
- **LegendExtension** - Base interface for all extensions with methods for grouping and typing
- **LegendLanguageExtension** - Extensions related to language processing
- **LegendPlanExtension** - Extensions related to execution plan generation and transformation
- **LegendConnectionExtension** - Extensions for connection management
- **LegendExternalFormatExtension** - Extensions for external data formats
- **LegendGenerationExtension** - Extensions for code generation

### Execution Interfaces
- **StoreExecutor** - Interface for executing plans against specific data stores
- **PlanExecutor** - Orchestrates execution of plans across different stores
- **ConnectionFactoryExtension** - Creates connections to data sources
- **ServiceExecutionExtension** - Handles service-specific execution logic

### Compiler Interfaces
- **CompilerExtension** - Extends the Pure compiler with custom logic
- **Processor** - Processes specific elements during compilation

These interfaces form a hierarchical structure that allows for modular extension of the engine's capabilities.

## Data Flow Through the System

The data flow in Legend Engine follows these general steps:

1. **Input Processing**:
   - Pure code is parsed into an abstract syntax tree (AST)
   - The AST is transformed into protocol models

2. **Compilation**:
   - Protocol models are compiled into Pure model objects
   - The compilation process involves multiple passes:
     - First pass: Creates Pure (M3) objects
     - Second pass: Resolves references
     - Third pass: Resolves cross-dependencies

3. **Plan Generation**:
   - Pure model objects are transformed into execution plans
   - Plans are optimized using transformers
   - Platform-specific code (e.g., Java) is generated

4. **Execution**:
   - Plans are executed against data sources
   - Results are transformed according to the defined model
   - Data is returned in the requested format

5. **Extension Points**:
   - Extensions can hook into various stages of this flow
   - ServiceLoader is used to discover extensions dynamically

## Execution Flow

1. Parse Pure code into an abstract syntax tree
2. Compile the AST into an execution plan
3. Execute the plan against specified data sources
4. Transform and return results according to the defined model

## Deployment Options

Legend Engine can be deployed as:
- A standalone server
- Embedded within other applications
- Part of the broader Legend platform

## Development Setup

Legend Engine requires:
- Maven 3.6+
- JDK 11
- For local development, configuration files are provided in the repository
