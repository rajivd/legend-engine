# Example Function Implementation: `containsIgnoreCase`

This document provides a complete example of implementing a new function in Legend Engine. We'll implement a hypothetical `containsIgnoreCase` function that checks if a string contains another string, ignoring case sensitivity.

## 1. Function Specification

**Name**: `containsIgnoreCase`  
**Category**: String functions  
**Purpose**: Check if a string contains another string, ignoring case sensitivity  
**Parameters**:
- `source`: String[1] - The source string to search in
- `substring`: String[1] - The substring to search for

**Return Type**: Boolean[1] - True if the source contains the substring (case-insensitive), false otherwise  
**Behavior**: Should return true if the lowercase version of the source string contains the lowercase version of the substring

## 2. Pure Function Implementation

First, we'll implement the Pure function definition:

```pure
// File: legend-engine-pure-code-compiled-core/src/main/resources/core/pure/corefunctions/stringExtension.pure

function <<PCT.function>> {doc.doc = 'Returns true if the first string contains the second string, ignoring case sensitivity'} 
meta::pure::functions::string::containsIgnoreCase(source:String[0..1], substring:String[1]):Boolean[1]
{
    $source->isNotEmpty() && $source->toOne()->toLower()->contains($substring->toLower())
}
```

## 3. Database-Specific Adapters

### 3.1 DuckDB Implementation

```pure
// File: legend-engine-xt-relationalStore-duckdb-pure/src/main/resources/core_relational_duckdb/relational/sqlQueryToString/duckdbExtension.pure

// Add to the existing dynaFnToSql list
dynaFnToSql('containsIgnoreCase', $allStates, ^ToSql(format='contains(lower(%s), lower(%s))', transform={p:String[2]|$p})),
```

### 3.2 Snowflake Implementation

```pure
// File: legend-engine-xt-relationalStore-snowflake-pure/src/main/resources/core_relational_snowflake/relational/sqlQueryToString/snowflakeExtension.pure

// Add to the existing dynaFnToSql list
dynaFnToSql('containsIgnoreCase', $allStates, ^ToSql(format='contains(lower(%s), lower(%s))', transform={p:String[2]|$p})),
```

### 3.3 H2 Implementation

```pure
// File: legend-engine-xt-relationalStore-core-pure/src/main/resources/core_relational/relational/sqlQueryToString/dbSpecific/h2/h2Extension.pure

// Add to the existing dynaFnToSql list
dynaFnToSql('containsIgnoreCase', $allStates, ^ToSql(format='LOWER(%s) LIKE ''%'' || LOWER(%s) || ''%''', transform={p:String[2]|$p})),
```

### 3.4 Default Implementation

```pure
// File: legend-engine-xt-relationalStore-core-pure/src/main/resources/core_relational/relational/sqlQueryToString/extensionDefaults.pure

// Add to the existing dynaFnToSql list
dynaFnToSql('containsIgnoreCase', $allStates, ^ToSql(format='LOWER(%s) LIKE ''%'' || LOWER(%s) || ''%''', transform={p:String[2]|$p})),
```

## 4. Pure Function Tests

```pure
// File: legend-engine-pure-code-compiled-core/src/main/resources/core/pure/corefunctions/tests/string/testContainsIgnoreCase.pure

import meta::pure::profiles::*;

function <<test.Test>> meta::pure::functions::string::tests::containsIgnoreCase::testContainsIgnoreCaseBasic():Boolean[1]
{
    // Basic functionality
    assert('Hello World'->containsIgnoreCase('hello'));
    assert('Hello World'->containsIgnoreCase('WORLD'));
    assert('Hello World'->containsIgnoreCase('o W'));
    assert('HELLO WORLD'->containsIgnoreCase('hello'));
    assert('hello world'->containsIgnoreCase('WORLD'));
    
    // Edge cases
    assert('Hello World'->containsIgnoreCase(''));
    assertFalse(''->containsIgnoreCase('hello'));
    assertFalse([]->containsIgnoreCase('hello'));
    
    // Negative cases
    assertFalse('Hello World'->containsIgnoreCase('goodbye'));
    assertFalse('Hello World'->containsIgnoreCase('Worlds'));
}

function <<test.Test>> meta::pure::functions::string::tests::containsIgnoreCase::testContainsIgnoreCaseSpecialChars():Boolean[1]
{
    // Special characters
    assert('Hello, World!'->containsIgnoreCase('hello,'));
    assert('Hello, World!'->containsIgnoreCase('WORLD!'));
    assert('Hello\nWorld'->containsIgnoreCase('hello\n'));
    assert('Hello\tWorld'->containsIgnoreCase('HELLO\t'));
    
    // Unicode characters
    assert('Héllö Wörld'->containsIgnoreCase('héllö'));
    assert('Héllö Wörld'->containsIgnoreCase('WÖRLD'));
}
```

