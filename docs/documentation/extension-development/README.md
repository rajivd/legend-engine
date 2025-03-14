# Extension Development Pattern

## Overview
Legend Engine is designed with extensibility as a core principle. This document outlines the patterns and best practices for developing extensions to the Legend Engine.

## Extension Types

### Store Extensions
Add support for new data stores:
- Database connectors
- NoSQL stores
- Service integrations
- Custom data sources

### External Format Extensions
Add support for data format schemas:
- JSON Schema
- XSD
- Avro
- Protobuf
- Custom formats

### Service Extensions
Extend service capabilities:
- Service execution
- Service testing
- Service validation
- Service generation

### Generation Extensions
Add code generation capabilities:
- Language-specific code generation
- Documentation generation
- Artifact generation

## Extension Development Process

### 1. Identify Extension Point
- Determine which aspect of Legend Engine to extend
- Identify the appropriate extension interface
- Understand the extension's responsibilities

The main extension interfaces in Legend Engine include:

```java
// Base extension interface
public interface LegendExtension
{
    default MutableList<String> group()
    {
        return Lists.mutable.empty();
    }

    default String type()
    {
        return "Unknown Type " + this.getClass().getName();
    }

    default MutableList<String> typeGroup()
    {
        return Lists.mutable.empty();
    }
}

// Language-specific extensions
public interface LegendLanguageExtension extends LegendExtension
{
    @Override
    default MutableList<String> typeGroup()
    {
        return Lists.mutable.with("Lang");
    }
}

// Plan generation and execution extensions
public interface LegendPlanExtension extends LegendExtension
{
    @Override
    default MutableList<String> typeGroup()
    {
        return Lists.mutable.with("Plan");
    }
}

// Compiler extensions
public interface CompilerExtension
{
    default MutableList<String> group()
    {
        return Lists.mutable.empty();
    }
    
    default Iterable<? extends Processor<?>> getExtraProcessors()
    {
        return Lists.immutable.empty();
    }
    
    // Additional methods for specific element types
}
```

### 2. Implement Extension Interface
- Implement the required interface methods
- Follow the extension pattern for the specific type
- Ensure proper error handling and validation

#### Multi-Pass Compilation Process

For compiler extensions, Legend Engine uses a multi-pass compilation process:

```java
public static <T extends PackageableElement> Processor<T> newProcessor(
   Class<T> elementClass,
   Collection<? extends Class<? extends PackageableElement>> prerequisiteClasses,
   BiFunction<? super T, CompileContext, org.finos.legend.pure.m3.coreinstance.meta.pure.metamodel.PackageableElement> firstPass,
   BiConsumer<? super T, CompileContext> secondPass,
   BiConsumer<? super T, CompileContext> thirdPass,
   BiFunction<? super T, CompileContext, RichIterable<? extends org.finos.legend.pure.m3.coreinstance.meta.pure.metamodel.PackageableElement>> prerequisiteElementsPass)
```

1. **First Pass**:
   - Create Pure (M3) objects and register them in the Pure graph
   - Set primitive values in the Pure (M3) objects
   - **MUST NOT** reference other elements in the Pure graph

2. **Second Pass**:
   - Resolve content of its own element and references to other elements
   - **MUST NOT** introspect content of other elements or check validity/correctness

3. **Third Pass**:
   - Resolve cross-dependencies
   - Introspect other elements
   - Perform validation

4. **Prerequisite Elements Pass**:
   - Return prerequisite elements that your element depends on
   - Used for dependency sorting to ensure correct compilation order

#### Example: Service Compiler Extension Implementation

Here's a simplified example of a Service Compiler Extension implementation:

