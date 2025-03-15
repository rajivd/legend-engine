# Legend Engine Documentation - Table of Contents

## Overview

This document provides a comprehensive table of contents for the Legend Engine documentation. It is organized into logical sections to help you navigate the documentation and find the information you need.

## Documentation Sections

### 1. Architecture and Core Components

- [Architecture Overview](../architecture/README.md) - High-level architecture of Legend Engine
  - Core components and high-level architecture
  - Extension mechanisms overview
  - Execution model and data flow

- [Module Structure](../modules/README.md) - Organization of the Legend Engine codebase
  - Core system modules
  - Store modules
  - Extension modules
  - Service modules
  - Build system and dependencies

- [Pure Language](../pure-language/README.md) - The domain-specific language used in Legend
  - Pure language philosophy
  - Grammar and protocol model
  - Compilation process
  - Extension mechanisms
  - Pure IDE

### 2. Extension Mechanisms

- [Extension Mechanisms](../extensions/README.md) - Detailed explanation of extension points
  - LegendExtension interface
  - Extension discovery
  - Extension loaders
  - Extension registry
  - Extension configuration

- [Extension Development Patterns](../extension-development/README.md) - Patterns for implementing extensions
  - Extension lifecycle
  - Extension registration
  - Extension dependencies
  - Extension versioning
  - Extension testing

### 3. Implementation Guidelines

- [Implementation Guidelines](../implementation-guidelines/README.md) - Best practices for implementing features
  - Code organization
  - Multi-pass compilation
  - Error handling and validation
  - Performance considerations
  - Testing strategies
  - Documentation requirements

- [Service Implementation](../service-implementation/README.md) - Implementing services in Legend Engine
  - Service execution models
  - Plan generation
  - Parameter handling
  - Error handling
  - Testing services

- [Database Connector Implementation](../database-connector-implementation/README.md) - Implementing database connectors
  - Database connector components
  - Module structure
  - Implementation process
  - Testing framework
  - Extension points
  - Best practices

### 4. Testing Framework

- [Testing Framework](../testing/README.md) - Testing approaches and infrastructure
  - Test types
  - Test execution
  - Test assertions
  - Test infrastructure
  - CI/CD integration
  - Best practices for testing

## Index of Key Concepts

### A
- **Architecture**
  - [Architecture Overview](../architecture/README.md)
  - [Module Structure](../modules/README.md)

