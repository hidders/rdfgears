# Workflows #

Workflows can either be created graphically or directly written as XML documents. A number of predefined workflows already exist, and they are stored within `rdfgears.u-sem\rdfgears-plugins\runtimeResources\rdfgears\data\workflows` (remember that this is the `rdfgears` directory which needs to be copied to tomcat's home directory when deploying U-Sem as explained in [InstallingUSem](InstallingUSem.md)).


## Adding a new Workflow to the Repository ##

When creating a new workflow graphically, the **save** option in the interface will create a new xml file describing the workflow in `<TOMCAT>\rdfgears\data\workflows`.
If this workflow should be included in the Google Code repository, the xml file also needs to be copied to `rdfgears.u-sem/rdfgears-plugins/runtimeResources/rdfgears/data/workflows`.


## Setting up a Workflow Graphically ##

You can either create a workflow by copying an existing one or by creating one from scratch.

To derive a new workflow from an existing one, simply click the **Copy** button available under each workflow in the UI. Make sure when asked to change the `Workflow Name` and `Workflow Unique ID`; the former is the name the workflow will appear under in the workflow list and the latter is the filename the workflow will be saved as.

To create a workflow from scratch, chose the option **File->New Workflow** to create a graphic workflow template in the UI. It contains two nodes: Input and Output. Both nodes are required to exist for each workflow.

On the right hand side, a number of **Details** need to be set for the workflow:
  * Workflow name: the name you can find your workflow under in the list on the left hand side of the UI
  * Workflow unique id: this is the actual filename the workflow will be saved under within `<TOMCAT>\rdfgears\data\workflows` (no need to add .xml here)
  * Workflow description: a description of what the workflow does, what it takes as input and what it outputs (optional)
  * Choose Category: if no category is chosen, the workflow is sorted under **Uncategorized**

Finally, don't forget to **File->Save** the workflow.