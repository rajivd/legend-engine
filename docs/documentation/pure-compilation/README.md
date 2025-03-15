# Pure Code Compilation Pipeline

## Overview

The Legend Engine compiles Pure code into executable plans through a multi-stage pipeline. This document explains the complete process from Pure code to execution, including the key components, transformation steps, and how database-specific implementations are integrated.

The compilation pipeline consists of the following major stages:

1. **Parsing and Model Building**: Pure code is parsed into an abstract syntax tree and transformed into a PureModel
2. **Routing and Clustering**: Functions are routed and clustered to prepare them for execution plan generation
3. **Execution Plan Generation**: Routed functions are transformed into execution plans
4. **Platform Binding**: Execution plans are bound to a specific platform (e.g., Java)
5. **Transformation**: Plans are transformed into version-specific models that can be serialized and executed

## Key Components

The compilation pipeline involves several key components:

- **PureModel**: Manages the Pure model and compilation process
- **PlanGenerator**: Generates execution plans from Pure functions
- **JavaPlatformBinder**: Binds execution plans to the Java platform
- **PlanTransformer**: Transforms execution plans for different versions
- **JavaPlatformImplementation**: Represents Java-specific implementation details

## Compilation Flow Diagram

```
Pure Code → Parser → AST → PureModel → Router → Clustered Functions → Plan Generator → 
Execution Plan → Platform Binder → Platform-Specific Plan → Plan Transformer → 
Executable Code
```

## Parsing and Model Building

The first stage of the compilation pipeline involves parsing Pure code into an abstract syntax tree (AST) and building a PureModel.

### PureModel Construction

The `PureModel` class is responsible for building and managing the Pure model. It performs multiple passes over the parsed elements:

```java
// First pass: Create Pure objects
processFirstPass(element);

// Second pass: Resolve references
processSecondPass(element);

// Prerequisite pass: Process prerequisite elements
processPrerequisiteElementsPass(element);

// Third pass: Process cross-dependencies
processThirdPass(element);
```

Each pass serves a specific purpose:
- **First Pass**: Creates Pure objects from the parsed elements
- **Second Pass**: Resolves references between objects
- **Prerequisite Pass**: Processes prerequisite elements needed by other elements
- **Third Pass**: Processes cross-dependencies between elements

The `PureModel` class also maintains indexes for types, packages, and other elements to facilitate lookups during compilation:

```java
// Indexes for efficient lookups
final MutableMap<String, Type> typesIndex;
final MutableMap<String, GenericType> typesGenericTypeIndex;
private final MutableMap<String, org.finos.legend.pure.m3.coreinstance.meta.pure.metamodel.PackageableElement> packageableElementsIndex;
```

### Parsing Process

The parsing process converts Pure code text into an abstract syntax tree (AST) using the `PureGrammarParser`:

```java
// Parse Pure code into a model context
PureModelContextData contextData = PureGrammarParser.newInstance().parseModel(pureCode);

// Compile the model context into a PureModel
PureModel pureModel = Compiler.compile(contextData, DeploymentMode.TEST, Identity.getAnonymousIdentity().getName());
```

The parser handles different Pure language constructs, including:
- Class definitions
- Function definitions
- Mapping definitions
- Service definitions
- Connection definitions
- Runtime definitions

The resulting AST is then processed by the `PureModel` class to build a complete model that can be used for execution plan generation.

## Routing and Clustering

After building the Pure model, the next stage involves routing and clustering functions to prepare them for execution plan generation.

### Function Routing

The routing process is handled by the `meta::pure::router::routeFunction` function in `router_main.pure`. This function takes a function definition and routes it according to a routing strategy:

```pure
function meta::pure::router::routeFunction(
    f:FunctionDefinition<Any>[1], 
    routingStrategy:RoutingStrategy[1], 
    exeCtx: meta::pure::runtime::ExecutionContext[1], 
    extensions:meta::pure::extension::Extension[*]
):FunctionDefinition<Any>[1]
{
   // Enriching and clustering function expressions
   let enrichedExpressions = enrichFunctionExpressions(...);
   let clusters = $enrichedExpressions->map(exp| $exp->clusterFunctionExpressions(...));
   
   ^$f(expressionSequence = $clusters->toOneMany());
}
```

The routing process involves several key steps:

