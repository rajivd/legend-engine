# Service Implementation

## Overview
Services in Legend Engine provide a way to expose functionality through well-defined interfaces. This document outlines the service implementation patterns, components, and best practices.

## Service Components

### Service Definition
Services are defined as packageable elements in the Pure language:

```
Service my::domain::PersonService
{
  pattern: '/persons';
  owners: ['Team A'];
  documentation: 'Service for person operations';
  execution: Single
  {
    function: |Person[*]| Person.all()->filter(p | $p.age > 18);
  }
}
```

### Service Execution Types

Legend Engine supports two primary service execution models:

### Single Execution
- Executes a single function with a defined set of parameters
- Implemented through `PureSingleExecution` in the protocol model
- Generates a `SingleExecutionPlan` for execution
- Example:

```
Service my::domain::PersonService
{
  pattern: '/persons';
  documentation: 'Service for person operations';
  execution: Single
  {
    function: |Person[*]| Person.all()->filter(p | $p.age > 18);
  }
}
```

### Multi Execution
- Supports multiple execution paths based on keys
- Implemented through `PureMultiExecution` in the protocol model
- Generates a `CompositeExecutionPlan` containing multiple execution plans
- Allows dynamic selection of execution path at runtime
- Example:

```
Service my::domain::PersonService
{
  pattern: '/persons';
  documentation: 'Service for person operations';
  execution: Multi
  {
    executions: [
      {
        key: 'getAll';
        mapping: my::mapping::PersonMapping;
        runtime: my::runtime::PersonRuntime;
        function: |Person[*]| Person.all();
      },
      {
        key: 'getAdults';
        mapping: my::mapping::PersonMapping;
        runtime: my::runtime::PersonRuntime;
        function: |Person[*]| Person.all()->filter(p | $p.age > 18);
      }
    ];
  }
}
```

### Service Parameters
- Define input parameters for service execution
- Support various data types and multiplicities
- Can be validated during execution
- Defined in the function signature
- Example:

```
Service my::domain::PersonService
{
  pattern: '/persons/{id}';
  documentation: 'Service for person operations';
  execution: Single
  {
    function: |id: String[1]| Person.all()->filter(p | $p.id == $id)->first();
  }
}
```

### Service Testing
- Test suites validate service behavior
- Test cases define input parameters and expected results
- Assertions verify service outputs

### Post-Validations
- Validate service execution results
- Define assertions for result validation
- Support complex validation logic

## Implementation Components

### ServiceCompilerExtension
The `ServiceCompilerExtensionImpl` class implements the `ServiceCompilerExtension` interface to provide service compilation capabilities:

- Processes service definitions through multi-pass compilation
- Handles service execution configuration
- Manages service tests and validations
- Validates parameter types and multiplicities

The compiler extension follows the standard multi-pass compilation process:

```java
public class ServiceCompilerExtensionImpl implements ServiceCompilerExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("PackageableElement", "Service");
    }

    @Override
    public Iterable<? extends Processor<?>> getExtraProcessors()
    {
        return Lists.immutable.with(
                Processor.newProcessor(
                        Service.class,
                        Lists.fixedSize.with(PackageableConnection.class, PackageableRuntime.class),
                        // First Pass - Create Pure objects
                        (service, context) -> processServiceFirstPass(service, context),
                        // Second Pass - Resolve references
                        (service, context) -> {
                            // Resolve references to other elements
                        },
                        // Third Pass - Validate and cross-reference
                        (service, context) -> {
                            Root_meta_legend_service_metamodel_Service pureService = 
                                (Root_meta_legend_service_metamodel_Service) context.pureModel.getPackageableElement(service);
                            
                            // Process execution
                            pureService._execution(processServiceExecution(service.execution, service, context));
                            
                            // Process tests
                            if (service.testSuites != null) {
                                // Process test suites
                            }
                            
                            // Validate post-validations
                            if (service.postValidations != null) {
                                // Process post-validations
                            }
                        }
                )
        );
    }
    
    // Implementation of first pass processing
    public Root_meta_legend_service_metamodel_Service processServiceFirstPass(Service service, CompileContext context)
    {
        return new Root_meta_legend_service_metamodel_Service_Impl(service.name, null, 
                context.pureModel.getClass("meta::legend::service::metamodel::Service"))
                ._name(service.name)
                ._stereotypes(ListIterate.collect(service.stereotypes, 
                    s -> context.resolveStereotype(s.profile, s.value, 
                        s.profileSourceInformation, s.sourceInformation)))
                ._taggedValues(ListIterate.collect(service.taggedValues, 
                    t -> new Root_meta_pure_metamodel_extension_TaggedValue_Impl("", null, 
                        context.pureModel.getClass("meta::pure::metamodel::extension::TaggedValue"))
                        ._tag(context.resolveTag(t.tag.profile, t.tag.value, 
                            t.tag.profileSourceInformation, t.tag.sourceInformation))
                        ._value(t.value)))
                ._pattern(service.pattern)
                ._owners(Lists.mutable.withAll(service.owners))
                ._documentation(service.documentation);
    }
}
```