```java
public class ServiceCompilerExtensionImpl implements ServiceCompilerExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("PackageableElement", "Service");
    }

    @Override
    public CompilerExtension build()
    {
        return new ServiceCompilerExtensionImpl();
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

### 3. Register Extension
- Register the extension with ServiceLoader
- Create META-INF/services entries
- Ensure unique extension identifiers

#### ServiceLoader Registration

To register your extension, create a file in the `META-INF/services` directory with the fully qualified name of the extension interface, containing the fully qualified name of your implementation:

```
# META-INF/services/org.finos.legend.engine.language.pure.compiler.toPureGraph.extension.CompilerExtension
org.finos.legend.engine.language.pure.dsl.service.compiler.toPureGraph.ServiceCompilerExtensionImpl
```

#### Extension Loader Implementation

Extensions are typically loaded using a pattern like this:

```java
public class MyExtensionLoader
{
    public static List<MyExtension> extensions()
    {
        List<MyExtension> extensions = Lists.mutable.withAll(ServiceLoader.load(MyExtension.class));
        // Validate extensions (e.g., check for duplicate keys)
        Set<String> extensionKeys = Sets.mutable.empty();
        for (MyExtension extension : extensions)
        {
            if (!extensionKeys.add(extension.getKey()))
            {
                throw new EngineException("Extension keys must be unique. Found duplicate key: '" 
                    + extension.getKey() + "'");
            }
        }
        return extensions;
    }
}
```

### 4. Test Extension
- Write comprehensive tests
- Validate functionality
- Ensure compatibility with existing features

### 5. Document Extension
- Document extension capabilities
- Provide usage examples
- Include configuration details

## Module Structure Pattern

Extensions typically follow a consistent module structure pattern:

### Protocol
- Defines data models and interfaces
- Contains POJOs representing domain concepts
- Example: `<module>-protocol`

### Pure
- Contains Pure language implementations
- Defines metamodels and transformations
- Example: `<module>-pure`

### Grammar
- Provides parsing and generation of textual DSLs
- Converts between text and object representations
- Example: `<module>-grammar`

### Compiler
- Implements compilation logic
- Transforms protocol objects to Pure model
- Example: `<module>-compiler`

### Execution
- Implements runtime execution
- Handles data processing and transformation
- Example: `<module>-execution`

### Tests
- Contains unit and integration tests
- Validates functionality
- Example: `<module>-tests`

## Example: Relational Store Extension

As described in the Legend Engine documentation:

1. **\<dbType\>-protocol**: Defines POJOs for connector-specific data source and authentication strategy specifications
2. **\<dbType\>-pure**: Contains DB-specific SQL generation logic in Pure language
3. **\<dbType\>-grammar**: Contains ANTLR-based code for bi-directional conversion between textual DSLs and POJOs
4. **\<dbType\>-execution**: Defines driver and authentication logic
5. **\<dbType\>-execution-tests**: Contains integration tests for connectivity and execution

## Implementation Patterns for Different Extension Types

### 1. Compiler Extension Implementation

Compiler extensions add support for new packageable elements and their compilation:

```java
public class MyCompilerExtension implements CompilerExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("MyExtension");
    }

    @Override
    public Iterable<? extends Processor<?>> getExtraProcessors()
    {
        return Lists.immutable.with(
            Processor.newProcessor(
                MyElement.class,
                Lists.fixedSize.with(DependencyElement1.class, DependencyElement2.class),
                (element, context) -> processFirstPass(element, context),
                (element, context) -> processSecondPass(element, context),
                (element, context) -> processThirdPass(element, context),
                (element, context) -> getPrerequisiteElements(element, context)
            )
        );
    }
    
    // First pass implementation
    private Root_meta_pure_metamodel_PackageableElement processFirstPass(MyElement element, CompileContext context)
    {
        // Create Pure object and set primitive values
        return new Root_meta_my_extension_MyElement_Impl(element.name, null, 
                context.pureModel.getClass("meta::my::extension::MyElement"))
                ._name(element.name)
                ._documentation(element.documentation);
    }
    
    // Second pass implementation
    private void processSecondPass(MyElement element, CompileContext context)
    {
        // Resolve references to other elements
        Root_meta_my_extension_MyElement pureElement = 
            (Root_meta_my_extension_MyElement) context.pureModel.getPackageableElement(element);
        
        if (element.reference != null)
        {
            pureElement._reference(context.resolvePackageableElement(element.reference));
        }
    }
    
    // Third pass implementation
    private void processThirdPass(MyElement element, CompileContext context)
    {
        // Validate and cross-reference
        Root_meta_my_extension_MyElement pureElement = 
            (Root_meta_my_extension_MyElement) context.pureModel.getPackageableElement(element);
        
        // Validate element properties
        if (pureElement._reference() == null)
        {
            throw new EngineException("Reference cannot be null", element.sourceInformation);
        }
    }
    
    // Prerequisite elements implementation
    private RichIterable<? extends org.finos.legend.pure.m3.coreinstance.meta.pure.metamodel.PackageableElement> 
        getPrerequisiteElements(MyElement element, CompileContext context)
    {
        // Return elements that this element depends on
        return Lists.immutable.empty();
    }
}
```

### 2. Language Extension Implementation

Language extensions add support for new language features:

```java
public class MyLanguageExtension implements LegendLanguageExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("MyLanguage");
    }
    
    @Override
    public String type()
    {
        return "MyLanguageExtension";
    }
    
    // Grammar parser implementation
    public MyElement parse(String text)
    {
        // Parse text into protocol model
        return new MyElement();
    }
    
    // Grammar composer implementation
    public String compose(MyElement element)
    {
        // Compose protocol model into text
        return "MyElement " + element.name;
    }
}
```

### 3. Plan Extension Implementation

Plan extensions add support for new execution plan generation and transformation:

```java
public class MyPlanExtension implements LegendPlanExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("MyPlan");
    }
    
    @Override
    public String type()
    {
        return "MyPlanExtension";
    }
    
    // Plan transformer implementation
    public ExecutionPlan transform(ExecutionPlan plan)
    {
        // Transform execution plan
        return plan;
    }
    
    // Plan generator implementation
    public ExecutionPlan generate(MyElement element)
    {
        // Generate execution plan
        return new ExecutionPlan();
    }
}
```

### 4. Extension Loader Implementation

Extensions are loaded using ServiceLoader:

```java
public class MyExtensionLoader
{
    public static List<MyExtension> extensions()
    {
        List<MyExtension> extensions = Lists.mutable.withAll(ServiceLoader.load(MyExtension.class));
        
        // Validate extensions
        Set<String> extensionKeys = Sets.mutable.empty();
        for (MyExtension extension : extensions)
        {
            if (!extensionKeys.add(extension.getKey()))
            {
                throw new EngineException("Extension keys must be unique. Found duplicate key: '" 
                    + extension.getKey() + "'");
            }
        }
        
        return extensions;
    }
}
```

### 5. Test Runner Extension Implementation

Test runner extensions add support for testing new element types:

```java
public class MyTestRunner implements TestRunner
{
    @Override
    public TestResult executeAtomicTest(Root_meta_pure_test_AtomicTest atomicTest, 
                                       PureModel pureModel, 
                                       PureModelContextData data)
    {
        // Execute atomic test
        MyTest test = (MyTest) atomicTest;
        
        try
        {
            // Test execution logic
            return new TestResult(test._id(), TestExecutionStatus.PASS, null);
        }
        catch (Exception e)
        {
            return new TestResult(test._id(), TestExecutionStatus.FAIL, e.getMessage());
        }
    }
    
