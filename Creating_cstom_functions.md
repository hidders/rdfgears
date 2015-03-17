# Introduction #


# RDF Gears documentation #

## RDF Gears plugin development ##

Third party programmers can implement new RGL functions in Java. First, we will discuss examples of how functions can be implemented. There are two ways to do this: either by extending the `SimplyTypedRGLFunction` or the `AtomicRGLFunction` class. Then we will discuss how such a function can be used in RDF Gears.

### Function lifecycle ###

The workflow execution consists of two phases. The first phase is the workflow loading. The second phase is the workflow execution. For every processor in a workflow, an instance of the appropriate RGLFunction is created. The lifecycle of such an instance is as follows:

  1. instantiation
  1. initialization
  1. typechecking
  1. execution
  1. termination

The first and the last step are JVM issues that do not need our attention: functions do not need an explicit constructor, and the garbage collection and/or program termination are of no concern now. We will now discuss the other phases: initialization, typechecking and execution phases. For these phases, appropriate methods must be implemented.

### Simple functions ###

The simplest way to implement an RGL function in Java is by extending the `SimplyTypedRGLFunction` class. This approach is good if your function will always receive and return the same input types, and the output must be null if any of the input values is null.

The three steps that must be implemented for SimplyTypedRGLFunction are the following:
  * instantiation: nothing to be done
  * initialization: to be implemented by the method `initialize(Map<String,String> config)`
  * typechecking: to be implemented by the method `getOutputType(TypeRow inputTypeRow)`
  * execution: to be implemented by the method `simpleExecute(ValueRow input)`
  * termination: nothing to be done

We will discuss this lifecycle by an example function that calculates the Jaro Similarity of two input strings. The implementation is given here (the implementation of the jaro method is omitted, for brevity).

```
public class JaroSimilarityFunction extends SimplyTypedRGLFunction {
	public static final String INPUT_1 = "s1";
	public static final String INPUT_2 = "s2";

	public void initialize(Map<String, String> config) {
		this.requireInputType(INPUT_1, RDFType.getInstance());
		this.requireInputType(INPUT_2, RDFType.getInstance());
	}

	public RGLType getOutputType() {
		return RDFType.getInstance();
	}

	public RGLValue simpleExecute(ValueRow inputRow) {
		RGLValue val1 = inputRow.get(INPUT_1);
		RGLValue val2 = inputRow.get(INPUT_2);
		if (!val1.isLiteral() || !val2.isLiteral())
			return ValueFactory.createNull("JaroSimilarity can only compare literals");

		String str1 = val1.asLiteral().getValueString();
		String str2 = val2.asLiteral().getValueString();
		double d = jaro(str1, str2);
		return ValueFactory.createLiteralDouble(d);
	}

	private static double jaro(String string1, String string2) {
		/* omitted */
	}
}
```

### Initialization ###

In the initialization phase, the initialize() function is called. This function fulfills two purpose. The first purpose of this method is the registration of function inputs and their expected types. This must happen for every function that has inputs (and most functions do, of course).

In the example, two inputs are registered, both of type RDFValue. As another example, consider a function that has an input named "bag" that accepts a value of type **Bag(T)**, for some element-type **T**. The output of the function is the size of the bag. The initialize method would call `requireInputType("bag", BagType.getInstance(new SuperTypePattern()))` to type the input port "bag" as a Bag containing elements of arbitrary type.

The second purpose of this method is the configuration of the function. Configuration parameters can be passed to the config map. For most SimplyTypedRGLFunction implementations, this will not be relevant and this method parameter should be ignored.

Remark 1. A `SimplyTypedRGLFunction` that does use the config parameter is the `SPARQLFunction` implementation. The ins and outs of RGL function configuration are explained in the section "Initialization" under "Advanced functions".

### Typechecking ###

The second call in our function is `getOutputType()`. It should just return the output type of the function, as in `return RDFType.getInstance()`. Note that the output type is fixed, once you implemented this function. If you want the output type to depend on the types of the input values, the `SimplyTypedRGLFunction` is not the right choice.

In some circumstances it is desirable to have the output type depend on the configuration parameters set in the initialize() method. This is possible. As an example, see the `SPARQLFunction` implementation in the repository.

### Execution ###

In the execution phase, the `simpleExecute()` function is called. It may be called multiple times, if its processor (or a workflow that uses that processor) is marked for iteration. Above we show an example implementation.

This method defines the semantics of the RGL function. For all implementations of `SimplyTypedRGLFunction`, it holds that the output is NULL if any of the input values is NULL (Note that this is about `RGL NULL` values. The Java null value can and may never occur in place of an RGL value).

This is guaranteed by the RGL engine, and thus the method implementation does not need to deal with NULL values. If any of the inputs of a `SimplyTypedRGLFunction` is `NULL`, then the `simpleExecute()` function is not called.