1. **Expression Enrichment**: The `enrichFunctionExpressions` function adds metadata to function expressions, including type information and execution context.

2. **Expression Clustering**: The `clusterFunctionExpressions` function groups related expressions based on their execution context and dependencies.

3. **Function Reconstruction**: The original function is reconstructed with the clustered expressions, preserving its signature but optimizing its execution.

### Clustering

Clustering groups related function expressions together based on their execution context. The `clusterFunctionExpressions` function analyzes function expressions and groups them according to:

- **Store Dependencies**: Expressions that operate on the same data store are clustered together to minimize data movement.
- **Execution Context**: Expressions with similar execution requirements are grouped together.
- **Function Characteristics**: Expressions with similar characteristics (e.g., pure functions vs. side-effecting functions) are clustered.

The result is a set of clustered value specifications that can be efficiently transformed into execution plans:

```pure
function meta::pure::router::clustering::clusterFunctionExpressions(
    vs:ValueSpecification[1], 
    routingStrategy:RoutingStrategy[1], 
    exeCtx:ExecutionContext[1], 
    extensions:Extension[*]
):ClusteredValueSpecification[1]
{
    // Analyze expression dependencies
    let dependencies = $vs->extractDependencies();
    
    // Group expressions by store
    let storeGroups = $dependencies->groupBy(d | $d.store);
    
    // Create clusters
    let clusters = $storeGroups->map(g | ^Cluster(
        expressions = $g.expressions,
        store = $g.store
    ));
    
    // Return clustered value specification
    ^ClusteredValueSpecification(
        clusters = $clusters,
        originalValueSpecification = $vs
    );
}
```

The clustering process is critical for optimizing execution, as it allows the execution plan generator to create efficient plans that minimize data movement and leverage database-specific optimizations.

## Execution Plan Generation

The execution plan generation stage transforms clustered functions into execution plans. This is handled by the `PlanGenerator` class.

### PlanGenerator

The `PlanGenerator` class provides methods to generate execution plans from function definitions:

```java
public static Root_meta_pure_executionPlan_ExecutionPlan generateExecutionPlanAsPure(
    FunctionDefinition<?> l, 
    Mapping mapping, 
    Root_meta_core_runtime_Runtime pureRuntime, 
    Root_meta_pure_runtime_ExecutionContext context, 
    PureModel pureModel, 
    PlanPlatform platform, 
    String planId, 
    RichIterable<? extends Root_meta_pure_extension_Extension> extensions)
{
    return generateExecutionPlanAsPure(l, mapping, pureRuntime, context, pureModel, platform, planId, false, extensions).getOne();
}
```

The actual plan generation is delegated to Pure functions in `executionPlan_generation.pure`:

```pure
function meta::pure::executionPlan::executionPlan(
    f:FunctionDefinition<Any>[1], 
    mapping:Mapping[1], 
    runtime:Runtime[1], 
    exeCtx:ExecutionContext[1], 
    extensions:Extension[*]
):ExecutionPlan[1]
{
    // Generate execution nodes
    let nodes = generateExecutionNodes($f, $mapping, $runtime, $exeCtx, $extensions);
    
    // Create execution plan
    ^ExecutionPlan(
        func = $f,
        mapping = $mapping,
        runtime = $runtime,
        executionNodes = $nodes,
        serializer = getDefaultSerializer()
    );
}
```

### Execution Nodes

The execution plan consists of execution nodes that represent the operations to be performed during execution. Each node has a specific type and implementation:

```pure
Class meta::pure::executionPlan::ExecutionNode
{
    // Unique identifier for the node
    nodeId: String[1];
    
    // Implementation details for the node
    implementation: PlatformImplementation[0..1];
    
    // Input and output for the node
    resultType: ResultType[0..1];
    
    // Child nodes that this node depends on
    executionNodes: ExecutionNode[*];
}
```

Common types of execution nodes include:

- **SequenceExecutionNode**: Executes a sequence of nodes in order
- **AllocationExecutionNode**: Allocates memory for results
- **FunctionExecutionNode**: Executes a specific function
- **RelationalExecutionNode**: Executes SQL against a relational database
- **StoreExecutionNode**: Executes operations against a specific store

### Plan Generation Process

The plan generation process involves several steps:

1. **Node Creation**: Create execution nodes for each clustered function expression
2. **Node Connection**: Connect nodes based on their dependencies
3. **Node Optimization**: Optimize nodes to improve execution performance
4. **Plan Assembly**: Assemble nodes into a complete execution plan

The resulting execution plan is a tree of execution nodes that can be executed to produce the desired result.

## Platform Binding

The platform binding stage transforms the execution plan into a platform-specific implementation. In Legend Engine, the primary platform is Java.

### JavaPlatformBinder

The `JavaPlatformBinder` class is responsible for binding execution plans to the Java platform:

```java
public Root_meta_pure_executionPlan_ExecutionPlan bindPlanToPlatform(
    Root_meta_pure_executionPlan_ExecutionPlan plan, 
    String planId, 
    PureModel pureModel, 
    RichIterable<? extends Root_meta_pure_extension_Extension> extensions)
{
    String platformId = core_java_platform_binding_legendJavaPlatformBinding_legendJavaPlatformBinding
        .Root_meta_pure_executionPlan_platformBinding_legendJava_legendJavaPlatformBindingId__String_1_(
            pureModel.getExecutionSupport()
        );
    return core_pure_executionPlan_executionPlan_generation
        .Root_meta_pure_executionPlan_generatePlatformCode_ExecutionPlan_1__String_1__PlatformBindingConfig_1__Extension_MANY__ExecutionPlan_1_(
            plan, platformId, getLegendJavaPlatformBindingConfig(planId), extensions, pureModel.getExecutionSupport()
        );
}
```

The platform binding process involves:

1. **Identifying the Platform**: The platform ID is retrieved from the Java platform binding.
2. **Creating a Platform Binding Configuration**: A configuration is created with platform-specific settings.
3. **Generating Platform-Specific Code**: Platform-specific code is generated for each execution node.
4. **Assembling the Platform-Specific Execution Plan**: The execution plan is updated with platform-specific implementation details.

### JavaPlatformImplementation

The result of platform binding is an execution plan with Java-specific implementation details, represented by the `JavaPlatformImplementation` class:

```java
public class JavaPlatformImplementation extends PlatformImplementation
{
    private List<JavaClass> classes;
    private String executionClassFullName;
    
    // Getters and setters
    
    @Override
    public String get_type()
    {
        return "java";
    }
}
```

The `JavaPlatformImplementation` contains:

- **Java Classes**: Generated Java classes that implement the execution logic
- **Execution Class Full Name**: The fully qualified name of the main execution class

### Platform Binding Configuration

The platform binding configuration specifies how the execution plan should be bound to the platform:

```pure
function meta::pure::executionPlan::platformBinding::legendJava::getLegendJavaPlatformBindingConfig(
    planId:String[0..1]
):PlatformBindingConfig[1]
{
    ^PlatformBindingConfig(
        platformId = legendJavaPlatformBindingId(),
        bindingDetail = ^JavaPlatformBindingConfig(
            planId = $planId
        )
    );
}
```

The configuration includes:

- **Platform ID**: Identifies the platform (Java)
- **Binding Detail**: Platform-specific binding details
- **Plan ID**: Optional identifier for the plan

The platform binding stage is crucial for transforming the abstract execution plan into a concrete implementation that can be executed on the Java platform.

## Transformation

The transformation stage converts the platform-specific execution plan into a version-specific model that can be serialized and executed.

### PlanTransformer

The `PlanTransformer` interface defines methods for transforming execution plans:

```java
public interface PlanTransformer
{
    boolean supports(String version);
    
    Object transformToVersionedModel(
        Root_meta_pure_executionPlan_ExecutionPlan purePlan, 
        String version, 
        RichIterable<? extends Root_meta_pure_extension_Extension> extensions, 
        ExecutionSupport executionSupport);
}
```

The `LegendPlanTransformers` class provides implementations for different versions of the protocol:

```java
public class LegendPlanTransformers
{
    public static Iterable<PlanTransformer> transformers()
    {
        return Lists.immutable.with(
            new DevPlanTransformer(),
            new V1PlanTransformer()
            // Additional transformers for other versions
        );
    }
}
```

### Transformation Process

The transformation process is handled by the `serializeToJSON` method in the `PlanGenerator` class:

