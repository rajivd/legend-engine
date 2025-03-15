# Implementing New Functions in Legend Engine

This guide provides step-by-step instructions for implementing new functions in Legend Engine, including how to create the function definition, implement database-specific translations, and write tests to verify the implementation.

## Table of Contents

1. [Understanding Function Implementation in Legend Engine](#understanding-function-implementation-in-legend-engine)
2. [Step-by-Step Implementation Guide](#step-by-step-implementation-guide)
3. [Template Code for Function Implementation](#template-code-for-function-implementation)
4. [Testing Your Function Implementation](#testing-your-function-implementation)
5. [Parameterized Compatibility Testing (PCT)](#parameterized-compatibility-testing-pct)
6. [Common Pitfalls and Solutions](#common-pitfalls-and-solutions)

## Understanding Function Implementation in Legend Engine

Legend Engine functions are implemented in the Pure language and then translated to various target platforms (SQL dialects, Java, etc.) through a set of adapters. The implementation follows a layered approach:

1. **Core Function Definition**: Defines the function signature and behavior in Pure language
2. **Platform-Specific Adapters**: Translate the Pure function to platform-specific code
3. **Testing Framework**: Verifies the function works correctly across platforms

Functions in Legend Engine can be categorized as:

- **Essential Functions**: Core functions required for basic operations
- **Standard Functions**: Common utility functions
- **Grammar Functions**: Functions related to language features
- **Relation Functions**: Functions specific to relational operations

## Step-by-Step Implementation Guide

### 1. Identify the Function Requirements

Before implementing a new function, clearly define:

- Function name and purpose
- Input parameters and types
- Return type and multiplicity
- Expected behavior across different platforms

### 2. Implement the Pure Function Definition

Create the Pure function definition in the appropriate module:

```pure
function <<PCT.function>> {doc.doc = 'Description of what the function does'} 
meta::pure::functions::{category}::{functionName}(param1:Type1[m], param2:Type2[n]):ReturnType[p]
{
    // Pure implementation of the function
    // This serves as the reference implementation
}
```

Place this file in the appropriate location based on the function category:

- String functions: `legend-engine-pure-code-compiled-core/src/main/resources/core/pure/corefunctions/stringExtension.pure`
- Collection functions: `legend-engine-pure-code-compiled-core/src/main/resources/core/pure/corefunctions/collectionExtension.pure`
- Math functions: `legend-engine-pure-functions-standard-pure/src/main/resources/core_functions_standard/math/...`

### 3. Implement Database-Specific Adapters

For each supported database platform, implement a SQL translation:

```pure
function <<PCT.adapter>> meta::relational::functions::sqlQueryToString::{database}::{functionName}(param1:Type1[m], param2:Type2[n]):String[1]
{
    // Return the SQL string for this database platform
    '{sql_function}(' + $param1 + ', ' + $param2 + ')'
}
```

Add the function to the database extension file:

```pure
dynaFnToSql('{functionName}', $allStates, ^ToSql(format='{sql_format}', transform={p:String[*]|{transformation}})),
```

Place these implementations in:
- `legend-engine-xt-relationalStore-{database}-pure/src/main/resources/core_relational_{database}/relational/sqlQueryToString/{database}Extension.pure`

### 4. Update Default SQL Translation (Optional)

If the function has a common SQL translation pattern across most databases, add it to the default extensions:

```pure
dynaFnToSql('{functionName}', $allStates, ^ToSql(format='{default_sql_format}', transform={p:String[*]|{default_transformation}})),
```

Place this in:
- `legend-engine-xt-relationalStore-core-pure/src/main/resources/core_relational/relational/sqlQueryToString/extensionDefaults.pure`

### 5. Write Tests for the Function

Create tests for the Pure implementation:

```pure
function <<test.Test>> meta::pure::functions::{category}::tests::{functionName}::test{TestName}():Boolean[1]
{
    // Test cases for the function
    assert({functionName}(input1, input2) == expectedOutput);
    assert({functionName}(input3, input4) == expectedOutput2);
}
```

Place these tests in:
- `legend-engine-pure-code-compiled-core/src/main/resources/core/pure/corefunctions/tests/{category}/test{FunctionName}.pure`

## Template Code for Function Implementation

### Example: Implementing a Hypothetical `startsWith` Function

#### 1. Pure Function Definition

```pure
function <<PCT.function>> {doc.doc = 'Returns true if the first string starts with the second string'} 
meta::pure::functions::string::startsWith(string:String[1], prefix:String[1]):Boolean[1]
{
    $string->substring(0, $prefix->length()) == $prefix
}
```

#### 2. Database-Specific Adapter for DuckDB

```pure
function <<PCT.adapter>> meta::relational::functions::sqlQueryToString::duckdb::startsWith(s:String[1], p:String[1]):String[1]
{
    'starts_with(' + $s + ', ' + $p + ')'
}
```

#### 3. DuckDB Extension Entry

```pure
// In duckdbExtension.pure
dynaFnToSql('startsWith', $allStates, ^ToSql(format='starts_with(%s, %s)', transform={p:String[2]|$p})),
```

#### 4. Default SQL Translation

```pure
// In extensionDefaults.pure
dynaFnToSql('startsWith', $allStates, ^ToSql(format=likePattern('%s%'), transform={p:String[2]|[$p->at(0), $p->at(1)->transformLikeParamsDefault()]})),
```

#### 5. Pure Function Tests

```pure
function <<test.Test>> meta::pure::functions::string::tests::startsWith::testStartsWithBasic():Boolean[1]
{
    assert('Hello World'->startsWith('Hello'));
    assert('Hello'->startsWith('H'));
    assert('Hello'->startsWith('Hello'));
    assertFalse('Hello'->startsWith('World'));
    assertFalse('Hello'->startsWith('hello'));
    assertFalse(''->startsWith('H'));
    assert(''->startsWith(''));
}
```

## Testing Your Function Implementation

### 1. Unit Tests for Pure Implementation

Write comprehensive unit tests that cover:
- Basic functionality
- Edge cases
- Error conditions
- Type variations

### 2. PCT Tests for Database Translations

Create PCT tests to verify the function works across database platforms:

```java
// In Test_Relational_{Database}_EssentialFunctions_PCT.java
public class Test_Relational_{Database}_EssentialFunctions_PCT extends PCTReportConfiguration
{
    private static final ReportScope reportScope = PlatformCodeRepositoryProvider.essentialFunctions;
    private static final Adapter adapter = CoreExternalTestConnectionCodeRepositoryProvider.{database}Adapter;
    private static final String platform = "compiled";
    private static final MutableList<ExclusionSpecification> expectedFailures = Lists.mutable.with(
        // Add any expected failures for your function
        one("meta::pure::functions::{category}::tests::{functionName}::test{TestName}_Function_1__Boolean_1_", "Reason for failure"),
    );
    
    // ... rest of the class
}
```

### 3. Database-Specific Function Tests

Create tests that verify the SQL translation works correctly:

```pure
function <<paramTest.Test>> meta::relational::tests::dbSpecificTests::sqlQueryTests::dynaFunctions::{functionName}::test{TestName}(config:DbTestConfig[1]):Boolean[1]
{
  let dynaFunc = ^DynaFunction(name='{functionName}', parameters=[^Literal(value='TestValue1'), ^Literal(value='TestValue2')]);
  let expected = ^Literal(value=true);
  runDynaFunctionDatabaseTest($dynaFunc, $expected, $config);
}
```

## Parameterized Compatibility Testing (PCT)

The PCT framework ensures functions work consistently across platforms:

1. **Function Stereotypes**:
   - `<<PCT.function>>`: Marks a function for PCT testing
   - `<<PCT.adapter>>`: Marks a platform-specific adapter
   - `<<PCT.platformOnly>>`: Marks a function only available on specific platforms

2. **Test Organization**:
   - Tests are organized under `meta::pure::functions::{category}::tests`
   - Each function should have its own test file

3. **Expected Failures**:
   - Document known implementation gaps in `expectedFailures` lists
   - Include clear reasons for failures

4. **PCT Reports**:
   - Reports are generated in JSON format under `target/classes/pct-reports/`
   - Use these reports to track implementation progress

## Common Pitfalls and Solutions

### 1. Type Conversion Issues

**Problem**: Different databases handle type conversions differently.

**Solution**: Implement explicit type conversions in your SQL translations:

```pure
dynaFnToSql('functionName', $allStates, ^ToSql(format='CAST(%s AS {type}) {operation} %s', transform={p:String[2]|$p})),
```

### 2. NULL Handling

**Problem**: NULL values are handled differently across databases.

**Solution**: Add NULL checks in your SQL translations:

```pure
dynaFnToSql('functionName', $allStates, ^ToSql(format='CASE WHEN %s IS NULL THEN NULL ELSE {operation}(%s, %s) END', transform={p:String[2]|[$p->at(0), $p->at(0), $p->at(1)]})),
```

### 3. Function Name Conflicts

**Problem**: Function names might conflict with reserved keywords.

**Solution**: Use database-specific escaping in your SQL translations:

```pure
dynaFnToSql('functionName', $allStates, ^ToSql(format='{escaped_function}(%s, %s)', transform={p:String[2]|$p})),
```

### 4. Performance Considerations

**Problem**: Naive implementations might perform poorly.

**Solution**: Optimize SQL translations for each database:

```pure
// For database A with optimized function
dynaFnToSql('functionName', $allStates, ^ToSql(format='OPTIMIZED_FUNCTION(%s, %s)', transform={p:String[2]|$p})),

// For database B without optimized function
dynaFnToSql('functionName', $allStates, ^ToSql(format='ALTERNATIVE_APPROACH(%s, %s)', transform={p:String[2]|$p})),
```

## Conclusion

Implementing new functions in Legend Engine requires understanding both the Pure language semantics and the target platform specifics. By following this guide, you can create robust, well-tested functions that work consistently across all supported platforms.

Remember to:
1. Start with a clear function specification
2. Implement the Pure reference implementation
3. Add platform-specific translations
4. Write comprehensive tests
5. Document any limitations or expected failures
