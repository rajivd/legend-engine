# Implementation Guidelines for Legend Engine

## Overview

This document provides comprehensive guidelines for implementing new features in Legend Engine. It covers best practices for code organization, error handling, performance considerations, testing strategies, and documentation requirements.

## Code Organization

### Module Structure

Legend Engine follows a modular architecture with clear separation of concerns. When implementing new features, follow this structure:

1. **Protocol Module**: Define data structures and interfaces
   - Contains POJOs representing the feature's data model
   - Defines serialization/deserialization support
   - Implements protocol extensions

2. **Pure Module**: Implement Pure language components
   - Contains Pure language functions and classes
   - Defines mappings between Pure and JSON representations
   - Implements feature-specific Pure extensions

3. **Grammar Module**: Implement parsing and composition
   - Contains ANTLR grammar for parsing textual representations
   - Implements parsers and composers
   - Handles error reporting and validation

4. **Compiler Module**: Implement compilation logic
   - Implements compiler extensions
   - Handles multi-pass compilation
   - Resolves references and dependencies

5. **Execution Module**: Implement execution logic
   - Implements execution plan generation
   - Handles runtime execution
   - Manages resources and connections

6. **Test Module**: Implement tests
   - Contains unit tests for each component
   - Implements integration tests
   - Provides test utilities and fixtures

### Extension Mechanisms

Legend Engine uses a flexible extension mechanism based on Java's ServiceLoader pattern. When implementing new features:

1. **Define Extension Interfaces**:
   - Extend appropriate base interfaces (`LegendExtension`, `CompilerExtension`, etc.)
   - Define clear contracts for extension points
   - Document extension capabilities

2. **Implement Extensions**:
   - Follow the single responsibility principle
   - Implement only the methods needed for your feature
   - Use composition over inheritance

3. **Register Extensions**:
   - Create META-INF/services files for ServiceLoader discovery
   - Register extensions in the appropriate loaders
   - Ensure proper initialization order

Example of extension registration:

```java
// META-INF/services/org.finos.legend.engine.language.pure.compiler.toPureGraph.extension.CompilerExtension
org.finos.legend.engine.language.pure.dsl.myfeature.compiler.MyFeatureCompilerExtension
```

### Package Naming Conventions

Follow these naming conventions for packages:

- `org.finos.legend.engine.protocol.pure.v1.model.packageableElement.myfeature`: Protocol classes
- `org.finos.legend.engine.language.pure.dsl.myfeature.grammar`: Grammar classes
- `org.finos.legend.engine.language.pure.dsl.myfeature.compiler`: Compiler classes
- `org.finos.legend.engine.language.pure.dsl.myfeature.execution`: Execution classes

### Class Naming Conventions

Follow these naming conventions for classes:

- `MyFeature`: Main feature class
- `MyFeatureCompilerExtension`: Compiler extension
- `MyFeatureParserExtension`: Parser extension
- `MyFeatureComposerExtension`: Composer extension
- `MyFeatureExecutionExtension`: Execution extension
- `MyFeatureTestSuite`: Test suite

## Multi-Pass Compilation

Legend Engine uses a multi-pass compilation process to handle dependencies and cross-references. When implementing compiler extensions:

### First Pass

The first pass is responsible for creating Pure objects and registering them in the model:

```java
@Override
public Iterable<? extends Processor<?>> getExtraProcessors()
{
    return Lists.immutable.with(
        Processor.newProcessor(
            MyFeature.class,
            Lists.fixedSize.with(DependencyClass.class),
            // First Pass - Create Pure objects
            (myFeature, context) -> {
                // Create Pure object
                Root_meta_myfeature_MyFeature pureMyFeature = new Root_meta_myfeature_MyFeature_Impl(
                    myFeature.name, 
                    null, 
                    context.pureModel.getClass("meta::myfeature::MyFeature")
                )
                ._name(myFeature.name);
                
                // Register in model
                context.pureModel.registerElement(pureMyFeature);
                
                return pureMyFeature;
            },
            // Second Pass
            (myFeature, context) -> { /* Second pass implementation */ },
            // Third Pass
            (myFeature, context) -> { /* Third pass implementation */ }
        )
    );
}
```

