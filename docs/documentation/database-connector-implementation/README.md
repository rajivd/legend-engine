# Database Connector Implementation

## Overview
Legend Engine supports various database connectors through its extensible architecture. This document outlines the process and patterns for implementing database connectors in Legend Engine, with a focus on relational database connectors.

## Database Connector Components

Developing a new relational connector has 5 main parts:

1. **DB-Specific SQL Generation Logic**: Generate SQL queries tailored to the specific database dialect
   - Implement database-specific SQL syntax and functions
   - Handle dialect-specific query optimization
   - Support database-specific data types and conversions

2. **Connector-Specific Data Source and Authentication Specifications**: Model the connection parameters and authentication strategies
   - Define protocol classes for connection specifications
   - Implement authentication strategy specifications
   - Support serialization/deserialization of connection parameters

3. **Driver Logic and Authentication Logic**: Implement the connection and query execution
   - Create connection managers for database interaction
   - Implement authentication flows for secure connections
   - Handle connection pooling and resource management

4. **Integration Testing**: Test SQL generation and execution against a test database instance
   - Validate SQL generation against expected output
   - Test query execution with test data
   - Verify data type handling and conversions

5. **Connection Acquisition Testing**: Test connection establishment with the database
   - Verify connection establishment with different authentication methods
   - Test connection parameter validation
   - Ensure proper error handling for connection failures

### Prerequisites

Before implementing a new database connector, ensure that:

1. The database type is included in the `DatabaseType` enum in both:
   - [Pure DatabaseType](https://github.com/finos/legend-pure/blob/master/legend-pure-m2-store-relational/src/main/resources/platform/relational/relationalRuntime.pure)
   - [Protocol DatabaseType](https://github.com/finos/legend-engine/blob/master/legend-engine-xt-relationalStore-protocol/src/main/java/org/finos/legend/engine/protocol/pure/v1/model/packageableElement/store/relational/connection/DatabaseType.java)

2. You have access to a test instance of the database for integration testing
3. You understand the SQL dialect and connection requirements of the database

## Module Structure

A relational connector follows a specific module structure, where `<dbType>` represents the database type (e.g., `sqlserver`, `snowflake`, `h2`):

1. **\<dbType\>-protocol**: Defines POJOs representing connector-specific data source and authentication strategy specifications
   - Contains Java classes for connection specifications
   - Defines authentication strategy classes
   - Implements serialization/deserialization support

2. **\<dbType\>-pure**: Contains DB-specific SQL generation logic in Pure language and specifies PURE to JSON conversion
   - Implements SQL generation functions in Pure language
   - Defines database-specific query transformations
   - Specifies mappings between Pure and JSON representations

3. **\<dbType\>-grammar**: Contains ANTLR-based code for bi-directional conversion between textual DSLs and POJOs
   - Implements parsers for connection strings
   - Provides composers for generating textual representations
   - Handles error reporting and validation

4. **\<dbType\>-execution**: Defines driver and authentication logic
   - Implements connection managers
   - Provides authentication flows
   - Handles query execution and result processing

5. **\<dbType\>-execution-tests**: Contains integration tests for connectivity and execution
   - Tests SQL generation against expected output
   - Validates query execution with test data
   - Verifies connection establishment

### Using the Maven Archetype

Legend Engine provides a Maven archetype to generate the base project structure for a new database connector:

```bash
mvn archetype:generate \
  -DarchetypeGroupId=org.finos.legend.engine \
  -DarchetypeArtifactId=legend-engine-xt-relationalStore-dbExtension-archetype \
  -DdbType=YourDbType \
  -DlegendEngineVersion=<latest-version>
```

This will generate the five modules described above with the basic structure and placeholder implementations.

## Implementation Process

### 1. Define Protocol Classes

Protocol classes define the data structures for connection specifications and authentication strategies. If existing common specifications work for your database, you can reuse them. Otherwise, you'll need to create custom specifications.

#### Example: Using Existing Specifications

For many databases, you can use the existing `StaticDatasourceSpecification` and `UserNamePasswordAuthenticationStrategy`:

```java
// Using existing specifications
StaticDatasourceSpecification datasourceSpec = new StaticDatasourceSpecification();
datasourceSpec.host = "localhost";
datasourceSpec.port = 1433;
datasourceSpec.databaseName = "mydb";

UserNamePasswordAuthenticationStrategy authSpec = new UserNamePasswordAuthenticationStrategy();
authSpec.baseVaultReference = "myDbAccount.";
authSpec.userNameVaultReference = "user";
authSpec.passwordVaultReference = "password";

RelationalDatabaseConnection conn = new RelationalDatabaseConnection(
    datasourceSpec, 
    authSpec, 
    DatabaseType.SqlServer
);
```

#### Example: Creating Custom Specifications

If you need custom specifications, create new classes in the protocol module:

```java
public class CustomDatasourceSpecification extends DatasourceSpecification
{
    public String customProperty;
    
    @Override
    public <T> T accept(DatasourceSpecificationVisitor<T> visitor)
    {
        return visitor.visit(this);
    }
}
```

### 2. Implement SQL Generation Logic

SQL generation logic is implemented in Pure language in the `<dbType>-pure` module. This involves creating functions that generate SQL for different operations, handling database-specific syntax and optimizations.

#### Example: SQL Generation Function in Pure

```
function <<db.GenerationSpecification.Generator>> meta::relational::functions::sqlQueryToString::myDbType::myDbTypeGenerationSpecification(): meta::relational::functions::sqlQueryToString::GenerationSpecification[1]
{
    ^meta::relational::functions::sqlQueryToString::GenerationSpecification
    (
        name = 'MyDbType',
        
        // Function for generating SQL for a select statement
        selectSQLGenerator = {selectSQLQuery: SelectSQLQuery[1], topLevel: Boolean[1], config: GenerationConfig[1] |
            // Database-specific SQL generation logic
            let columns = $selectSQLQuery.columns->map(c | $c->processSelectColumn($config));
            let fromSQL = $selectSQLQuery.data->processJoinTreeNode($config);
            let whereClause = if($selectSQLQuery.filter->isEmpty(), | '', | ' where ' + $selectSQLQuery.filter->toOne()->processOperation($config));
            
            'select ' + $columns->joinStrings(', ') + ' from ' + $fromSQL + $whereClause;
        }
    )
}
```

### 3. Develop Grammar Support

Grammar support involves creating parsers and composers for the textual representation of connection specifications. This is implemented in the `<dbType>-grammar` module using ANTLR.

#### Example: Grammar Parser Extension

```java
public class MyDbTypeConnectionParserExtension implements IRelationalDatabaseConnectionParserExtension
{
    @Override
    public String getDbType()
    {
        return "MyDbType";
    }
    
    @Override
    public DatasourceSpecification parseDatasourceSpecification(String dbType, Consumer<String> walkerErrorListener, ParseTreeWalkerSourceInformation walkerSourceInformation)
    {
        // Parse datasource specification from text
        MyDbTypeDatasourceSpecification spec = new MyDbTypeDatasourceSpecification();
        // Parse properties
        return spec;
    }
    
    @Override
    public AuthenticationStrategy parseAuthenticationStrategy(String dbType, Consumer<String> walkerErrorListener, ParseTreeWalkerSourceInformation walkerSourceInformation)
    {
        // Parse authentication strategy from text
        return new UserNamePasswordAuthenticationStrategy();
    }
}
```

### 4. Implement Execution Logic

Execution logic involves creating connection managers and authentication flows for interacting with the database. This is implemented in the `<dbType>-execution` module.

#### Example: Connection Manager

```java
public class MyDbTypeManager extends RelationalDatabaseManager
{
    private static final String DRIVER_CLASSNAME = "com.mydb.jdbc.Driver";
    
    @Override
    public String buildURL(StaticDatasourceSpecification datasourceSpecification)
    {
        String host = datasourceSpecification.host;
        int port = datasourceSpecification.port;
        String databaseName = datasourceSpecification.databaseName;
        
        return "jdbc:mydb://" + host + ":" + port + "/" + databaseName;
    }
    
    @Override
    public String getDriverClassName()
    {
        return DRIVER_CLASSNAME;
    }
}
```

#### Example: Authentication Flow

```java
public class MyDbTypeStaticWithUserPasswordFlow implements DatabaseAuthenticationFlow<StaticDatasourceSpecification, UserNamePasswordAuthenticationStrategy>
{
    @Override
    public Class<StaticDatasourceSpecification> getDatasourceClass()
    {
        return StaticDatasourceSpecification.class;
    }
    
    @Override
    public Class<UserNamePasswordAuthenticationStrategy> getAuthenticationStrategyClass()
    {
        return UserNamePasswordAuthenticationStrategy.class;
    }
    
    @Override
    public DatabaseType getDatabaseType()
    {
        return DatabaseType.MyDbType;
    }
    
    @Override
    public Credential makeCredential(Identity identity, StaticDatasourceSpecification datasourceSpecification, UserNamePasswordAuthenticationStrategy authStrategy) throws Exception
    {
        String userNameVaultKey = authStrategy.baseVaultReference == null ? 
            authStrategy.userNameVaultReference : 
            authStrategy.baseVaultReference + authStrategy.userNameVaultReference;
            
        String passwordVaultKey = authStrategy.baseVaultReference == null ? 
            authStrategy.passwordVaultReference : 
            authStrategy.baseVaultReference + authStrategy.passwordVaultReference;
            
        String userName = Vault.INSTANCE.getValue(userNameVaultKey);
        String password = Vault.INSTANCE.getValue(passwordVaultKey);
        
        return new PlaintextUserPasswordCredential(userName, password);
    }
}
```

### 5. Write Integration Tests

Integration tests validate both SQL generation and execution against a test database instance. This is implemented in the `<dbType>-execution-tests` module.

#### Example: Test Database Setup

You can set up a test database instance using either a static connection or a dynamic connection with Docker:

```java
public class MyDbTypeUsingTestContainer implements DynamicTestConnection
{
    private final MyDbTypeContainer container = new MyDbTypeContainer("mydb:latest");
    private VaultImplementation vaultImplementation;
    
    @Override
    public DatabaseType getDatabaseType()
    {
        return DatabaseType.MyDbType;
    }
    
    @Override
    public void setup()
    {
        // Start container
        this.container.start();
        
        // Register vault with credentials
        Properties properties = new Properties();
        properties.put("myDbAccount.user", "testuser");
        properties.put("myDbAccount.password", "testpassword");
        this.vaultImplementation = new PropertiesVaultImplementation(properties);
        Vault.INSTANCE.registerImplementation(this.vaultImplementation);
    }
    
    @Override
    public RelationalDatabaseConnection getConnection()
    {
        StaticDatasourceSpecification spec = new StaticDatasourceSpecification();
        spec.host = this.container.getHost();
        spec.port = this.container.getMappedPort(1234); // Default port
        spec.databaseName = "testdb";
        
        UserNamePasswordAuthenticationStrategy authSpec = new UserNamePasswordAuthenticationStrategy();
        authSpec.baseVaultReference = "myDbAccount.";
        authSpec.userNameVaultReference = "user";
        authSpec.passwordVaultReference = "password";
        
        return new RelationalDatabaseConnection(spec, authSpec, DatabaseType.MyDbType);
    }
    
    @Override
    public void cleanup()
    {
        Vault.INSTANCE.unregisterImplementation(this.vaultImplementation);
        this.container.stop();
    }
}
```

#### Example: Connection Acquisition Test

```java
@Test
public void testConnectivity() throws Exception
{
    MyDbTypeUsingTestContainer testContainer = new MyDbTypeUsingTestContainer();
    testContainer.setup();
    try
    {
        RelationalDatabaseConnection systemUnderTest = testContainer.getConnection();
        Connection connection = this.connectionManagerSelector.getDatabaseConnection(
            (MutableList<CommonProfile>) null, 
            systemUnderTest
        );
        testConnection(connection, 1, "select 1");
    }
    finally
    {
        testContainer.cleanup();
    }
}
```

#### Example: SQL Generation Test

```java
@Test
public void testSelectQuery()
{
    String expected = "SELECT t0.ID, t0.NAME FROM PERSON t0 WHERE t0.AGE > 18";
    String actual = generateSql("Person.all()->filter(p | $p.age > 18)->project([p | $p.id, p | $p.name], ['ID', 'NAME'])");
    assertEquals(expected, actual);
}
```

## Testing Framework

Legend Engine provides a comprehensive testing framework for database connectors that validates both SQL generation and execution capabilities.

### Connectivity Tests

Connectivity tests verify that the connector can establish connections to the database with different authentication methods:

- `ExternalIntegration_TestConnectionAcquisitionWithFlowProvider_<DbType>`: Tests connection establishment using authentication flows

```java
@Test
public void testConnectivity() throws Exception
{
    // Get connection specification
    RelationalDatabaseConnection systemUnderTest = getTestConnection();
    
    // Acquire connection
    Connection connection = this.connectionManagerSelector.getDatabaseConnection(
        (MutableList<CommonProfile>) null, 
        systemUnderTest
    );
    
    // Test connection with a simple query
    testConnection(connection, 1, "select 1");
}
```

### SQL Generation and Execution Tests

SQL generation and execution tests validate that the connector generates correct SQL for various operations and can execute them against the database:

- `Test_Relational_DbSpecific_<DbType>_UsingPureClientTestSuite`: Tests SQL generation and execution using the standard test suite

```java
// In Pure language (sqlServerTestSuiteInvoker.pure)
function <<test.Test>> meta::relational::tests::sqlserver::testSuite::selectSubstring(): Boolean[1]
{
  meta::relational::tests::testSuite::selectSubstring(DatabaseType.SqlServer);
}

// Standard test suite (testSuite.pure)
function <<test.Test>> meta::relational::tests::testSuite::selectSubstring(dbType:DatabaseType[1]): Boolean[1]
{
  let result = execute(
    |Person.all()->project([p | $p.firstName->substring(0, 1)], ['initial']),
    meta::relational::tests::mapping::personMapping,
    ^Runtime(connectionStores = ^ConnectionStore(
      connection = ^TestDatabaseConnection(type = $dbType),
      store = meta::relational::tests::db
    ))
  );
  
  assertEquals(['J', 'S', 'E'], $result.rows->map(r | $r.values->at(0)));
}
```

### Pure IDE for SQL Generation Testing

Legend Engine provides a Pure IDE for testing SQL generation without executing against a database:

1. Launch the Pure IDE from the `<dbType>-pure` module
2. Open the SQL generation files in the IDE
3. Run tests with `^DbTestConfig(dbType=DatabaseType.<DbType>)` to see generated SQL
4. Use `^DbTestConfig(dbType=DatabaseType.<DbType>, expectedSql='')` to assert against expected SQL

### Test Categories

Test results are classified into five categories:

1. **Passed completely**: The test passes without any issues
2. **Ignored (feature not supported)**: The feature is not supported by the database and is explicitly marked as unsupported
3. **Partially ignored (feature works but deviates from standard)**: The feature works but has database-specific behavior that deviates from the standard
4. **Failure requiring fixes**: The test fails and requires fixes to the connector implementation
5. **Unknown status due to setup issues**: The test cannot be run due to setup or configuration issues

### Test Database Options

There are two approaches for setting up test databases:

1. **Static Test Connection**: Use an existing database instance
   - Define connection details in a JSON configuration file
   - Suitable for databases with complex setup requirements

2. **Dynamic Test Connection**: Launch a database container at test time
   - Use Docker containers via the TestContainers library
   - Automatically starts and stops the database for tests
   - Provides a clean environment for each test run

## Extension Points

Legend Engine provides several extension points for implementing database connectors. These extension points allow you to integrate your connector with the Legend Engine framework.

### RelationalDatabaseConnectionExtension

Extends connection capabilities for relational databases:
- Handles connection establishment
- Manages authentication
- Provides database metadata

```java
public class MyDbTypeRelationalDatabaseConnectionExtension implements RelationalDatabaseConnectionExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("Store", "Relational", "MyDbType");
    }
    
    @Override
    public DatabaseManager getDatabaseManager(RelationalDatabaseConnection connection)
    {
        if (connection.type == DatabaseType.MyDbType)
        {
            return new MyDbTypeManager();
        }
        return null;
    }
}
```

### DatabaseAuthenticationFlow

Implements authentication for database connections:
- Creates credentials for database connections
- Handles different authentication strategies
- Manages secure credential storage

```java
public class MyDbTypeStaticWithUserPasswordFlow implements DatabaseAuthenticationFlow<StaticDatasourceSpecification, UserNamePasswordAuthenticationStrategy>
{
    @Override
    public Class<StaticDatasourceSpecification> getDatasourceClass()
    {
        return StaticDatasourceSpecification.class;
    }
    
    @Override
    public Class<UserNamePasswordAuthenticationStrategy> getAuthenticationStrategyClass()
    {
        return UserNamePasswordAuthenticationStrategy.class;
    }
    
    @Override
    public DatabaseType getDatabaseType()
    {
        return DatabaseType.MyDbType;
    }
    
    @Override
    public Credential makeCredential(Identity identity, StaticDatasourceSpecification datasourceSpecification, UserNamePasswordAuthenticationStrategy authStrategy) throws Exception
    {
        // Create credentials from authentication strategy
        return new PlaintextUserPasswordCredential(username, password);
    }
}
```

### RelationalDatabaseCompilerExtension

Extends compiler capabilities for relational databases:
- Processes database-specific elements
- Validates database configuration
- Handles database-specific features

```java
public class MyDbTypeCompilerExtension implements RelationalDatabaseCompilerExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("Store", "Relational", "MyDbType");
    }
    
    @Override
    public Iterable<? extends Processor<?>> getExtraProcessors()
    {
        return Lists.immutable.with(
            Processor.newProcessor(
                MyDbTypeSpecificElement.class,
                // Process database-specific elements
            )
        );
    }
}
```

### RelationalDatabaseExecutionExtension

Extends execution capabilities for relational databases:
- Executes SQL queries
- Processes query results
- Handles database-specific execution features

```java
public class MyDbTypeExecutionExtension implements RelationalDatabaseExecutionExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("Store", "Relational", "MyDbType");
    }
    
    @Override
    public DatabaseManager getDatabaseManager(RelationalDatabaseConnection connection)
    {
        if (connection.type == DatabaseType.MyDbType)
        {
            return new MyDbTypeManager();
        }
        return null;
    }
    
    @Override
    public void executeStatement(Connection connection, String sql) throws SQLException
    {
        // Execute SQL statement with database-specific handling
    }
}
```

### Extension Registration

Extensions are registered using Java's ServiceLoader mechanism. Create a file in `META-INF/services` with the fully qualified name of the extension interface and list your implementation class:

```
# META-INF/services/org.finos.legend.engine.plan.execution.stores.relational.connection.RelationalDatabaseConnectionExtension
org.finos.legend.engine.plan.execution.stores.relational.connection.mydbtype.MyDbTypeRelationalDatabaseConnectionExtension
```

## Best Practices

### SQL Generation

1. **Follow the database's SQL dialect specifications**
   - Adhere to the database's SQL syntax and semantics
   - Consult the database's documentation for specific syntax requirements
   - Test generated SQL against the actual database

2. **Handle edge cases and special syntax**
   - Handle reserved keywords properly (e.g., by quoting identifiers)
   - Support database-specific functions and operators
   - Handle special data types and conversions

3. **Optimize queries for performance**
   - Implement database-specific query optimizations
   - Use appropriate indexing hints if supported
   - Consider query plan optimization for complex queries

4. **Provide clear error messages for unsupported features**
   - Mark unsupported features with messages starting with '[unsupported-api] '
   - Explain why a feature is unsupported and suggest alternatives
   - Document limitations in the connector documentation

5. **Document SQL generation limitations**
   - Clearly document features that are not supported
   - Explain database-specific behaviors that differ from standards
   - Provide examples of supported and unsupported queries

### Connection Management

1. **Implement connection pooling for performance**
   - Use connection pooling to improve performance
   - Configure appropriate pool sizes based on expected load
   - Implement proper connection validation and cleanup

2. **Handle connection timeouts and retries**
   - Set appropriate connection timeouts
   - Implement retry logic for transient failures
   - Provide clear error messages for connection failures

3. **Secure sensitive connection information**
   - Use vault references for credentials
   - Never hardcode credentials in the code
   - Support different authentication mechanisms

4. **Validate connection parameters**
   - Validate connection parameters before attempting to connect
   - Provide clear error messages for invalid parameters
   - Support default values for optional parameters

5. **Provide detailed error messages for connection failures**
   - Include relevant details in error messages
   - Log connection failures with appropriate context
   - Suggest possible solutions for common issues

### Testing

1. **Test with realistic database schemas**
   - Create test schemas that reflect real-world usage
   - Include tables with various data types and relationships
   - Test with both simple and complex schemas

2. **Include edge cases and complex queries**
   - Test queries with complex joins and subqueries
   - Include edge cases like empty results and null values
   - Test with large datasets to verify performance

3. **Validate error handling**
   - Test error scenarios like invalid queries and connection failures
   - Verify that error messages are clear and helpful
   - Ensure proper resource cleanup after errors

4. **Test with different authentication methods**
   - Test all supported authentication methods
   - Verify that authentication failures are handled properly
   - Test with both valid and invalid credentials

5. **Verify performance with large datasets**
   - Test with large datasets to verify performance
   - Measure query execution time and resource usage
   - Identify and address performance bottlenecks

### Handling Unsupported Features

- **Mark unsupported features**: Use messages starting with '[unsupported-api] ' to clearly indicate unsupported features
- **Document deviations**: Clearly document any deviations from standard behavior
- **Provide alternatives**: Suggest alternatives for unsupported features when possible
- **Gradual implementation**: Implement unsupported features over time, starting with the most commonly used ones

## Working with Pure Code

Legend Engine uses the Pure language for SQL generation logic. To work with Pure code:

1. **Launch the Pure IDE**: Run the `PureIDELight` class in the `<dbType>-pure` module
2. **Edit Pure files**: Modify the SQL generation functions in the Pure IDE
3. **Test SQL generation**: Use the test suite to verify SQL generation
4. **Debug SQL generation**: Use `^DbTestConfig(dbType=DatabaseType.<DbType>, expectedSql='')` to see generated SQL

## Complete Implementation Example

For a complete step-by-step guide on implementing a new database connector, refer to the [new connector tutorial](https://github.com/finos/legend-engine/blob/master/docs/store/extensions/Relational/new-connector-tutorial.md) in the Legend Engine documentation.

This tutorial walks through the entire process of implementing a new database connector, from setting up the project structure to testing the implementation against a real database instance.
