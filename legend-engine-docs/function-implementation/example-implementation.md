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

## 10. Java Implementation

When a Pure function is used in Legend Engine, it needs a corresponding Java implementation. This section demonstrates how to implement the Java side of our `containsIgnoreCase` function.

### 10.1 Java Helper Method

First, add a static method to `FunctionsHelper.java`:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/main/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/FunctionsHelper.java

public static boolean containsIgnoreCase(String source, String substring)
{
    return source != null && substring != null && source.toLowerCase().contains(substring.toLowerCase());
}
```

### 10.2 Native Function Implementation

Next, create a class that extends `AbstractNativeFunctionGeneric` to connect the Pure function to the Java implementation:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/main/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/natives/string/ContainsIgnoreCase.java

package org.finos.legend.pure.runtime.java.extension.functions.compiled.natives.string;

import org.finos.legend.pure.runtime.java.compiled.generation.processors.natives.AbstractNativeFunctionGeneric;

public class ContainsIgnoreCase extends AbstractNativeFunctionGeneric
{
    public ContainsIgnoreCase()
    {
        super("FunctionsGen.containsIgnoreCase", 
              new Class[]{String.class, String.class}, 
              "containsIgnoreCase_String_$0_1$__String_1__Boolean_1_");
    }
}
```

### 10.3 Register the Native Function

Finally, register the native function in `FunctionsExtensionCompiled.java`:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/main/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/FunctionsExtensionCompiled.java

@Override
public List<NativeFunction> getExtraNativeFunctions()
{
    return Lists.mutable.with(
        // ... existing functions
        new ContainsIgnoreCase()
    );
}
```

### 10.4 Update FunctionsGen.java

Add the function to `FunctionsGen.java` to make it available to the generated code:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/main/resources/org/finos/legend/pure/runtime/java/extension/functions/compiled/FunctionsGen.java

public static boolean containsIgnoreCase(String source, String substring)
{
    return org.finos.legend.pure.runtime.java.extension.functions.compiled.FunctionsHelper.containsIgnoreCase(source, substring);
}
```

## 11. Java Implementation Testing

After implementing the Java side of the `containsIgnoreCase` function, we need to test it thoroughly to ensure it works correctly. This section demonstrates how to test the Java implementation.

### 11.1 Unit Tests for Java Helper Methods

First, create unit tests for the `FunctionsHelper.containsIgnoreCase` method:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/test/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/FunctionsHelperTest.java

package org.finos.legend.pure.runtime.java.extension.functions.compiled;

import org.junit.Assert;
import org.junit.Test;

public class FunctionsHelperTest
{
    @Test
    public void testContainsIgnoreCase()
    {
        // Basic functionality
        Assert.assertTrue(FunctionsHelper.containsIgnoreCase("Hello World", "hello"));
        Assert.assertTrue(FunctionsHelper.containsIgnoreCase("Hello World", "WORLD"));
        Assert.assertTrue(FunctionsHelper.containsIgnoreCase("Hello World", "o W"));
        
        // Edge cases
        Assert.assertTrue(FunctionsHelper.containsIgnoreCase("Hello World", ""));
        Assert.assertFalse(FunctionsHelper.containsIgnoreCase("", "hello"));
        Assert.assertFalse(FunctionsHelper.containsIgnoreCase(null, "hello"));
        
        // Negative cases
        Assert.assertFalse(FunctionsHelper.containsIgnoreCase("Hello World", "goodbye"));
        Assert.assertFalse(FunctionsHelper.containsIgnoreCase("Hello World", "Worlds"));
    }
}
```

### 11.2 Integration Tests for Pure-to-Java Compilation

Next, create integration tests that verify the function works correctly when compiled from Pure to Java:

```java
// File: legend-engine-pure-runtime-java-extension-compiled-functions-unclassified/src/test/java/org/finos/legend/pure/runtime/java/extension/functions/compiled/Test_Pure_Java_Functions.java

package org.finos.legend.pure.runtime.java.extension.functions.compiled;

import org.finos.legend.pure.m3.execution.FunctionExecution;
import org.finos.legend.pure.m3.tests.function.base.PureExpressionTest;
import org.finos.legend.pure.m4.coreinstance.CoreInstance;
import org.junit.Assert;
import org.junit.Test;

public class Test_Pure_Java_Functions extends PureExpressionTest
{
    @Test
    public void testContainsIgnoreCaseCompilation()
    {
        // Test basic functionality
        CoreInstance result1 = compileAndExecute("'Hello World'->containsIgnoreCase('hello')");
        Assert.assertEquals(true, this.getResultBoolean(result1));
        
        // Test edge cases
        CoreInstance result2 = compileAndExecute("'Hello World'->containsIgnoreCase('')");
        Assert.assertEquals(true, this.getResultBoolean(result2));
        
        CoreInstance result3 = compileAndExecute("''->containsIgnoreCase('hello')");
        Assert.assertEquals(false, this.getResultBoolean(result3));
        
        // Test negative cases
        CoreInstance result4 = compileAndExecute("'Hello World'->containsIgnoreCase('goodbye')");
        Assert.assertEquals(false, this.getResultBoolean(result4));
    }
    
    @Override
    protected FunctionExecution getFunctionExecution()
    {
        return this.getCompiledFunctionExecution();
    }
}
```

### 11.3 Testing Native Function Registration

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
    public void testContainsIgnoreCaseRegistration()
    {
        // Get the list of native functions from the extension
        FunctionsExtensionCompiled extension = new FunctionsExtensionCompiled();
        ListIterable<NativeFunction> nativeFunctions = extension.getExtraNativeFunctions();
        
        // Verify that our function is in the list
        boolean found = false;
        for (NativeFunction function : nativeFunctions)
        {
            if (function.getName().equals("containsIgnoreCase_String_$0_1$__String_1__Boolean_1_"))
            {
                found = true;
                break;
            }
        }
        
        Assert.assertTrue("ContainsIgnoreCase function should be registered", found);
    }
}
```

### 11.4 Running the Tests

To run the tests, use the following Maven command:

```bash
mvn test -Dtest=FunctionsHelperTest,Test_Pure_Java_Functions,NativeFunctionRegistrationTest
```

This will run all the tests for the Java implementation of the `containsIgnoreCase` function.

## Conclusion

This example demonstrates the complete process of implementing a new `containsIgnoreCase` function in Legend Engine, from Pure definition to Java implementation and testing. By following this pattern, you can implement other functions while ensuring they work consistently across all supported database platforms.