```java
public static String serializeToJSON(
    Root_meta_pure_executionPlan_ExecutionPlan purePlan, 
    String clientVersion, 
    PureModel pureModel, 
    RichIterable<? extends Root_meta_pure_extension_Extension> extensions, 
    Iterable<? extends PlanTransformer> transformers)
{
    String cl = clientVersion == null ? PureClientVersions.production : clientVersion;
    MutableList<? extends PlanTransformer> handlers = Iterate.selectWith(
        transformers, PlanTransformer::supports, cl, Lists.mutable.empty());
    Assert.assertTrue(handlers.size() == 1, 
        () -> "Zero or more than one handler (" + handlers.size() + ") was found for protocol " + cl);
    Object transformed = handlers.get(0).transformToVersionedModel(
        purePlan, cl, extensions, pureModel.getExecutionSupport());
    return serializeToJSON(transformed, pureModel);
}
```

The transformation process involves:

1. **Selecting a Transformer**: A transformer is selected based on the client version.
2. **Transforming the Plan**: The selected transformer converts the execution plan into a version-specific model.
3. **Serializing the Model**: The transformed model is serialized to JSON.

### DevPlanTransformer

The `DevPlanTransformer` is a special transformer used for development and testing:

```java
public class DevPlanTransformer implements PlanTransformer
{
    @Override
    public boolean supports(String version)
    {
        return "vX_X_X".equals(version);
    }
    
    @Override
    public Root_meta_protocols_pure_vX_X_X_metamodel_executionPlan_ExecutionPlan transformToVersionedModel(
        Root_meta_pure_executionPlan_ExecutionPlan purePlan, 
        String version,
        RichIterable<? extends Root_meta_pure_extension_Extension> extensions, 
        ExecutionSupport executionSupport)
    {
        return core_pure_protocol_vX_X_X_transfers_executionPlan
            .Root_meta_protocols_pure_vX_X_X_transformation_fromPureGraph_executionPlan_transformPlan_ExecutionPlan_1__Extension_MANY__ExecutionPlan_1_(
                purePlan, extensions, executionSupport);
    }
}
```

Each transformer converts the execution plan into a model compatible with a specific version of the protocol. This allows for backward compatibility and versioning of execution plans, ensuring that clients using different versions of the protocol can still execute plans correctly.

## Database-Specific Implementations

Legend Engine supports multiple database platforms through database-specific implementations. These implementations are integrated into the compilation pipeline through extensions.

### Database Extensions

Database-specific extensions are defined in the `sqlQueryToString` folders for each database. For example, the DuckDB extension is defined in `duckdbExtension.pure`:

```pure
function meta::relational::functions::sqlQueryToString::duckdb::getDynaFunctionToSqlForDuckDB(): DynaFunctionToSql[*]
{
    [
        dynaFnToSql('abs',                   $allStates,            ^ToSql(format='abs(%s)')),
        dynaFnToSql('ceiling',               $allStates,            ^ToSql(format='ceiling(%s)')),
        dynaFnToSql('contains',              $allStates,            ^ToSql(format='contains(%s, %s)', transform={p:String[2]|$p})),
        dynaFnToSql('in',                    $allStates,            ^ToSql(format='%s in %s', transform={p:String[2] | if($p->at(1)->startsWith('(') && $p->at(1)->endsWith(')'), | $p, | [$p->at(0), ('(' + $p->at(1) + ')')])})),
        // Additional function mappings
    ]
}
```

These extensions map Pure functions to database-specific SQL implementations. Each database has its own extension that defines how Pure functions should be translated to SQL for that specific database.

The database extensions are loaded dynamically through the extension mechanism, allowing Legend Engine to support new databases without modifying the core codebase.

### PCT Framework

The Parameterized Compatibility Testing (PCT) framework tests functions across different database platforms. Functions requiring PCT support are marked with stereotypes:

- `<<PCT.adapter>>`: Indicates a database adapter implementation
- `<<PCT.function>>`: Indicates a function that should be tested across databases
- `<<PCT.platformOnly>>`: Indicates a function that is only available on specific platforms

The PCT framework generates reports that document known implementation gaps for each database platform. These reports are stored in the `target/classes/pct-reports/` directory and provide valuable information about which functions are supported by which databases.

Example of a PCT function definition:

```pure
function <<PCT.function>> meta::pure::functions::string::contains(source:String[1], target:String[1]):Boolean[1]
{
    $source->indexOf($target) != -1
}
```