### Second Pass

The second pass resolves references to other elements:

```java
// Second Pass - Resolve references
(myFeature, context) -> {
    Root_meta_myfeature_MyFeature pureMyFeature = (Root_meta_myfeature_MyFeature) context.pureModel.getOrCreatePackageableElement(
        myFeature.getPath(),
        myFeature.sourceInformation,
        context.processorSupport
    );
    
    // Resolve references
    if (myFeature.reference != null)
    {
        pureMyFeature._reference(
            (Root_meta_reference_Reference) context.resolvePackageableElement(
                myFeature.reference.path,
                myFeature.reference.sourceInformation,
                Root_meta_reference_Reference.class
            )
        );
    }
}
```

### Third Pass

The third pass performs validation and cross-referencing:

```java
// Third Pass - Validate and cross-reference
(myFeature, context) -> {
    Root_meta_myfeature_MyFeature pureMyFeature = (Root_meta_myfeature_MyFeature) context.pureModel.getOrCreatePackageableElement(
        myFeature.getPath(),
        myFeature.sourceInformation,
        context.processorSupport
    );
    
    // Validate
    if (pureMyFeature._reference() == null)
    {
        throw new EngineException(
            "MyFeature must have a reference",
            myFeature.sourceInformation,
            EngineErrorType.COMPILATION
        );
    }
    
    // Cross-reference
    pureMyFeature._reference()._usedBy(pureMyFeature);
}
```

## Error Handling and Validation

### Error Types

Legend Engine defines several error types in `EngineErrorType`:

- `COMPILATION`: Errors during compilation
- `PARSER`: Errors during parsing
- `EXECUTION`: Errors during execution
- `AUTHENTICATION`: Authentication errors
- `CONFIGURATION`: Configuration errors

### Exception Handling

Follow these guidelines for exception handling:

1. **Use EngineException for user-facing errors**:
   - Include source information when available
   - Provide clear error messages
   - Specify the appropriate error type

```java
throw new EngineException(
    "Invalid reference: " + referencePath,
    sourceInformation,
    EngineErrorType.COMPILATION
);
```

2. **Use checked exceptions for recoverable errors**:
   - Document exception conditions
   - Provide recovery mechanisms
   - Handle exceptions at appropriate levels

3. **Use unchecked exceptions for programming errors**:
   - Use assertions for invariant violations
   - Document preconditions and postconditions
   - Fail fast for unrecoverable errors

### Validation Strategies

Implement validation at multiple levels:

1. **Protocol Validation**:
   - Validate data structures during deserialization
   - Use JSON Schema validation when appropriate
   - Provide clear error messages for invalid data

2. **Grammar Validation**:
   - Validate syntax during parsing
   - Report errors with source information
   - Provide suggestions for fixing errors

3. **Compiler Validation**:
   - Validate semantics during compilation
   - Check references and dependencies
   - Ensure type safety and consistency

4. **Execution Validation**:
   - Validate inputs before execution
   - Check preconditions and invariants
   - Validate results after execution

## Performance Considerations

### Memory Management

Follow these guidelines for memory management:

1. **Use appropriate data structures**:
   - Choose data structures based on access patterns
   - Consider memory overhead of collections
   - Use primitive collections when appropriate

2. **Avoid unnecessary object creation**:
   - Reuse objects when possible
   - Use builders for complex objects
   - Consider object pooling for frequently created objects

3. **Manage large data sets efficiently**:
   - Process data in chunks
   - Use streaming APIs for large collections
   - Release resources promptly

### Execution Optimization

Optimize execution for performance:

1. **Minimize database round trips**:
   - Batch database operations
   - Use appropriate fetch strategies
   - Optimize query generation

2. **Parallelize execution when appropriate**:
   - Use parallel streams for CPU-bound tasks
   - Use async/await for I/O-bound tasks
   - Consider thread safety and coordination

3. **Cache expensive computations**:
   - Use appropriate caching strategies
   - Consider cache invalidation
   - Document caching behavior

