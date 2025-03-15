# Implementing Native Functions in Legend Engine

This guide provides step-by-step instructions for implementing native functions in Legend Engine, which are functions that leverage the native capabilities of specific database platforms rather than relying on Pure language translations.

## Table of Contents

1. [Understanding Native Functions](#understanding-native-functions)
2. [When to Use Native Functions](#when-to-use-native-functions)
3. [Step-by-Step Implementation Guide](#step-by-step-implementation-guide)
4. [Template Code for Native Function Implementation](#template-code-for-native-function-implementation)
5. [Testing Native Functions](#testing-native-functions)
6. [Common Pitfalls and Solutions](#common-pitfalls-and-solutions)

## Understanding Native Functions

Native functions in Legend Engine are functions that directly leverage the built-in capabilities of a specific database platform. Instead of implementing the function logic in Pure and then translating it to SQL, native functions map directly to the database's native SQL functions.

Key characteristics of native functions:

1. **Platform-Specific**: They are specific to a particular database platform
2. **Performance Optimized**: They leverage optimized implementations in the database
3. **Feature-Rich**: They can access database-specific features not available in Pure

## When to Use Native Functions

Consider implementing a native function when:

1. **Performance is Critical**: The database's native implementation is significantly more efficient
2. **Complex Functionality**: The function requires complex logic that's already implemented in the database
3. **Database-Specific Features**: The function needs to use features unique to a specific database
4. **No Pure Equivalent**: There's no reasonable way to implement the function in Pure language

## Step-by-Step Implementation Guide

### 1. Identify the Native Function Requirements

Before implementing a native function, clearly define:

- Function name and purpose
- Input parameters and types
- Return type and multiplicity
- Target database platform(s)
- Native SQL function to leverage

### 2. Implement the Pure Function Definition with PCT.platformOnly Stereotype

Create the Pure function definition with the `<<PCT.platformOnly>>` stereotype:

```pure
function <<PCT.platformOnly>> {doc.doc = 'Description of what the function does. This function is only available on specific platforms.'} 
meta::pure::functions::{category}::{functionName}(param1:Type1[m], param2:Type2[n]):ReturnType[p]
{
    // This function is only available on specific platforms
    // No Pure implementation is provided
    fail('This function is only available on specific platforms');
}
```

Place this file in the appropriate location based on the function category.

### 3. Implement Database-Specific Adapter

For each supported database platform, implement a SQL translation:

```pure
function <<PCT.adapter>> meta::relational::functions::sqlQueryToString::{database}::{functionName}(param1:Type1[m], param2:Type2[n]):String[1]
{
    // Return the SQL string for this database's native function
    '{native_sql_function}(' + $param1 + ', ' + $param2 + ')'
}
```

Add the function to the database extension file:

```pure
dynaFnToSql('{functionName}', $allStates, ^ToSql(format='{native_sql_format}', transform={p:String[*]|{transformation}})),
```

Place these implementations in:
- `legend-engine-xt-relationalStore-{database}-pure/src/main/resources/core_relational_{database}/relational/sqlQueryToString/{database}Extension.pure`

### 4. Document the Platform Limitations

In the function documentation, clearly specify:

- Which platforms support the function
- Any platform-specific behavior differences
- Alternative approaches for unsupported platforms

### 5. Update Expected Failures in PCT Tests

Since the function is platform-specific, add expected failures for unsupported platforms:

```java
// In Test_Relational_{UnsupportedDatabase}_EssentialFunctions_PCT.java
private static final MutableList<ExclusionSpecification> expectedFailures = Lists.mutable.with(
    // ... existing failures
    
    // Add expected failure for your platform-specific function
    one("meta::pure::functions::{category}::tests::{functionName}::test{TestName}_Function_1__Boolean_1_", 
        "This function is only available on specific platforms"),
);
```

### 6. Write Tests for the Function

Create tests for the native function implementation:

```pure
function <<test.Test>> meta::pure::functions::{category}::tests::{functionName}::test{TestName}():Boolean[1]
{
    // Test cases for the function
    // These will only pass on supported platforms
    assert({functionName}(input1, input2) == expectedOutput);
    assert({functionName}(input3, input4) == expectedOutput2);
}
```

## Template Code for Native Function Implementation

### Example: Implementing a Hypothetical `geoDistance` Native Function for Snowflake

#### 1. Pure Function Definition

```pure
function <<PCT.platformOnly>> {doc.doc = 'Calculates the distance between two geographic points. This function is only available on Snowflake.'} 
meta::pure::functions::geo::geoDistance(lat1:Float[1], long1:Float[1], lat2:Float[1], long2:Float[1]):Float[1]
{
    // This function is only available on Snowflake
    fail('This function is only available on Snowflake');
}
```

#### 2. Snowflake-Specific Adapter

```pure
function <<PCT.adapter>> meta::relational::functions::sqlQueryToString::snowflake::geoDistance(lat1:Float[1], long1:Float[1], lat2:Float[1], long2:Float[1]):String[1]
{
    'HAVERSINE(' + $lat1 + ', ' + $long1 + ', ' + $lat2 + ', ' + $long2 + ')'
}
```

#### 3. Snowflake Extension Entry

```pure
// In snowflakeExtension.pure
dynaFnToSql('geoDistance', $allStates, ^ToSql(format='HAVERSINE(%s, %s, %s, %s)', transform={p:String[4]|$p})),
```

#### 4. Expected Failures for Unsupported Platforms

```java
// In Test_Relational_H2_EssentialFunctions_PCT.java
private static final MutableList<ExclusionSpecification> expectedFailures = Lists.mutable.with(
    // ... existing failures
    
    // Add expected failure for the Snowflake-specific function
    one("meta::pure::functions::geo::tests::geoDistance::testGeoDistanceBasic_Function_1__Boolean_1_", 
        "This function is only available on Snowflake"),
);
```

#### 5. Function Tests

```pure
function <<test.Test>> meta::pure::functions::geo::tests::geoDistance::testGeoDistanceBasic():Boolean[1]
{
    // These tests will only pass on Snowflake
    assertEq(0.0, geoDistance(40.7128, -74.0060, 40.7128, -74.0060));
    assertApproxEq(2.5, geoDistance(40.7128, -74.0060, 40.7128, -74.0360), 0.1);
}
```

## Testing Native Functions

### 1. Platform-Specific Unit Tests

Write tests that are clearly marked as platform-specific:

```pure
function <<test.Test>> {doc.doc = 'This test only passes on Snowflake'} 
meta::pure::functions::geo::tests::geoDistance::testGeoDistanceSnowflake():Boolean[1]
{
    // Snowflake-specific test
    assertEq(0.0, geoDistance(40.7128, -74.0060, 40.7128, -74.0060));
}
```

### 2. PCT Tests with Expected Failures

Configure PCT tests to expect failures on unsupported platforms:

```java
// In Test_Relational_DuckDB_EssentialFunctions_PCT.java
private static final MutableList<ExclusionSpecification> expectedFailures = Lists.mutable.with(
    // ... existing failures
    
    // Add expected failures for platform-specific functions
    one("meta::pure::functions::geo::tests::geoDistance::testGeoDistanceBasic_Function_1__Boolean_1_", 
        "This function is only available on Snowflake"),
);
```

### 3. Database-Specific Function Tests

Create tests that verify the SQL translation works correctly on supported platforms:

```pure
function <<paramTest.Test>> meta::relational::tests::dbSpecificTests::sqlQueryTests::dynaFunctions::geoDistance::testBasic(config:DbTestConfig[1]):Boolean[1]
{
  let dynaFunc = ^DynaFunction(name='geoDistance', parameters=[
    ^Literal(value=40.7128), ^Literal(value=-74.0060), 
    ^Literal(value=40.7128), ^Literal(value=-74.0060)
  ]);
  let expected = ^Literal(value=0.0);
  runDynaFunctionDatabaseTest($dynaFunc, $expected, $config);
}
```

## Common Pitfalls and Solutions

### 1. Function Availability Across Database Versions

**Problem**: Native functions might be available in some versions of a database but not others.

**Solution**: Document version requirements and implement version checks:

```pure
dynaFnToSql('geoDistance', 
    $allStates->filter(s | $s.dbVersion >= '5.0'), 
    ^ToSql(format='HAVERSINE(%s, %s, %s, %s)', transform={p:String[4]|$p})),
```

### 2. Parameter Type Differences

**Problem**: Native functions might expect different parameter types than your Pure function.

**Solution**: Add explicit type conversions in your SQL translations:

```pure
dynaFnToSql('geoDistance', $allStates, ^ToSql(
    format='HAVERSINE(CAST(%s AS FLOAT), CAST(%s AS FLOAT), CAST(%s AS FLOAT), CAST(%s AS FLOAT))', 
    transform={p:String[4]|$p})),
```

### 3. Return Type Differences

**Problem**: Native functions might return different types than your Pure function expects.

**Solution**: Add explicit type conversions for the return value:

```pure
dynaFnToSql('geoDistance', $allStates, ^ToSql(
    format='CAST(HAVERSINE(%s, %s, %s, %s) AS DOUBLE)', 
    transform={p:String[4]|$p})),
```

### 4. Function Name Conflicts

**Problem**: Function names might conflict with reserved keywords or other functions.

**Solution**: Use database-specific escaping in your SQL translations:

```pure
dynaFnToSql('geoDistance', $allStates, ^ToSql(
    format='"HAVERSINE"(%s, %s, %s, %s)', 
    transform={p:String[4]|$p})),
```

## Conclusion

Implementing native functions in Legend Engine allows you to leverage the full power of specific database platforms while maintaining a consistent interface in your Pure code. By following this guide, you can create robust, well-tested native functions that provide optimal performance on supported platforms.

Remember to:
1. Clearly mark functions as platform-specific with `<<PCT.platformOnly>>`
2. Implement database-specific adapters for supported platforms
3. Document platform limitations and version requirements
4. Configure PCT tests with appropriate expected failures
5. Write comprehensive tests for supported platforms