### ServicePlanGenerator
The `ServicePlanGenerator` class is responsible for generating execution plans for services:

```java
public class ServicePlanGenerator
{
    public static ExecutionPlan generateServiceExecutionPlan(
            Service service, 
            Root_meta_pure_runtime_ExecutionContext context, 
            PureModel pureModel, 
            String clientVersion, 
            PlanPlatform platform, 
            RichIterable<? extends Root_meta_pure_extension_Extension> extensions, 
            Iterable<? extends PlanTransformer> transformers)
    {
        return generateServiceExecutionPlan(service, context, pureModel, clientVersion, platform, null, extensions, transformers);
    }
    
    // Implementation for single execution
    public static SingleExecutionPlan generateSingleExecutionPlan(
            PureSingleExecution singleExecution, 
            Root_meta_pure_runtime_ExecutionContext context, 
            PureModel pureModel, 
            String clientVersion, 
            PlanPlatform platform, 
            String planId, 
            RichIterable<? extends Root_meta_pure_extension_Extension> extensions, 
            Iterable<? extends PlanTransformer> transformers)
    {
        Mapping mapping = singleExecution.mapping != null ? pureModel.getMapping(singleExecution.mapping) : null;
        Root_meta_core_runtime_Runtime runtime = singleExecution.runtime != null ? 
                HelperRuntimeBuilder.buildPureRuntime(singleExecution.runtime, pureModel.getContext()) : null;
        LambdaFunction<?> lambda = HelperValueSpecificationBuilder.buildLambda(
                singleExecution.func.body, singleExecution.func.parameters, pureModel.getContext());
        
        return PlanGenerator.generateExecutionPlan(lambda, mapping, runtime, context, 
                pureModel, clientVersion, platform, planId, extensions, transformers);
    }
    
    // Implementation for multi execution
    public static CompositeExecutionPlan generateCompositeExecutionPlan(
            PureMultiExecution multiExecution, 
            Root_meta_pure_runtime_ExecutionContext context, 
            PureModel pureModel, 
            String clientVersion, 
            PlanPlatform platform, 
            String planId, 
            RichIterable<? extends Root_meta_pure_extension_Extension> extensions, 
            Iterable<? extends PlanTransformer> transformers)
    {
        // Generate a composite plan with multiple execution paths
        return generateCompositeExecutionPlan(null, multiExecution, context, pureModel, 
                clientVersion, platform, planId, extensions, transformers, null);
    }
}
```

### ServiceRunner
The `ServiceRunner` interface defines the contract for executing services:

```java
public interface ServiceRunner
{
    /**
     * Get the fully qualified service path (e.g. pack1::pack2::MyService)
     */
    String getServicePath();

    /**
     * Get plan executor information (connection pool details etc.)
     */
    PlanExecutorInfo getPlanExecutorInfo();

    /**
     * Get the list of variables the service expects
     */
    default List<ServiceVariable> getServiceVariables()
    {
        throw new UnsupportedOperationException("Not implemented");
    }

    /**
     * Run the service and return the serialized result
     */
    default String run(ServiceRunnerInput serviceRunnerInput)
    {
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        this.run(serviceRunnerInput, outputStream);
        return outputStream.toString();
    }

    /**
     * Run the service and write the serialized result to the passed output stream
     */
    void run(ServiceRunnerInput serviceRunnerInput, OutputStream outputStream);
}
```

### ServiceLegendPureCoreExtension
The `ServiceLegendPureCoreExtension` class implements the `FeatureLegendPureCoreExtension` interface to provide Pure language extensions for services:

```java
public class ServiceLegendPureCoreExtension implements FeatureLegendPureCoreExtension
{
    @Override
    public String functionFile()
    {
        return "core_service/service/extension.pure";
    }

    @Override
    public String functionSignature()
    {
        return "meta::legend::service::serviceExtension__Extension_1_";
    }

    @Override
    public MutableList<String> group()
    {
        return org.eclipse.collections.impl.factory.Lists.mutable.with("PackageableElement", "Service");
    }
}
```

### Service Testing Framework
- Executes service tests with defined parameters
- Validates test assertions
- Reports test results
- Implemented through the `ServiceTestRunner` class

## Plan Generation and Execution

### Plan Generation Process

The plan generation process for services follows these steps:

1. **Service Identification**: The service is identified by its path
2. **Execution Type Determination**: The system determines if it's a single or multi-execution service
3. **Plan Generation**:
   - For single execution, a `SingleExecutionPlan` is generated
   - For multi-execution, a `CompositeExecutionPlan` is generated with multiple execution paths
4. **Plan Transformation**: The generated plan is transformed using registered transformers
5. **Plan Optimization**: The plan is optimized for execution

#### Single Execution Plan Generation

```java
// Generate a single execution plan
SingleExecutionPlan plan = ServicePlanGenerator.generateSingleExecutionPlan(
    singleExecution,
    context,
    pureModel,
    clientVersion,
    platform,
    planId,
    extensions,
    transformers
);
```

#### Composite Execution Plan Generation

```java
// Generate a composite execution plan
CompositeExecutionPlan plan = ServicePlanGenerator.generateCompositeExecutionPlan(
    multiExecution,
    context,
    pureModel,
    clientVersion,
    platform,
    planId,
    extensions,
    transformers
);
```

### Plan Execution Process

The plan execution process follows these steps:

1. **Parameter Validation**: Service parameters are validated against the expected types and multiplicities
2. **Execution Context Setup**: The execution context is set up with runtime information
3. **Plan Execution**:
   - For single execution, the plan is executed directly
   - For multi-execution, the appropriate execution path is selected based on the key
4. **Result Serialization**: The execution result is serialized according to the specified format
5. **Post-Validation**: Optional post-validations are applied to the result

```java
// Execute a service
ServiceRunnerInput input = new ServiceRunnerInput()
    .withParameters(parameters)
    .withSerializationFormat(format);

String result = serviceRunner.run(input);
```

## Extension Points

### Service Compiler Extension
Extends the compiler to handle service definitions:
- Processes service elements through multi-pass compilation
- Validates service configuration
- Manages service tests and validations
- Example: `ServiceCompilerExtensionImpl`

### Service Execution Extension
Extends service execution capabilities:
- Adds custom execution logic
- Supports different execution environments
- Handles specialized parameter processing
- Example: `ServiceExecutionExtension`

```java
public interface ServiceExecutionExtension
{
    /**
     * Get the execution extension type
     */
    String getType();

    /**
     * Generate an execution plan for the service
     */
    ExecutionPlan generateExecutionPlan(Service service, 
                                       PureModel pureModel, 
                                       Root_meta_pure_runtime_ExecutionContext context, 
                                       String clientVersion);
}
```

### Service Test Runner Extension
Extends service testing capabilities:
- Adds custom test validation
- Supports different testing scenarios
- Provides specialized test reporting
- Example: `ServiceTestRunner`

```java
public class ServiceTestRunner implements TestRunner
{
    @Override
    public TestResult executeAtomicTest(Root_meta_pure_test_AtomicTest atomicTest, 
                                       PureModel pureModel, 
                                       PureModelContextData data)
    {
        // Execute service test
        ServiceTest test = (ServiceTest) atomicTest;
        
        // Set up test parameters
        // Execute service
        // Validate assertions
        
        return testResult;
    }
}
```

## Implementation Process

### 1. Define Service Protocol
- Create protocol classes for service elements
- Define service execution models
- Specify service test models

### 2. Implement Compiler Extension
- Process service elements
- Validate service configuration
- Handle service tests and validations

### 3. Implement Execution Logic
- Execute service functions
- Process parameters
- Return results

### 4. Implement Testing Framework
- Execute service tests
- Validate test assertions
- Report test results

### 5. Register Extensions
- Register compiler extensions
- Register execution extensions
- Register test runner extensions

## Service Parameters and Runtime Handling

### Parameter Definition

Service parameters are defined in the function signature of the service execution:

```
Service my::domain::PersonService
{
  pattern: '/persons/{id}';
  execution: Single
  {
    function: |id: String[1], age: Integer[0..1]| 
      Person.all()
        ->filter(p | $p.id == $id)
        ->filter(p | if($age->isEmpty(), true, $p.age >= $age->toOne()));
  }
}
```

Parameters can have:
- Different types (String, Integer, Boolean, custom types)
- Different multiplicities ([1], [0..1], [*])
- Default values
- Type constraints

### Parameter Validation

Parameters are validated during service execution:
1. **Type Validation**: Ensures parameters match the expected types
2. **Multiplicity Validation**: Ensures parameters have the correct cardinality
3. **Custom Validation**: Additional validation logic can be implemented

```java
// Validate service parameters
for (ServiceVariable variable : serviceVariables)
{
    Object value = parameters.get(variable.getName());
    
    // Validate type
    if (!variable.getType().isInstance(value))
    {
        throw new InvalidParameterException("Parameter '" + variable.getName() + 
            "' has invalid type. Expected: " + variable.getType().getName());
    }
    
    // Validate multiplicity
    if (value == null && variable.isRequired())
    {
        throw new InvalidParameterException("Required parameter '" + 
            variable.getName() + "' is missing");
    }
}
```

### Runtime Handling

Services can be configured with different runtimes:

```
Service my::domain::PersonService
{
  pattern: '/persons';
  execution: Single
  {
    mapping: my::mapping::PersonMapping;
    runtime: my::runtime::PersonRuntime;
    function: |Person[*]| Person.all();
  }
}
```

The runtime provides:
- Connection information for data sources
- Authentication details
- Execution context configuration
- Performance settings

Runtime handling involves:
1. **Runtime Resolution**: The runtime is resolved from the service definition
2. **Connection Acquisition**: Connections are acquired based on the runtime
3. **Context Setup**: The execution context is set up with runtime information
4. **Resource Management**: Connections are properly managed and released

## Error Handling and Validation

### Error Types

Legend Engine services handle various error types:
- **Compilation Errors**: Errors during service compilation
- **Validation Errors**: Errors during parameter validation
- **Execution Errors**: Errors during plan execution
- **Post-Validation Errors**: Errors during result validation

### Error Handling Strategies

1. **Early Validation**: Validate parameters before execution
2. **Structured Error Responses**: Return structured error information
3. **Detailed Error Messages**: Provide detailed error messages
4. **Source Information**: Include source information for errors
5. **Error Categorization**: Categorize errors for better handling

```java
try
{
    // Execute service
    return serviceRunner.run(input);
}
catch (InvalidParameterException e)
{
    // Handle parameter validation error
    return createErrorResponse(400, "Invalid Parameter", e.getMessage());
}
catch (ExecutionException e)
{
    // Handle execution error
    return createErrorResponse(500, "Execution Error", e.getMessage());
}
catch (Exception e)
{
    // Handle unexpected error
    return createErrorResponse(500, "Internal Error", "An unexpected error occurred");
}
```

### Post-Validation

Services can include post-validations to validate execution results:

```
Service my::domain::PersonService
{
  pattern: '/persons';
  execution: Single
  {
    function: |Person[*]| Person.all();
  }
  
  postValidations:
  [
    {
      description: 'Validate result is not empty';
      parameters: [result: Any[*]];
      assertions:
      [
        {
          id: 'not-empty';
          assertion: |result: Any[*]| !$result->isEmpty();
        }
      ];
    }
  ];
}
```

Post-validations are executed after the service execution and can:
- Validate result structure
- Validate result content
- Enforce business rules
- Ensure data quality

## Best Practices

### Service Design
1. Follow RESTful patterns for service endpoints
2. Use clear and consistent naming
3. Provide comprehensive documentation
4. Define appropriate ownership
5. Include meaningful test cases
6. Use appropriate execution type (Single vs. Multi)
7. Define clear parameter types and multiplicities

### Implementation Guidelines
1. Validate input parameters thoroughly
2. Handle errors gracefully with appropriate status codes
3. Provide meaningful error messages
4. Optimize execution for performance
5. Include source information for error reporting
6. Use post-validations for result validation
7. Implement proper resource management

### Testing Guidelines
1. Test both positive and negative cases
2. Include edge cases and boundary conditions
3. Validate error handling
4. Test with realistic data
5. Ensure backward compatibility
6. Test parameter validation
7. Test post-validations
8. Test different execution paths for multi-execution services
