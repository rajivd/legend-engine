# Legend Engine Testing Framework

## Overview
Legend Engine has a comprehensive testing framework that ensures the reliability and correctness of its components. This document outlines the testing approaches, patterns, and infrastructure used in Legend Engine.

## Testing Approaches

### Unit Testing
- Tests individual components in isolation
- Uses JUnit for Java components
- Includes round-trip tests for grammar parsing and composition
- Validates error handling and edge cases

### Integration Testing
- Tests interactions between components
- Validates end-to-end functionality
- Includes database connectivity and execution tests
- Tests external format processing

### Database Integration Tests
As described in the Legend Engine documentation:

> "This document lists the integration tests run against various supported databases."

Database integration tests are categorized into:
1. Connectivity Tests - Verify connection to databases
2. SQL Generation and Execution Tests - Validate SQL generation and execution

These tests run against multiple database types:
- BigQuery
- Databricks
- MS SQL Server
- PostgreSQL
- Redshift
- Snowflake
- Spanner
- Athena

### Test Result Categories
Test results are classified into five categories:
1. Passed completely
2. Ignored (feature not supported)
3. Partially ignored (feature works but deviates from standard)
4. Failure requiring fixes
5. Unknown status due to setup issues

## Testing Infrastructure

### CI/CD Integration
- GitHub Actions workflows for automated testing
- Separate workflows for different database types
- Summary reports for test results

### Test Suites
- `Test_Relational_DbSpecific_<DbType>_UsingPureClientTestSuite` - Tests SQL generation/execution
- `ExternalIntegration_TestConnectionAcquisitionWithFlowProvider_<DbType>` - Tests connectivity

### Test Configuration
- Configuration files for test connections
- Docker-based test databases for some database types
- FINOS-hosted database instances for others

## Testing Patterns

### Grammar Testing
As described in the Pure Grammar documentation:

1. `Test...GrammarRoundtrip` - Tests both grammar parser and composer
2. `Test...GrammarParser` - Verifies parsing errors are thrown appropriately

### Service Testing
- Tests service execution
- Validates service parameters and results
- Tests service post-validations

### Feature Testing
- Tests specific features in isolation
- Validates feature behavior against specifications
- Ensures backward compatibility

## Best Practices

### Writing Tests
1. Test both positive and negative cases
2. Include edge cases and boundary conditions
3. Ensure test independence
4. Use appropriate assertions
5. Document test purpose and expectations

### Fixing Failing Tests
1. Run specific test suites to identify failures
2. Check logs for error details
3. Fix implementation or mark as unsupported if necessary
4. For connectivity issues, verify configuration
5. For SQL generation issues, fix the SQL generation code

### Handling Unsupported Features
- Mark unsupported features with messages starting with '[unsupported-api] '
- Document deviations from standards
- Gradually deprecate non-standard implementations

## Development Workflow
1. Write tests before implementation (TDD approach)
2. Run tests locally during development
3. Ensure all tests pass before submitting changes
4. Use the Pure IDE for faster development
5. Check CI/CD results for comprehensive validation
