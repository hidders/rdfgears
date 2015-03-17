# Introduction #

This page describes how to run the RDF Gears interpreter ("engine") as a command line tool. This is useful for workflow debugging, interrupting workflow execution, and for [core development](DevelopingRDFGears.md).

Make sure you first [installed and built](InstallingUSem.md) RDF Gears correctly. The build process creates a fat executable jar `rdfgears-cli-0.1-SNAPSHOT-shade.jar`. This is the one relevant for this page.


# Details #
To run the RDF Gears interpreter, use the command

`[user@host ~/workspace/rdfgears/rdfgears-core]$ java -jar ../rdfgears-cli/target/rdfgears-cli-0.1-SNAPSHOT-shade.jar --help`

to see the options. This will show something like

```
RDF Gears version 0.1
Use the --help option for usage info

The options available are:
        [--debug-level -d value] : The log4j debug level
                (DEBUG/INFO/WARN/ERROR/OFF etc)
        [--disable-optimizer] : Disable the workflow optimizer
        [--diskBased] : Use disk based backend. Only recomended for really large data.
        [--help] : show this help message
        [--outputformat value] : The format for the RGL output data [xml|informal|none]. Default: xml
        [--typecheck-only -t] : Do not execute, only typecheck
        [--workflow -w value] : Path to the workflow to be executed
        [--workflow-path value] : List of ':' separated paths where the
                (nested) workflows can be found. Default: .:./workflows/

```

## Executing an example workflow ##

Assume a workflow `my/workflow` is defined, and is stored in the file `/tmp/rdfgears/my/workflow.xml`. The workflow directory is thus `/tmp/rdfgears/`. The workflow is executed as follows:

`[user@host ~/workspace/rdfgears/rdfgears-core]$ java -jar target/rdfgears-exec-0.1-SNAPSHOT-shade.jar --workflow-path=/tmp/rdfgears -w my/workflow`

## Using the rdfgears.config file ##
Note that some configuration parameters will be fetched from the file `rdfgears.config`, if it is available in the current working directory. For example, you can set

`workflow.path = .:./workflows/:/tmp/rdfgears/workflows/`

to make RDF Gears look for workflows in the current directory, and then in `./workflows`, and then in `/tmp/rdfgears`. Command line options (e.g. the `--workflow-path` setting) will override any settings in the `rdfgears.config` file.