# Implementing PCT Functions in Legend Engine

## Overview

Parameterized Compatibility Testing (PCT) is a framework in Legend Engine that allows functions to be written once and executed across multiple database platforms. This guide explains how to implement PCT functions, including the necessary components, extension mechanisms, and testing approaches.

## What is PCT?

PCT enables cross-database compatibility testing by:

1. Defining functions with the `<<PCT.function>>` stereotype
2. Implementing database-specific adapters with the `<<PCT.adapter>>` stereotype
3. Providing SQL translations for each supported database platform
4. Testing function execution across different database types

## Components of PCT Implementation

### 1. PCT Function Definition

PCT functions are defined in Pure code with the `<<PCT.function>>` stereotype:

```pure
function <<PCT.function>> meta::pure::functions::collection::in(value:Any[1], collection:Any[*]):Boolean[1]
{
    $collection->exists(x | $value == $x)
}
```

### 2. Database Adapters

Database adapters are defined with the `<<PCT.adapter>>` stereotype and specify the database type:

```pure
function <<PCT.adapter>> {PCT.adapterName='H2'} meta::relational::tests::pct::testAdapterForRelationalWithH2Execution<X|o>(f:Function<{->X[o]}>[1]):X[o]
{
  meta::relational::tests::pct::testAdapterForRelationalExecution(
      $f,
      meta::pure::testConnection::getTestConnection(DatabaseType.H2)
  )
}
```

### 3. SQL Translation Implementation

SQL translations are implemented in database-specific extension files:

```pure
function meta::relational::functions::sqlQueryToString::snowflake::getSupportedFunctions():Map<String, meta::pure::metamodel::function::Function<{->Any[*]}>>[1]
{
  newMap([
    // SQL translations for functions
    dynaFnToSql('abs',                   $allStates,            ^ToSql(format='abs(%s)')),
    dynaFnToSql('ceiling',               $allStates,            ^ToSql(format='ceiling(%s)')),
    dynaFnToSql('concat',                $allStates,            ^ToSql(format='concat%s', transform={p:String[*]|$p->joinStrings('(', ', ', ')')})),
    // ... more translations
  ]);
}
```

### 4. Reprocessing Mechanism

The PCT framework uses a reprocessing mechanism to transform Pure functions into database-specific SQL:

```pure
function meta::relational::tests::pct::process::reprocess(a:Any[1], state:ProcessingState[1]):ProcessingState[1]
{
  $a->match(
    [
      z:FunctionExpression[1]| /* reprocessing logic */,
      ix:InstanceValue[1]| /* reprocessing logic */,
      // ... more cases
    ]
  );
}
```

## Step-by-Step Guide to Implementing a PCT Function

### 1. Define the Function with PCT.function Stereotype

Create a Pure function with the `<<PCT.function>>` stereotype:

```pure
function <<PCT.function>> meta::pure::functions::myNamespace::myFunction(param1:Type1[1], param2:Type2[*]):ReturnType[1]
{
    // Pure implementation of the function
    // This implementation will be used when the function is executed in Pure
}
```

### 2. Implement Database-Specific SQL Translations

For each supported database type, implement SQL translations in the corresponding extension file:

#### For H2

```pure
// In h2Extension.pure
function meta::relational::functions::sqlQueryToString::h2::getSupportedFunctions():Map<String, meta::pure::metamodel::function::Function<{->Any[*]}>>[1]
{
  newMap([
    // Add your function translation
    dynaFnToSql('myFunction', $allStates, ^ToSql(format='H2_SPECIFIC_SQL_FORMAT(%s)')),
    // ... other translations
  ]);
}
```

#### For Snowflake

```pure
// In snowflakeExtension.pure
function meta::relational::functions::sqlQueryToString::snowflake::getSupportedFunctions():Map<String, meta::pure::metamodel::function::Function<{->Any[*]}>>[1]
{
  newMap([
    // Add your function translation
    dynaFnToSql('myFunction', $allStates, ^ToSql(format='SNOWFLAKE_SPECIFIC_SQL_FORMAT(%s)')),
    // ... other translations
  ]);
}
```

### 3. Handle Complex Transformations (if needed)

For functions that require complex transformations, implement custom transformation logic:

```pure
function meta::relational::functions::sqlQueryToString::myDatabase::transformMyFunctionParams(params:String[*]):String[1]
{
  // Custom transformation logic
  // ...
  $transformedResult;
}

// Then use it in the SQL translation
dynaFnToSql('myFunction', $allStates, ^ToSql(format='SQL_FORMAT(%s)', transform={p:String[*]|$p->transformMyFunctionParams()})),
```

### 4. Create PCT Tests

Create tests to verify your function works across different database platforms:

```pure
function <<PCT.test>> meta::pure::functions::myNamespace::tests::testMyFunction<Z|y>(f:Function<{Function<{->Z[y]}>[1]->Z[y]}>[1]):Boolean[1]
{
    // Test cases for your function
    assert($f->eval(|myFunction(param1, param2)));
    // ... more test cases
}
```

### 5. Register the Function in the PCT Framework

Ensure your function is registered in the PCT framework by adding it to the appropriate module:

```pure
// In your module's extension file
function meta::pure::extension::myModule::getPCTFunctions():Function<Any>[*]
{
  [
    meta::pure::functions::myNamespace::myFunction_Type1_1__Type2_MANY__ReturnType_1_
    // ... other functions
  ]
}
```

## Common Patterns and Best Practices

### 1. Function Parameter Handling