## 5. Database-Specific Function Tests

```pure
// File: legend-engine-xt-relationalStore-core-pure/src/main/resources/core_relational/relational/sqlQueryToString/testSuite/dynaFunctions/string.pure

function <<paramTest.Test>> meta::relational::tests::dbSpecificTests::sqlQueryTests::dynaFunctions::containsIgnoreCase::testBasic(config:DbTestConfig[1]):Boolean[1]
{
  let dynaFunc = ^DynaFunction(name='containsIgnoreCase', parameters=[^Literal(value='Hello World'), ^Literal(value='hello')]);
  let expected = ^Literal(value=true);
  runDynaFunctionDatabaseTest($dynaFunc, $expected, $config);
}

function <<paramTest.Test>> meta::relational::tests::dbSpecificTests::sqlQueryTests::dynaFunctions::containsIgnoreCase::testNegative(config:DbTestConfig[1]):Boolean[1]
{
  let dynaFunc = ^DynaFunction(name='containsIgnoreCase', parameters=[^Literal(value='Hello World'), ^Literal(value='goodbye')]);
  let expected = ^Literal(value=false);
  runDynaFunctionDatabaseTest($dynaFunc, $expected, $config);
}

function <<paramTest.Test>> meta::relational::tests::dbSpecificTests::sqlQueryTests::dynaFunctions::containsIgnoreCase::testEmptySource(config:DbTestConfig[1]):Boolean[1]
{
  let dynaFunc = ^DynaFunction(name='containsIgnoreCase', parameters=[^Literal(value=''), ^Literal(value='hello')]);
  let expected = ^Literal(value=false);
  runDynaFunctionDatabaseTest($dynaFunc, $expected, $config);
}

function <<paramTest.Test>> meta::relational::tests::dbSpecificTests::sqlQueryTests::dynaFunctions::containsIgnoreCase::testEmptySubstring(config:DbTestConfig[1]):Boolean[1]
{
  let dynaFunc = ^DynaFunction(name='containsIgnoreCase', parameters=[^Literal(value='Hello World'), ^Literal(value='')]);
  let expected = ^Literal(value=true);
  runDynaFunctionDatabaseTest($dynaFunc, $expected, $config);
}
```

## 6. PCT Test Configuration

```java
// File: legend-engine-xt-relationalStore-duckdb-PCT/src/test/java/org/finos/legend/engine/plan/execution/stores/relational/test/duckdb/pct/Test_Relational_DuckDB_EssentialFunctions_PCT.java

// Add to the expectedFailures list if there are any known issues
private static final MutableList<ExclusionSpecification> expectedFailures = Lists.mutable.with(
    // ... existing failures
    
    // If there are any known issues with containsIgnoreCase, add them here
    // one("meta::pure::functions::string::tests::containsIgnoreCase::testContainsIgnoreCaseSpecialChars_Function_1__Boolean_1_", "Reason for failure"),
);
```

## 7. Building and Testing

After implementing the function, follow these steps to build and test:

1. Build the project:
   ```bash
   mvn clean install -DskipTests
   ```

2. Run the specific tests for your function:
   ```bash
   mvn test -Dtest=Test_Relational_DuckDB_EssentialFunctions_PCT
   ```

3. Check the PCT report:
   ```bash
   cat target/classes/pct-reports/duckdb-essential-functions-pct-report.json
   ```

## 8. Troubleshooting Common Issues

### 8.1 NULL Handling

If your function needs to handle NULL values properly, consider modifying the SQL translation:

```pure
dynaFnToSql('containsIgnoreCase', $allStates, ^ToSql(format='CASE WHEN %s IS NULL THEN false ELSE contains(lower(%s), lower(%s)) END', transform={p:String[2]|[$p->at(0), $p->at(0), $p->at(1)]})),
```

### 8.2 Performance Optimization

For better performance on large strings, some databases might benefit from a different approach:

```pure
// For databases with POSITION function
dynaFnToSql('containsIgnoreCase', $allStates, ^ToSql(format='POSITION(LOWER(%s) IN LOWER(%s)) > 0', transform={p:String[2]|[$p->at(1), $p->at(0)]})),
```

## 9. Documentation

Update the function documentation to include:

1. Function purpose and behavior
2. Parameter descriptions
3. Return value description
4. Examples of usage
5. Any platform-specific considerations

## Conclusion

This example demonstrates the complete process of implementing a new `containsIgnoreCase` function in Legend Engine. By following this pattern, you can implement other functions while ensuring they work consistently across all supported database platforms.