### Resource Management

Properly manage resources:

1. **Use try-with-resources for closeable resources**:
   - Ensure resources are closed properly
   - Handle exceptions during resource cleanup
   - Document resource lifecycle

```java
try (Connection connection = connectionManager.getConnection())
{
    // Use connection
}
```

2. **Release resources in finally blocks when necessary**:
   - Ensure resources are released even on exceptions
   - Handle exceptions during resource cleanup
   - Consider using cleanup utilities

3. **Monitor resource usage**:
   - Log resource acquisition and release
   - Track resource usage metrics
   - Implement resource limits and timeouts

## Testing Strategies

### Unit Testing

Write comprehensive unit tests:

1. **Test each component in isolation**:
   - Mock dependencies
   - Test edge cases
   - Verify error handling

2. **Use appropriate test frameworks**:
   - JUnit for Java tests
   - TestNG for parameterized tests
   - Mockito for mocking

3. **Follow test naming conventions**:
   - Use descriptive test names
   - Group related tests in test classes
   - Document test purpose and expectations

### Integration Testing

Write integration tests to verify component interactions:

1. **Test end-to-end workflows**:
   - Test with real dependencies
   - Verify system behavior
   - Test error handling and recovery

2. **Use test containers for external dependencies**:
   - Database containers
   - Service containers
   - Configure containers for testing

3. **Implement test utilities**:
   - Test data generators
   - Test fixtures
   - Test helpers

### Test Categories

Categorize tests for better organization:

1. **Unit Tests**: Test individual components in isolation
2. **Integration Tests**: Test component interactions
3. **Functional Tests**: Test end-to-end functionality
4. **Performance Tests**: Test performance characteristics
5. **Compatibility Tests**: Test compatibility with different environments

### Test Coverage

Ensure adequate test coverage:

1. **Code Coverage**: Aim for high code coverage
2. **Branch Coverage**: Test all branches and conditions
3. **Path Coverage**: Test all execution paths
4. **Mutation Testing**: Verify test quality with mutation testing

## Documentation Requirements

### Code Documentation

Document your code thoroughly:

1. **Javadoc for public APIs**:
   - Document purpose and behavior
   - Document parameters and return values
   - Document exceptions and error conditions

```java
/**
 * Compiles a MyFeature element into a Pure model.
 *
 * @param myFeature the MyFeature element to compile
 * @param context the compilation context
 * @return the compiled Pure object
 * @throws EngineException if compilation fails
 */
public Root_meta_myfeature_MyFeature compile(MyFeature myFeature, CompileContext context)
{
    // Implementation
}
```

2. **Comments for complex logic**:
   - Explain non-obvious algorithms
   - Document design decisions
   - Explain workarounds and limitations

3. **TODO and FIXME comments**:
   - Mark incomplete or temporary code
   - Document known issues
   - Provide context for future improvements

### Feature Documentation

Document your features:

1. **README files**:
   - Provide overview and purpose
   - Document usage examples
   - List limitations and known issues

2. **Architecture documentation**:
   - Document component interactions
   - Explain design decisions
   - Provide diagrams and visualizations

3. **User documentation**:
   - Provide user guides
   - Document configuration options
   - Include troubleshooting information

### Extension Documentation

Document your extensions:

1. **Extension points**:
   - Document available extension points
   - Explain extension capabilities
   - Provide extension examples

2. **Extension registration**:
   - Document registration process
   - Explain discovery mechanism
   - List required configuration

3. **Extension best practices**:
   - Document recommended patterns
   - Explain common pitfalls
   - Provide performance considerations

## Implementation Process

Follow this process when implementing new features:

1. **Design Phase**:
   - Define feature requirements
   - Design data structures and interfaces
   - Document design decisions

2. **Implementation Phase**:
   - Implement protocol classes
   - Implement grammar support
   - Implement compiler extensions
   - Implement execution logic
   - Implement tests

3. **Testing Phase**:
   - Run unit tests
   - Run integration tests
   - Verify performance
   - Test error handling