Example of a PCT adapter implementation:

```pure
function <<PCT.adapter>> meta::relational::functions::sqlQueryToString::duckdb::contains(s:String[1], t:String[1]):String[1]
{
    'contains(' + $s + ', ' + $t + ')'
}
```

### Native Function Implementations

For functions that require custom Java implementations, Legend Engine provides a mechanism through the `org.finos.legend.pure.runtime.java.extension.functions.compiled.natives` package:

```java
package org.finos.legend.pure.runtime.java.extension.functions.compiled.natives.string;

import org.finos.legend.pure.runtime.java.compiled.generation.processors.natives.AbstractNativeFunctionGeneric;

public class Matches extends AbstractNativeFunctionGeneric
{
    public Matches()
    {
        super("matches_String_1__String_1__Boolean_1_", "meta::pure::functions::string::matches_String_1__String_1__Boolean_1_");
    }
    
    @Override
    public String build(String pattern)
    {
        return "new PurePattern(\"" + pattern + "\").matches";
    }
}
```

These native implementations provide optimized Java code for specific functions, which can be more efficient than the generated code. Native functions are registered through the extension mechanism and are automatically used when the corresponding Pure function is called.

The native function implementation mechanism allows Legend Engine to leverage the full power of Java for complex operations while still providing a consistent Pure interface to users.

## Complete Compilation Process

The complete compilation process from Pure code to execution involves all the stages described above. Here's a step-by-step overview:

1. **Parse Pure Code**: The Pure code is parsed into an abstract syntax tree.
   ```java
   PureModelContextData contextData = PureGrammarParser.newInstance().parseModel(pureCode);
   ```

2. **Build PureModel**: The AST is transformed into a PureModel through multiple passes:
   - First pass: Create Pure objects
   - Second pass: Resolve references
   - Prerequisite pass: Process prerequisite elements
   - Third pass: Process cross-dependencies
   ```java
   PureModel pureModel = Compiler.compile(contextData, DeploymentMode.TEST, Identity.getAnonymousIdentity().getName());
   ```

3. **Route Functions**: Functions are routed according to a routing strategy.
   ```pure
   let routedFunction = meta::pure::router::routeFunction($function, $routingStrategy, $executionContext, $extensions);
   ```

4. **Cluster Expressions**: Function expressions are clustered based on their execution context.
   ```pure
   let clusters = $enrichedExpressions->map(exp| $exp->clusterFunctionExpressions($routingStrategy, $executionContext, $extensions));
   ```

5. **Generate Execution Plan**: Clustered functions are transformed into an execution plan with execution nodes.
   ```java
   Root_meta_pure_executionPlan_ExecutionPlan plan = PlanGenerator.generateExecutionPlanAsPure(
       routedFunction, mapping, runtime, context, pureModel, platform, planId, extensions);
   ```

6. **Bind to Platform**: The execution plan is bound to a specific platform (Java) with platform-specific implementation details.
   ```java
   Root_meta_pure_executionPlan_ExecutionPlan boundPlan = javaPlatformBinder.bindPlanToPlatform(
       plan, planId, pureModel, extensions);
   ```

7. **Transform Plan**: The platform-specific plan is transformed into a version-specific model.
   ```java
   Object transformed = planTransformer.transformToVersionedModel(
       boundPlan, clientVersion, extensions, pureModel.getExecutionSupport());
   ```

8. **Serialize Plan**: The transformed plan is serialized to JSON for storage or transmission.
   ```java
   String serializedPlan = PlanGenerator.serializeToJSON(transformed, pureModel);
   ```

9. **Execute Plan**: The serialized plan is deserialized and executed by the PlanExecutor.
   ```java
   Result result = PlanExecutor.newPlanExecutor().execute(serializedPlan, inputParameters);
   ```

This process allows Pure code to be efficiently compiled and executed across different database platforms and versions of the protocol. The modular design of the compilation pipeline enables extensibility at each stage, allowing for the addition of new database platforms, function implementations, and execution strategies without modifying the core codebase.

The compilation process is designed to be efficient and scalable, with support for parallel processing of elements during model building and optimization of execution plans to minimize data movement and leverage database-specific optimizations. The result is a powerful and flexible execution engine that can handle a wide range of data transformation and integration scenarios.