Instead, the first found `NULL` value is propagated to the output of the processor.
Assuming that all values are non-NULL (although bags and records may still contain NULL values), the `simpleExecute(ValueRow inputRow)` method is called. For all input names that were registered with requireInputType(), the named input can be fetched from the inputRow with the get() call.
Note that this function expects literals, and that the typechecking system guarantees only that the inputs are of type RDFValue.

For this reason, in order to be typesafe, the inputs must be checked with the isLiteral() call before they can be cast to a Literal with the `asLiteral()` method (For more details on the API for RGL values, study the function documentation in the Java source (or the javadoc generated therefrom). Of particular interest are the classes `ValueFactory`, `RGLValue`, `BagValue`, `RecordValue` and the related `FieldIndexMapFactory`, `LiteralValue`, `URIValue`, `GraphValue`).

In this section we also described how the `ValueFactory` can be used to create return values.

## Advanced functions ##

Another way to implement RGL functions is by extending the `AtomicRGLFunction`. This approach gives all the possibilities that `SimplyTypedRGLFunction` does. In addition, it allows polymorphism and the possibility to deal with `NULL` values manually. This comes at the cost of a slightly more complex interface. It is worth to study the implementation of the RGL core operators, as they provide good examples for polymorphic functions. Let us take a look at the `RecordProject` function in the next code snippet. We will discuss how each lifecycle phase is implemented for this function.

```
public class RecordProject extends NNRCFunction {
	public static final String CONFIGKEY_PROJECTFIELD = "projectField";
	public static final String INPUT_NAME = "record";
	private String fieldName; // the field we project
	
	public void initialize(Map<String, String> config) {
		this.requireInput(INPUT_NAME);
		this.fieldName = config.get(CONFIGKEY_PROJECTFIELD);
	}
	
	public RGLType getOutputType(TypeRow inputTypeRow) throws FunctionTypingException {
		RGLType actualInputType = inputTypeRow.get(INPUT_NAME);
		
		TypeRow typeRow = new TypeRow();
		typeRow.put(fieldName, new SuperTypePattern()); /* any type will do */
		RGLType requiredInputType = RecordType.getInstance(typeRow);
		
		if (actualInputType.isSubtypeOf(requiredInputType)){
			RGLType fieldType = ((RecordType)actualInputType).getFieldType(fieldName);
			assert(fieldType!=null);
			return fieldType;
		} else {
			throw new FunctionTypingException(INPUT_NAME, requiredInputType, actualInputType);
		}
	}
	
	public RGLValue execute(ValueRow input) {
		RGLValue r = input.get(INPUT_NAME);

		if (r.isNull()) return r; // propagate the error

		return r.asRecord().get(fieldName);

	}
}

```

### Initialization ###

The registration of inputs does not require specifying a type. As line 7 shows, the `requireInput()` call just receives a name. It registers a function input (and thus a processor port). Later, in the typechecking phase, the required type of the port will be registered.

An unrelated thing that happens in this example, is that a configuration parameter is used (line 8) to determine what field should be projected. This allows the `RecordProject` function to be configured for the projection of any record field. The `Map<String,String>` parameter that is passed is built by using the `<config param="key">value</config>` elements from the RGL workflow XML document.

### Typechecking ###

The function `getOutputType(TypeRow inputTypeRow)` receives a row over types. For all field names registered in the initialization phase, the inputTypeRow will contain a type. In the function the expected type is constructed. Note how a `SuperTypePattern()` is used to match any type for the record-field. Also note that the `fieldName` variable that was set as a configuration parameter is now used.

### Implementing streaming bags ###

The RGL data transformation of an RGL function can be implemented using the RGL API. It provides a conceptual view over the RGL values and does not in any way require understanding of the engine optimizations. For functions that return a bag value, the user merely needs to provide a mechanism to iterate over this bag once. So the implementation does not actually construct the bag, but does specify an iterator that defines it.

The bag implementations are thus agnostic to the way they are materialized by RDF Gears for multiple iterations. This makes implementation of third-party functions simple and allows them to seamlessly integrate with the typechecking mechanism and to benefit from the engine optimizations.


## Allowing dynamic function loading ##

When loading a workflow or when listing the available custom functions, the `ServiceLoader` class is used. If you distribute your custom function in a JAR file, a file `/META-INF/services/nl.tudelft.rdfgears.rgl.function.RGLFunction` must exist and it must contain a line with the full name (including package path) of your function.

An example is `nl.tudelft.rdfgears.rgl.function.standard.BagSize`. You can use any package for your function, as long as it extends RGLFunction. Note that the rdfgears-plugins package already contains this file and it will automatically be included in the jar built by `$mvn package`.