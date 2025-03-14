# Legend Engine Architecture Overview

## Introduction
Legend Engine is a data transformation and execution engine developed by FINOS. It provides the core execution capabilities for the Legend platform, enabling data transformation, model execution, and integration with various data sources.

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
