# Introduction #

In this tutorial we demonstrate how to create and deploy U-Sem plug-ins on the OSGi-enabled version of U-Sem (available [here](https://code.google.com/p/rdfgears/source/checkout?repo=u-sem-osgi))

# Requirements #

In order to create a plugin for U-Sem you need:

  * U-Sem - installed and running
  * Eclipse IDE (>= 3.7 Indigo). We recommend using the _Eclipse for RCP and RAP Developers_ [package](http://www.eclipse.org/downloads)
  * U-Sem SDK for Eclipse, available in the sdk directory of the [OSGi-enabled U-Sem](https://code.google.com/p/rdfgears/source/checkout?repo=u-sem-osgi).

# Setting up the IDE #

Before being able to start developing plug-ins for U-Sem we have to set up the IDE. This process consists of two main steps:

## Import the U-Sem core project ##

Plug-ins normally depend on functionality provided by the U-Sem project and therefore, in order to be able to compile them correctly we need to import the U-Sem project into the IDE. Currently, all dependencies are contained in the **rdfgears-core** sub-project, and thus it is enough to import only. For more information on importing existing projects in eclipse read the _Importing existing projects_ section in the Eclipse documentation.

## Install the U-Sem SDK ##

U-Sem comes with SDK that helps engineers to easily create plug-ins. The SDK provides project template which creates and automatically makes all necessary configurations for a plug-in project.

In order to install it, you need to import the project `sdk/nl.tudelft.wis.usem.plugin.templates` into Eclipse; the project contains all the needed dependencies, including a dependency on the rddgears-core project. The latter should then be also imported in Eclipse, or this dependency should be changed to a dependency on the file rdfgears-core.jar.

Compile the project and then export it using the option `deployable plug-ins and fragments`.

Then copy the SDK jar file to the `dropins` folder of Eclipse and start/restart the IDE.

# Create plug-in #

## Step 1 - Create plug-in project ##

In order to create new plug-in project for U-Sem the following steps have to be followed:

  * Open the **New plug-in wizard** in Eclipse - it can be found under File/New/Plug-in Project
  * Fill in all standard configuration details (such as name of the project, vendor name) and click next until the _Templates_ section appears
  * Select the `U-SEM Plugin` template and click next.
  * The plug-in template creates sample RDFGears function and workflow which are wired to work together. You need to specify their names.
  * Click Finish and the plug-in is automatically created and configured.

## Step 2 - Modify the created artefacts manually ##

Engineers will need to adapt the skeleton created by the Wizard to their own needs. In case the function only requires one input, the modifications are straightfoward. We will show how to expand the function to accept two inputs. For more information, please have a look at the [specification](https://code.google.com/p/rdfgears/wiki/Creating_cstom_functions).

In the following we show the code for a function that has two integer inputs and provide as output their multiplication.

```
/**
 * A function that multiplies its inputs
 * 
 */
public class Multiply extends SimplyTypedRGLFunction {

	/**
	 * The name of the input field providing the statement that will be analysed
	 */
	public static final String INPUT1 = "input1";
	public static final String INPUT2 = "input2";


	public Multiply() {
		this.requireInputType(INPUT1, RDFType.getInstance());
		this.requireInputType(INPUT2, RDFType.getInstance());
	}

	@Override
	public RGLType getOutputType() {
		return RDFType.getInstance();
	}

	@Override
	public RGLValue simpleExecute(ValueRow inputRow) {
		// typechecking the input
		RGLValue rdfValue1 = inputRow.get(INPUT1);
		RGLValue rdfValue2 = inputRow.get(INPUT2);
				
		if (!rdfValue1.isLiteral() || !rdfValue2.isLiteral() )
			return ValueFactory.createNull("Cannot handle non literal input in "
					+ getFullName());

		// extracting the strings from the input
		String input1 = rdfValue1.asLiteral().getValueString();
		String input2 = rdfValue2.asLiteral().getValueString();
		Integer a = new Integer(input1);
		Integer b = new Integer(input2);
		
		return ValueFactory.createLiteralDouble(a*b);
	}

}

```

With respect to the template, there are now two inputs registered in the constructor, instead of one given by the template.

The XML description of the function also needs to be modified to include one additional input:

```

<processor label="Multiply" category="Custom Java">
	<description>
	This is a custom java function that multiplies two inputs
	</description>
	<inputs>
		<function type="custom-java"> 
			<param name="implementation" value="multiply.Multiply"/>
		</function>
		<data iterate="false" name="input1" label="INPUT1">
			<type><var name="input1"/></type>
		</data>
		<data iterate="false" name="input2" label="INPUT2">
			<type><var name="input2"/></type>
		</data>
	</inputs> 
	<output/>
</processor>

```

In case you also want to use the workflow generated by the Wizard (this is useful to have a starting point to run the component in RDFGears, without needing to create a workflow first), you also need to modify it to include the additional input:

```

<?xml version="1.0" encoding="UTF-8"?>
<rdfgears>
   <metadata>
      <id>MultiplyWF</id>
	  <name>MultiplyWF</name>
	  <category/>
      <description>MultiplyWF</description>
      <password>imreal</password>
   </metadata>
   <workflow>
      <workflowInputList x="57" y="66">
         <workflowInputPort name="a"/>
         <workflowInputPort name="b"/>
      </workflowInputList>
      <network output="node_628" x="379" y="239">
         <processor id="node_628" x="245" y="84">
            <function type="custom-java">
               <config param="implementation">multiply.Multiply</config>
            </function>
            <inputPort iterate="false" name="input1">
               <source workflowInputPort="a"/>
            </inputPort>
            <inputPort iterate="false" name="input2">
               <source workflowInputPort="b"/>
            </inputPort>
         </processor>
      </network>
   </workflow>
</rdfgears>

```

### Register the artefacts ###

The following steps are already provided by the template, so you should not need to modify them.

The already produced artefacts has to be registered so that the engine knows about their existence. The registration is done in the `start` method of the `Activator` java class.

In order to register functions, the following code has to be included:

```
context.registerService(FunctionDescriptor.class, new FunctionDescriptor(getClass().getResourceAsStream("/<xml>"), <class>), null)
```

Where `<xml>` is the path to the xml file describing the function and `<class>` is the java class that contains function's logic.

In order to register workflows the following code has to be included:

```
context.registerService(WorkflowTemplate.class, new WorkflowTemplate(getClass().getResourceAsStream("/<xml>")), null);
```

Where `<xml>` is the path to the xml file describing the workflow.

### Configure plug-in export ###

Before exporting the plug-in make sure all resource files (xml, etc.) will be included. Eclipse has a tab called `Build` where you can specify what resources will be included in the Binary buid (left part of the panel). Include at least the following:

  * bin
  * META-INF
  * functions
  * templates

More information on how to do that can be found under the _Plug-in Build_ section in Eclipse documentation.

## Step 3 - Export the project ##

Once the plug-in is ready, it has to be exported into a jar file so that it can be later deployed to the U-Sem platform. Eclipse provides wizard that completely automates this process. It is accessible under `File/Export/Deployable plug-ins and fragments`. More information about it can be found under the _Plug-in Export_ section in Eclipse documentation.

# Deploy U-Sem #

To deploy the OSGi-enabled version of U-Sem, follow the instructions at [InstallingUSem](InstallingUSem.md), using the [u-sem-osgi](https://code.google.com/p/rdfgears/source/checkout?repo=u-sem-osgi) repository instead of the u-sem one. Compile the sources, then copy the following wars to the `Tomcat/<webapps>`:

  * `rdfgears.u-sem-osgi/rdfgears-rest/target/rdfgears-rest-0.2-SNAPSHOT.war` as `rdfgears-rest.war`
  * `rdfgears.u-sem-osgi/rdfgears-ui/target/rdfgears-ui-0.1-SNAPSHOT.war` as `rdfgears-ui.war``
  * `rdfgears.u-sem-osgi/datamanagement-ui/target/datamanagement-ui-0.2-SNAPSHOT.war` as `datamanagement-ui.war`
  * `rdfgears.u-sem-osgi/u-sem/target/u-sem-0.2-SNAPSHOT.war` as `u-sem.war`
  * `rdfgears.u-sem-osgi/usem-plugin-ui/target/usem-plugin-ui-0.2-SNAPSHOT.war` as `u-sem-plugin-ui.war`

You can use the following script to copy automatically all war files to the Tomcat directory, assuming you are launching it from the root of the cloned repository `rdfgears.u-sem-osgi' and that Tomcat is installed in /usr/local/apache-tomcat-6.0.37/:

```
for i in $(find . -name *.war -print);
do
      DEST=$(echo ${i} | sed 's/\.\/.*\/\(.*\)-0.2-SNAPSHOT.war/\1.war/g');
      cp  ${i} /usr/local/apache-tomcat-6.0.37/webapps/${DEST};
done
```

The configuration then is the same as the one in [InstallingUSem](InstallingUSem.md)

# Install plug-ins in U-Sem #

Once the OSGi-enabled version of U-Sem has been deployed, the functionality for installing a plug-in to the U-Sem platform is available under the `Plug-ins` section in the user interface of U-Sem.

Users have to click on the `Upload new plug-in` button which will enable them to select a plug-in to be installed. Finally, after the successful installation of the plug-in all artefacts provided by it will be available to use in U-Sem.