- **Authentication**
  - [Database Connector Implementation - Authentication](../database-connector-implementation/README.md#implementation-process)
  - [Service Implementation - Authentication](../service-implementation/README.md)

### B
- **Best Practices**
  - [Implementation Guidelines](../implementation-guidelines/README.md)
  - [Database Connector Implementation - Best Practices](../database-connector-implementation/README.md#best-practices)
  - [Testing - Best Practices](../testing/README.md#best-practices)

### C
- **Compilation**
  - [Pure Language - Compilation Process](../pure-language/README.md#compilation-process)
  - [Implementation Guidelines - Multi-Pass Compilation](../implementation-guidelines/README.md#multi-pass-compilation)

- **Compiler Extensions**
  - [Extension Mechanisms - CompilerExtension](../extensions/README.md#compilerextension)
  - [Extension Development Patterns - Compiler Extensions](../extension-development/README.md)

- **Connection Management**
  - [Database Connector Implementation - Connection Management](../database-connector-implementation/README.md#connection-management)

### D
- **Database Connectors**
  - [Database Connector Implementation](../database-connector-implementation/README.md)
  - [Module Structure - Stores](../modules/README.md#stores)

- **Dependencies**
  - [Module Structure - Dependencies Between Modules](../modules/README.md#dependencies-between-modules)
  - [Extension Development Patterns - Extension Dependencies](../extension-development/README.md)

### E
- **Error Handling**
  - [Implementation Guidelines - Error Handling and Validation](../implementation-guidelines/README.md#error-handling-and-validation)
  - [Service Implementation - Error Handling](../service-implementation/README.md#error-handling)

- **Execution**
  - [Service Implementation - Execution Models](../service-implementation/README.md#service-execution-models)
  - [Database Connector Implementation - Execution Logic](../database-connector-implementation/README.md#implementation-process)

- **Extensions**
  - [Extension Mechanisms](../extensions/README.md)
  - [Extension Development Patterns](../extension-development/README.md)
  - [Architecture - Extension Mechanisms](../architecture/README.md#extension-mechanisms)

### G
- **Grammar**
  - [Pure Language - Grammar](../pure-language/README.md#pure-grammar)
  - [Database Connector Implementation - Grammar Support](../database-connector-implementation/README.md#implementation-process)

### I
- **Implementation Guidelines**
  - [Implementation Guidelines](../implementation-guidelines/README.md)
  - [Service Implementation](../service-implementation/README.md)
  - [Database Connector Implementation](../database-connector-implementation/README.md)

- **Integration Testing**
  - [Testing Framework - Integration Testing](../testing/README.md#integration-testing)
  - [Database Connector Implementation - Integration Testing](../database-connector-implementation/README.md#testing-framework)

### M
- **Module Structure**
  - [Module Structure](../modules/README.md)
  - [Database Connector Implementation - Module Structure](../database-connector-implementation/README.md#module-structure)

- **Multi-Pass Compilation**
  - [Implementation Guidelines - Multi-Pass Compilation](../implementation-guidelines/README.md#multi-pass-compilation)
  - [Pure Language - Compilation Process](../pure-language/README.md#compilation-process)

### P
- **Parameters**
  - [Service Implementation - Parameter Handling](../service-implementation/README.md#parameter-handling)

- **Performance**
  - [Implementation Guidelines - Performance Considerations](../implementation-guidelines/README.md#performance-considerations)

- **Plan Generation**
  - [Service Implementation - Plan Generation](../service-implementation/README.md#plan-generation)

- **Protocol Model**
  - [Pure Language - Protocol Model](../pure-language/README.md#pure-grammar)
  - [Database Connector Implementation - Protocol Classes](../database-connector-implementation/README.md#implementation-process)

- **Pure Language**
  - [Pure Language](../pure-language/README.md)
  - [Architecture - Pure Language](../architecture/README.md)

### S
- **Service Implementation**
  - [Service Implementation](../service-implementation/README.md)
  - [Module Structure - Services](../modules/README.md#services)

- **SQL Generation**
  - [Database Connector Implementation - SQL Generation](../database-connector-implementation/README.md#implementation-process)
  - [Database Connector Implementation - Best Practices](../database-connector-implementation/README.md#best-practices)

### T
- **Testing**
  - [Testing Framework](../testing/README.md)
  - [Implementation Guidelines - Testing Strategies](../implementation-guidelines/README.md#testing-strategies)
  - [Service Implementation - Testing](../service-implementation/README.md#testing)
  - [Database Connector Implementation - Testing](../database-connector-implementation/README.md#testing-framework)

- **Test Types**
  - [Testing Framework - Test Types](../testing/README.md#test-types-and-patterns)
  - [Testing Framework - Test Categories](../testing/README.md#test-categories)

### V
- **Validation**
  - [Implementation Guidelines - Validation Strategies](../implementation-guidelines/README.md#validation-strategies)
  - [Service Implementation - Validation](../service-implementation/README.md#parameter-validation)

## How to Use This Documentation

1. **New to Legend Engine?** Start with the [Architecture Overview](../architecture/README.md) to understand the high-level architecture and core components.

2. **Want to implement a new feature?** Read the [Implementation Guidelines](../implementation-guidelines/README.md) for best practices and patterns.

3. **Need to extend Legend Engine?** Check out the [Extension Mechanisms](../extensions/README.md) and [Extension Development Patterns](../extension-development/README.md).

4. **Implementing a specific component?** Go directly to the relevant implementation guide:
   - [Service Implementation](../service-implementation/README.md)
   - [Database Connector Implementation](../database-connector-implementation/README.md)

5. **Need to test your implementation?** Refer to the [Testing Framework](../testing/README.md) for testing approaches and best practices.

6. **Looking for a specific concept?** Use the [Index of Key Concepts](#index-of-key-concepts) to find relevant sections.
