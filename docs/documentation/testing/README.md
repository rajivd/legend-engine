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

### TestableRunner Architecture

The core of the Legend Engine testing framework is the `TestableRunner` class, which provides a unified way to execute tests for different types of testable elements:

```java
public class TestableRunner
{
    public RunTestsResult doTests(List<RunTestsTestableInput> runTestsTestableInputs, PureModel pureModel, PureModelContextData data)
    {
        RunTestsResult runTestsResult = new RunTestsResult();
        for (RunTestsTestableInput testableInput : runTestsTestableInputs)
        {
            // Get the testable element
            Testable testable = (Testable) packageableElement;
            
            // Find the appropriate test runner for this testable
            TestRunner testRunner = TestableRunnerExtensionLoader.forTestable(testable);
            
            // Execute atomic tests and test suites
            for (Test test : testable._tests())
            {
                // Execute atomic tests
                if ((test instanceof Root_meta_pure_test_AtomicTest) && 
                    (testIds.isEmpty() || testIdStrings.contains(test._id())))
                {
                    runTestsResult.results.add(
                        testRunner.executeAtomicTest((Root_meta_pure_test_AtomicTest) test, pureModel, data));
                }

                // Execute test suites
                if (test instanceof Root_meta_pure_test_TestSuite)
                {
                    // ... test suite execution logic
                }
            }
        }
        return runTestsResult;
    }
    
    // Debug mode execution
    public DebugTestsResult debugTests(List<RunTestsTestableInput> runTestsTestableInputs, 
                                      PureModel pureModel, PureModelContextData data)
    {
        // Similar to doTests but returns debug information
    }
}
```

### Specialized Test Runners

Legend Engine provides specialized test runners for different types of testable elements:

1. **ServiceTestRunner** - Executes tests for service elements
   - Handles service parameter validation
   - Supports both single and multi-execution test suites
   - Validates service results against expected outputs

2. **MappingTestRunner** - Executes tests for mapping elements
   - Tests data transformations
   - Validates mapping results

3. **FunctionTestRunner** - Executes tests for function elements
   - Tests function execution with different inputs
   - Validates function outputs

4. **PersistenceTestRunner** - Executes tests for persistence elements
   - Tests data persistence operations
   - Validates data integrity

Each test runner implements the `TestRunner` interface:

```java
public interface TestRunner
{
    TestResult executeAtomicTest(Root_meta_pure_test_AtomicTest atomicTest, PureModel pureModel, PureModelContextData data);
    List<TestResult> executeTestSuite(Root_meta_pure_test_TestSuite testSuite, List<String> testIds, PureModel pureModel, PureModelContextData data);
    TestDebug debugAtomicTest(Root_meta_pure_test_AtomicTest atomicTest, PureModel pureModel, PureModelContextData data);
    List<TestDebug> debugTestSuite(Root_meta_pure_test_TestSuite testSuite, List<String> testIds, PureModel pureModel, PureModelContextData data);
}
```

### Test Execution Modes

Legend Engine supports two primary test execution modes:

1. **Standard Execution** (`doTests`)
   - Executes tests and returns results
   - Used for normal test execution and validation

2. **Debug Execution** (`debugTests`)
   - Executes tests with additional debugging information
   - Allows step-by-step inspection of test execution
   - Provides variable values and execution state

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

## Test Types and Patterns

### Single-Execution Tests
Single-execution tests run a single operation and validate the result:
- Simple input/output validation
- Function execution tests
- Mapping validation tests

Example of a single-execution test in Pure:
```
function <<test.Test>> testPersonMapping(): Boolean[1]
{
  let result = execute(|Person.all()->project([p | $p.firstName, p | $p.lastName], ['First', 'Last']), personMapping, testRuntime());
  assertEquals(['John', 'Smith'], $result.rows->at(0).values);
}
```

### Multi-Execution Tests
Multi-execution tests run multiple operations in sequence:
- Service tests with multiple steps
- Database tests with setup, execution, and validation
- Complex workflow tests

Example of a multi-execution test suite in Pure:
```
function <<test.TestSuite>> testServiceFlow(): TestSuite[1]
{
  meta::pure::test::TestSuite
  {
    name: 'Service Flow Tests';
    tests: 
    [
      {
        name: 'Create and Retrieve';
        function: {|
          // Step 1: Create data
          let createResult = executeService('my::service::CreatePerson', [^PersonInput(firstName='John', lastName='Smith')]);
          
          // Step 2: Retrieve data
          let retrieveResult = executeService('my::service::GetPerson', [^IdInput(id=$createResult.id)]);
          
          // Validation
          assertEquals('John', $retrieveResult.firstName);
        };
      }
    ];
  }
}
```

### Test Assertions and Validation

Legend Engine provides several assertion mechanisms:

1. **Pure Assertions**
   - `assertEquals(expected, actual)` - Compares values for equality
   - `assertTrue(condition)` - Validates a boolean condition
   - `assertEmpty(collection)` - Validates a collection is empty
   - `assertSize(size, collection)` - Validates collection size

2. **Service Test Assertions**
   - Response status code validation
   - Response body validation
   - Header validation
   - Error condition validation

3. **Mapping Test Assertions**
   - Data transformation validation
   - Schema validation
   - Value conversion validation

## Best Practices

### Writing Tests
1. Test both positive and negative cases
2. Include edge cases and boundary conditions
3. Ensure test independence
4. Use appropriate assertions
5. Document test purpose and expectations

### Writing Effective Tests for New Features

When adding new features to Legend Engine, follow these testing guidelines:

1. **Test Coverage**
   - Test all public APIs and interfaces
   - Include both success and failure scenarios
   - Test edge cases and boundary conditions
   - Test performance characteristics for critical operations

2. **Test Organization**
   - Group related tests in test suites
   - Use descriptive test names that explain what is being tested
   - Organize tests by feature or component

3. **Test Independence**
   - Each test should be independent and not rely on other tests
   - Clean up resources after tests complete
   - Use setup and teardown methods for common initialization

4. **Extension Testing**
   - When implementing a new extension, test all extension points
   - Verify extension discovery and registration
   - Test extension behavior in isolation and integration

5. **Database Connector Testing**
   - Test connection establishment
   - Test SQL generation for all supported operations
   - Test execution against actual database instances
   - Test error handling and recovery

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