    @Override
    public List<TestResult> executeTestSuite(Root_meta_pure_test_TestSuite testSuite, 
                                           List<String> testIds, 
                                           PureModel pureModel, 
                                           PureModelContextData data)
    {
        // Execute test suite
        List<TestResult> results = new ArrayList<>();
        
        for (Root_meta_pure_test_AtomicTest test : testSuite._tests())
        {
            if (testIds.isEmpty() || testIds.contains(test._id()))
            {
                results.add(executeAtomicTest(test, pureModel, data));
            }
        }
        
        return results;
    }
}
```

## Best Practices

### Design Principles
1. **Follow the single responsibility principle**
   - Each extension should focus on a specific aspect of functionality
   - Break complex extensions into smaller, focused extensions
   - Separate concerns between protocol, grammar, compiler, and execution

2. **Ensure proper error handling and validation**
   - Provide clear, actionable error messages
   - Include source information in error messages
   - Validate inputs early and thoroughly
   - Use `EngineException` with appropriate error types

3. **Maintain backward compatibility**
   - Avoid breaking changes to existing extensions
   - Use versioning for significant changes
   - Provide migration paths for users

4. **Follow existing patterns in the codebase**
   - Study similar extensions for guidance
   - Maintain consistent naming conventions
   - Use established design patterns

5. **Use functional interfaces for flexibility**
   - Leverage Java 8+ functional interfaces
   - Use lambdas for concise code
   - Enable composition of behavior

### Implementation Guidelines

1. **Multi-Pass Compilation**
   - Respect the separation of concerns in each pass
   - First pass: Create objects and set primitive values only
   - Second pass: Resolve references to other elements
   - Third pass: Validate and cross-reference

2. **Dependency Management**
   - Clearly specify prerequisite classes
   - Identify prerequisite elements
   - Avoid circular dependencies

3. **Error Handling**
   - Use `EngineException` for compilation errors
   - Include source information for better error reporting
   - Use appropriate error types from `EngineErrorType`
   - Example:
     ```java
     throw new EngineException("Invalid reference: " + reference, 
                              sourceInformation, 
                              EngineErrorType.COMPILATION);
     ```

4. **Extension Registration**
   - Use ServiceLoader for extension discovery
   - Create proper META-INF/services entries
   - Validate extension uniqueness
   - Example:
     ```
     # META-INF/services/org.finos.legend.engine.language.pure.compiler.toPureGraph.extension.CompilerExtension
     org.finos.legend.engine.language.pure.dsl.myextension.compiler.toPureGraph.MyCompilerExtensionImpl
     ```

5. **Module Structure**
   - Follow the established module structure pattern:
     - `<module>-protocol`: Data models and interfaces
     - `<module>-pure`: Pure language implementations
     - `<module>-grammar`: Parsing and generation of DSLs
     - `<module>-compiler`: Compilation logic
     - `<module>-execution`: Runtime execution
     - `<module>-tests`: Unit and integration tests

### Testing Guidelines

1. **Unit Testing**
   - Test each component in isolation
   - Mock dependencies
   - Test edge cases and error conditions
   - Example:
     ```java
     @Test
     public void testFirstPassCompilation()
     {
         MyElement element = new MyElement();
         element.name = "test";
         
         CompileContext context = mock(CompileContext.class);
         when(context.pureModel.getClass(anyString())).thenReturn(mockClass);
         
         Root_meta_my_extension_MyElement result = 
             myExtension.processFirstPass(element, context);
         
         assertEquals("test", result._name());
     }
     ```

2. **Grammar Testing**
   - Test both parsing and composition
   - Test round-trip conversions
   - Test error cases
   - Example:
     ```java
     @Test
     public void testGrammarRoundtrip()
     {
         String text = "MyElement test { property: value }";
         MyElement element = myExtension.parse(text);
         String composed = myExtension.compose(element);
         assertEquals(text, composed);
     }
     ```

3. **Integration Testing**
   - Test end-to-end functionality
   - Test interaction with other extensions
   - Test with real data
   - Example:
     ```java
     @Test
     public void testEndToEndCompilation()
     {
         String code = "MyElement test { property: value }";
         PureModelContextData data = PureGrammarParser.parse(code);
         PureModel model = new PureModel(data, null, null);
         
         Root_meta_my_extension_MyElement element = 
             (Root_meta_my_extension_MyElement) model.getPackageableElement("test");
         
         assertNotNull(element);
         assertEquals("test", element._name());
         assertEquals("value", element._property());
     }
     ```

4. **Performance Testing**
   - Test with large inputs
   - Measure compilation time
   - Identify bottlenecks
   - Example:
     ```java
     @Test
     public void testCompilationPerformance()
     {
         String code = generateLargeInput();
         long start = System.currentTimeMillis();
         PureModelContextData data = PureGrammarParser.parse(code);
         PureModel model = new PureModel(data, null, null);
         long end = System.currentTimeMillis();
         
         assertTrue("Compilation took too long: " + (end - start) + "ms", 
                   end - start < 5000);
     }
     ```

5. **Compatibility Testing**
   - Test with existing extensions
   - Test with different versions
   - Test with different configurations
   - Example:
     ```java
     @Test
     public void testCompatibilityWithOtherExtensions()
     {
         String code = "MyElement test { reference: otherElement }";
         String otherCode = "OtherElement otherElement {}";
         
         PureModelContextData data = PureGrammarParser.parse(code + "\n" + otherCode);
         PureModel model = new PureModel(data, null, null);
         
         Root_meta_my_extension_MyElement element = 
             (Root_meta_my_extension_MyElement) model.getPackageableElement("test");
         
         assertNotNull(element._reference());
         assertEquals("otherElement", element._reference()._name());
     }
     ```
