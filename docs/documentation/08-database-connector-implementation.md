# Database Connector Implementation

## Overview
Legend Engine supports various database connectors through its extensible architecture. This document outlines the process and patterns for implementing database connectors in Legend Engine.

## Database Connector Components

As described in the Legend Engine documentation, developing a new relational connector has 5 main parts:

1. **DB-Specific SQL Generation Logic**: Generate SQL queries tailored to the specific database dialect
2. **Connector-Specific Data Source and Authentication Specifications**: Model the connection parameters and authentication strategies
3. **Driver Logic and Authentication Logic**: Implement the connection and query execution
4. **Integration Testing**: Test SQL generation and execution against a test database instance
5. **Connection Acquisition Testing**: Test connection establishment with the database

## Module Structure

A relational connector follows a specific module structure:

1. **\<dbType\>-protocol**: Defines POJOs representing connector-specific data source and authentication strategy specifications
2. **\<dbType\>-pure**: Contains DB-specific SQL generation logic in Pure language and specifies PURE to JSON conversion
3. **\<dbType\>-grammar**: Contains ANTLR-based code for bi-directional conversion between textual DSLs and POJOs
4. **\<dbType\>-execution**: Defines driver and authentication logic
5. **\<dbType\>-execution-tests**: Contains integration tests for connectivity and execution

## Implementation Process

### 1. Define Protocol Classes
- Create data source specification classes
- Define authentication strategy classes
- Implement serialization/deserialization

### 2. Implement SQL Generation Logic
- Create Pure language functions for SQL generation
- Handle database-specific SQL syntax
- Implement query optimization strategies

### 3. Develop Grammar Support
- Create ANTLR grammar for parsing connection strings
- Implement bi-directional conversion between text and objects
- Handle error reporting and validation

### 4. Implement Execution Logic
- Create connection manager
- Implement authentication strategies
- Develop query execution and result processing

### 5. Write Integration Tests
- Test SQL generation against expected output
- Validate query execution with test data
- Verify connection establishment and authentication

## Testing Framework

Legend Engine provides a comprehensive testing framework for database connectors:

### Connectivity Tests
- `ExternalIntegration_TestConnectionAcquisitionWithFlowProvider_<DbType>`: Tests connection establishment

### SQL Generation and Execution Tests
- `Test_Relational_DbSpecific_<DbType>_UsingPureClientTestSuite`: Tests SQL generation and execution

### Test Categories
Test results are classified into five categories:
1. Passed completely
2. Ignored (feature not supported)
3. Partially ignored (feature works but deviates from standard)
4. Failure requiring fixes
5. Unknown status due to setup issues

## Extension Points

### RelationalDatabaseConnectionExtension
Extends connection capabilities for relational databases:
- Handles connection establishment
- Manages authentication
- Provides database metadata

### RelationalDatabaseCompilerExtension
Extends compiler capabilities for relational databases:
- Processes database-specific elements
- Validates database configuration
- Handles database-specific features

### RelationalDatabaseExecutionExtension
Extends execution capabilities for relational databases:
- Executes SQL queries
- Processes query results
- Handles database-specific execution features

## Best Practices

### SQL Generation
1. Follow the database's SQL dialect specifications
2. Handle edge cases and special syntax
3. Optimize queries for performance
4. Provide clear error messages for unsupported features
5. Document SQL generation limitations

### Connection Management
1. Implement connection pooling for performance
2. Handle connection timeouts and retries
3. Secure sensitive connection information
4. Validate connection parameters
5. Provide detailed error messages for connection failures

### Testing
1. Test with realistic database schemas
2. Include edge cases and complex queries
3. Validate error handling
4. Test with different authentication methods
5. Verify performance with large datasets

### Handling Unsupported Features
- Mark unsupported features with messages starting with '[unsupported-api] '
- Document deviations from standards
- Gradually deprecate non-standard implementations

## Example Implementation

For a step-by-step guide on implementing a new database connector, refer to the [new connector tutorial](../new-connector-tutorial.md) in the Legend Engine documentation.