4. **Documentation Phase**:
   - Document code
   - Write feature documentation
   - Update extension documentation

5. **Review Phase**:
   - Review code quality
   - Verify test coverage
   - Ensure documentation completeness

## Example Implementation

Here's an example of implementing a new feature in Legend Engine:

### Protocol Classes

```java
package org.finos.legend.engine.protocol.pure.v1.model.packageableElement.myfeature;

import org.finos.legend.engine.protocol.pure.v1.model.packageableElement.PackageableElement;
import org.finos.legend.engine.protocol.pure.v1.model.packageableElement.PackageableElementVisitor;

public class MyFeature extends PackageableElement
{
    public String property;
    public MyFeatureReference reference;
    
    @Override
    public <T> T accept(PackageableElementVisitor<T> visitor)
    {
        return visitor.visit(this);
    }
}
```

### Compiler Extension

```java
package org.finos.legend.engine.language.pure.dsl.myfeature.compiler;

import org.eclipse.collections.api.list.MutableList;
import org.eclipse.collections.impl.factory.Lists;
import org.finos.legend.engine.language.pure.compiler.toPureGraph.CompileContext;
import org.finos.legend.engine.language.pure.compiler.toPureGraph.extension.CompilerExtension;
import org.finos.legend.engine.language.pure.compiler.toPureGraph.extension.Processor;
import org.finos.legend.engine.protocol.pure.v1.model.packageableElement.myfeature.MyFeature;

public class MyFeatureCompilerExtension implements CompilerExtension
{
    @Override
    public MutableList<String> group()
    {
        return Lists.mutable.with("PackageableElement", "MyFeature");
    }
    
    @Override
    public Iterable<? extends Processor<?>> getExtraProcessors()
    {
        return Lists.immutable.with(
            Processor.newProcessor(
                MyFeature.class,
                Lists.fixedSize.empty(),
                // First Pass
                (myFeature, context) -> {
                    // Implementation
                },
                // Second Pass
                (myFeature, context) -> {
                    // Implementation
                },
                // Third Pass
                (myFeature, context) -> {
                    // Implementation
                }
            )
        );
    }
}
```

### Parser Extension

```java
package org.finos.legend.engine.language.pure.grammar.from;

import org.finos.legend.engine.language.pure.grammar.from.antlr4.MyFeatureParserGrammar;
import org.finos.legend.engine.protocol.pure.v1.model.packageableElement.myfeature.MyFeature;

public class MyFeatureParserExtension implements IParserExtension
{
    @Override
    public Iterable<? extends IParser> parsers()
    {
        return Lists.immutable.with(
            new MyFeatureParser()
        );
    }
    
    private class MyFeatureParser implements IParser
    {
        @Override
        public String getName()
        {
            return "MyFeature";
        }
        
        @Override
        public PackageableElement parse(String code, ParseContext context)
        {
            // Parse code into MyFeature
            return myFeature;
        }
    }
}
```

### Test Implementation

```java
package org.finos.legend.engine.language.pure.dsl.myfeature.test;

import org.finos.legend.engine.language.pure.compiler.test.TestCompilationFromGrammar;
import org.junit.Test;

public class TestMyFeatureCompilation extends TestCompilationFromGrammar
{
    @Test
    public void testBasicMyFeature()
    {
        test("MyFeature myFeature::MyFeature1\n" +
             "{\n" +
             "  property: 'value';\n" +
             "  reference: myFeature::Reference1;\n" +
             "}\n");
    }
    
    @Test
    public void testInvalidMyFeature()
    {
        test("MyFeature myFeature::InvalidMyFeature\n" +
             "{\n" +
             "  property: 'value';\n" +
             "  // Missing reference\n" +
             "}\n",
             "MyFeature must have a reference");
    }
}
```

## Conclusion

Following these guidelines will help ensure that your implementations integrate seamlessly with the Legend Engine ecosystem, maintain high quality standards, and provide a consistent user experience. Remember that the Legend Engine is a complex system with many interacting components, so careful design, thorough testing, and comprehensive documentation are essential for successful feature implementation.