When implementing SQL translations, consider different parameter types and multiplicities:

```pure
// Handle different parameter types
dynaFnToSql('myFunction', $allStates, ^ToSql(format='MY_SQL_FUNC(%s, %s)', transform={p:String[*]|
  // Transform parameters based on type
  if($p->at(0)->contains('VARCHAR'), 
     |'CAST(' + $p->at(1) + ' AS VARCHAR)', 
     |$p->at(1))
})),
```

### 2. Error Handling

Implement proper error handling for unsupported operations:

```pure
// In your SQL translation
dynaFnToSql('myFunction', $allStates, ^ToSql(format='%s', contextAwareTransform={p:String[*], s:SqlGenerationContext[1]|
  if($p->isEmpty(), 
     |fail('myFunction requires at least one parameter'), 
     |'MY_SQL_FUNC(' + $p->makeString(',') + ')')
})),
```

### 3. Database-Specific Features

Leverage database-specific features when appropriate:

```pure
// Use database-specific optimizations
dynaFnToSql('myFunction', $allStates->filter(s|$s.dbType == DatabaseType.Snowflake), 
            ^ToSql(format='SNOWFLAKE_OPTIMIZED_FUNC(%s)')),
dynaFnToSql('myFunction', $allStates->filter(s|$s.dbType != DatabaseType.Snowflake), 
            ^ToSql(format='GENERIC_FUNC(%s)')),
```

## Troubleshooting

### Common Issues

1. **Missing SQL Translation**: If you see the error "No SQL translation exists for the PURE function", you need to implement the SQL translation for that function.

2. **Type Conversion Issues**: Ensure your SQL translation handles type conversions correctly for different database platforms.

3. **Parameter Handling**: Make sure your SQL translation correctly handles all parameter multiplicities and types.

### Debugging Tips

1. Use the `debug` parameter to see the generated SQL:

```pure
let debug = ^DebugContext(debug=true);
meta::relational::tests::pct::testAdapterForRelationalExecution($f, $connection, $debug);
```

2. Examine the PCT reports to identify missing or incorrect translations:

```java
// In your Java test class
@Override
public ReportScope getReportScope()
{
    return YourModuleCodeRepositoryProvider.yourModule;
}
```

## Example: Implementing a New PCT Function

Let's walk through implementing a new PCT function called `formatPhoneNumber`:

### 1. Define the Function

```pure
function <<PCT.function>> meta::pure::functions::string::formatPhoneNumber(phoneNumber:String[1]):String[1]
{
    // Pure implementation
    let cleaned = $phoneNumber->replace('[^0-9]', '');
    if($cleaned->length() == 10,
       |'(' + $cleaned->substring(0, 3) + ') ' + $cleaned->substring(3, 6) + '-' + $cleaned->substring(6, 10),
       |$phoneNumber
    );
}
```

### 2. Implement SQL Translations

#### For H2

```pure
// In h2Extension.pure
dynaFnToSql('formatPhoneNumber', $allStates, ^ToSql(format=
  'CASE WHEN LENGTH(REGEXP_REPLACE(%s, \'[^0-9]\', \'\')) = 10 ' +
  'THEN CONCAT(\'(\', SUBSTRING(REGEXP_REPLACE(%s, \'[^0-9]\', \'\'), 1, 3), \') \', ' +
  'SUBSTRING(REGEXP_REPLACE(%s, \'[^0-9]\', \'\'), 4, 3), \'-\', ' +
  'SUBSTRING(REGEXP_REPLACE(%s, \'[^0-9]\', \'\'), 7, 4)) ' +
  'ELSE %s END',
  transform={p:String[1]|[$p, $p, $p, $p, $p]->makeString(',')}
)),
```

#### For Snowflake

```pure
// In snowflakeExtension.pure
dynaFnToSql('formatPhoneNumber', $allStates, ^ToSql(format=
  'CASE WHEN LENGTH(REGEXP_REPLACE(%s, \'[^0-9]\', \'\')) = 10 ' +
  'THEN CONCAT(\'(\', SUBSTRING(REGEXP_REPLACE(%s, \'[^0-9]\', \'\'), 1, 3), \') \', ' +
  'SUBSTRING(REGEXP_REPLACE(%s, \'[^0-9]\', \'\'), 4, 3), \'-\', ' +
  'SUBSTRING(REGEXP_REPLACE(%s, \'[^0-9]\', \'\'), 7, 4)) ' +
  'ELSE %s END',
  transform={p:String[1]|[$p, $p, $p, $p, $p]->makeString(',')}
)),
```

### 3. Create PCT Tests

```pure
function <<PCT.test>> meta::pure::functions::string::tests::testFormatPhoneNumber<Z|y>(f:Function<{Function<{->Z[y]}>[1]->Z[y]}>[1]):Boolean[1]
{
    assert($f->eval(|'1234567890'->formatPhoneNumber()) == '(123) 456-7890');
    assert($f->eval(|'123-456-7890'->formatPhoneNumber()) == '(123) 456-7890');
    assert($f->eval(|'(123) 456-7890'->formatPhoneNumber()) == '(123) 456-7890');
    assert($f->eval(|'abc'->formatPhoneNumber()) == 'abc');
}
```

## Conclusion

Implementing PCT functions in Legend Engine involves defining functions with the `<<PCT.function>>` stereotype, implementing database-specific adapters, and providing SQL translations for each supported database platform. By following this guide, you can extend the Legend Engine's cross-database compatibility capabilities and ensure your functions work consistently across different database platforms.
