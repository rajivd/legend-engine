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

## Java Implementation for Native Functions

When implementing native functions in Legend Engine, you also need to provide Java implementations that handle the platform-specific behavior. This section demonstrates how to implement the Java side of our hypothetical `geoDistance` function.

### 1. Java Helper Method

First, add a static method to `FunctionsHelper.java` that will throw an appropriate exception for unsupported platforms:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/main/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/FunctionsHelper.java

public static double geoDistance(double lat1, double long1, double lat2, double long2, SourceInformation sourceInformation)
{
    throw new PureExecutionException(
        sourceInformation,
        "The geoDistance function is only available on Snowflake. This function cannot be executed in Pure.",
        Stacks.mutable.empty()
    );
}
```

### 2. Native Function Implementation

Next, create a class that extends `AbstractNativeFunctionGeneric` to connect the Pure function to the Java implementation:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/main/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/natives/geo/GeoDistance.java

package org.finos.legend.pure.runtime.java.extension.functions.compiled.natives.geo;

import org.finos.legend.pure.runtime.java.compiled.generation.processors.natives.AbstractNativeFunctionGeneric;

public class GeoDistance extends AbstractNativeFunctionGeneric
{
    public GeoDistance()
    {
        super("FunctionsGen.geoDistance", 
              new Class[]{Double.class, Double.class, Double.class, Double.class, SourceInformation.class}, 
              "geoDistance_Float_1__Float_1__Float_1__Float_1__Float_1_");
    }
}
```

### 3. Register the Native Function

Register the native function in `FunctionsExtensionCompiled.java`:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/main/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/FunctionsExtensionCompiled.java

@Override
public List<NativeFunction> getExtraNativeFunctions()
{
    return Lists.mutable.with(
        // ... existing functions
        new GeoDistance()
    );
}
```

### 4. Update FunctionsGen.java

Add the function to `FunctionsGen.java` to make it available to the generated code:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/main/resources/org/finos/legend/pure/runtime/java/extension/functions/compiled/FunctionsGen.java

public static double geoDistance(double lat1, double long1, double lat2, double long2, SourceInformation sourceInformation)
{
    return org.finos.legend.pure.runtime.java.extension.functions.compiled.FunctionsHelper.geoDistance(lat1, long1, lat2, long2, sourceInformation);
}
```

### 5. Handling Platform-Specific Execution

For platform-specific functions, the Java implementation primarily serves to:

1. Provide appropriate error messages when the function is executed in Pure
2. Support the compilation process by defining the function signature
3. Enable proper error reporting with source information

The actual execution of the function happens in the database when the SQL is generated and executed, not in the Java implementation.

## Java Implementation Testing for Native Functions

After implementing the Java side of a native function, you need to test it thoroughly to ensure it behaves correctly. This section demonstrates how to test the Java implementation of platform-specific functions.

### 1. Unit Tests for Java Helper Methods

First, create unit tests for the `FunctionsHelper` method that verify it throws the correct exception:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/test/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/FunctionsHelperTest.java

package org.finos.legend.pure.runtime.java.extension.functions.compiled;

import org.finos.legend.pure.m3.exception.PureExecutionException;
import org.finos.legend.pure.m4.coreinstance.SourceInformation;
import org.junit.Assert;
import org.junit.Test;

public class FunctionsHelperTest
{
    @Test
    public void testGeoDistance()
    {
        // Create a source information object for error reporting
        SourceInformation sourceInformation = new SourceInformation("test.pure", 1, 1, 1, 20);
        
        try
        {
            // Call the function and expect an exception
            FunctionsHelper.geoDistance(40.7128, -74.0060, 40.7128, -74.0060, sourceInformation);
            Assert.fail("Expected PureExecutionException to be thrown");
        }
        catch (PureExecutionException e)
        {
            // Verify the exception message
            Assert.assertTrue(e.getMessage().contains("only available on Snowflake"));
            Assert.assertEquals(sourceInformation, e.getSourceInformation());
        }
    }
}
```

### 2. Integration Tests for Pure-to-Java Compilation

Next, create integration tests that verify the function throws the correct exception when compiled from Pure to Java:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/test/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/Test_Pure_Java_PlatformSpecificFunctions.java

package org.finos.legend.pure.runtime.java.extension.functions.compiled;

import org.finos.legend.pure.m3.exception.PureExecutionException;
import org.finos.legend.pure.m3.execution.FunctionExecution;
import org.finos.legend.pure.m3.tests.function.base.PureExpressionTest;
import org.junit.Assert;
import org.junit.Test;

public class Test_Pure_Java_PlatformSpecificFunctions extends PureExpressionTest
{
    @Test
    public void testGeoDistanceCompilation()
    {
        try
        {
            // Compile and execute a Pure expression that calls the platform-specific function
            compileAndExecute("geoDistance(40.7128, -74.0060, 40.7128, -74.0060)");
            Assert.fail("Expected PureExecutionException to be thrown");
        }
        catch (PureExecutionException e)
        {
            // Verify the exception message
            Assert.assertTrue(e.getMessage().contains("only available on Snowflake"));
        }
    }
    
    @Override
    protected FunctionExecution getFunctionExecution()
    {
        return this.getCompiledFunctionExecution();
    }
}
```

### 3. Testing Native Function Registration

Finally, verify that the native function is properly registered:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/test/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/NativeFunctionRegistrationTest.java

package org.finos.legend.pure.runtime.java.extension.functions.compiled;

import org.eclipse.collections.api.list.ListIterable;
import org.finos.legend.pure.runtime.java.compiled.generation.processors.natives.NativeFunction;
import org.junit.Assert;
import org.junit.Test;

public class NativeFunctionRegistrationTest
{
    @Test
    public void testGeoDistanceRegistration()
    {
        // Get the list of native functions from the extension
        FunctionsExtensionCompiled extension = new FunctionsExtensionCompiled();
        ListIterable<NativeFunction> nativeFunctions = extension.getExtraNativeFunctions();
        
        // Verify that our function is in the list
        boolean found = false;
        for (NativeFunction function : nativeFunctions)
        {
            if (function.getName().equals("geoDistance_Float_1__Float_1__Float_1__Float_1__Float_1_"))
            {
                found = true;
                break;
            }
        }
        
        Assert.assertTrue("GeoDistance function should be registered", found);
    }
}
```

### 4. Testing Platform-Specific Behavior

For platform-specific functions, it's important to test that:

1. The function throws the correct exception with a clear message when executed in Pure
2. The exception includes the source information for proper error reporting
3. The function is properly registered in the native function registry

### 5. Running the Tests

To run the tests, use the following Maven command:

```bash
mvn test -Dtest=FunctionsHelperTest,Test_Pure_Java_PlatformSpecificFunctions,NativeFunctionRegistrationTest
```

This will run all the tests for the Java implementation of the platform-specific function.

## Conclusion

Implementing native functions in Legend Engine allows you to leverage the full power of specific database platforms while maintaining a consistent interface in your Pure code. By following this guide, you can create robust, well-tested native functions that provide optimal performance on supported platforms.

Remember to:
1. Clearly mark functions as platform-specific with `<<PCT.platformOnly>>`
2. Implement database-specific adapters for supported platforms
3. Provide appropriate Java implementations that handle unsupported platforms gracefully
4. Document platform limitations and version requirements
5. Configure PCT tests with appropriate expected failures
6. Write comprehensive tests for supported platforms
7. Test Java implementations to ensure proper error handling
