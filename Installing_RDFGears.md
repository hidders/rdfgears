# Introduction #
This page describes how to get RDF Gears. Currently there are no releases. You can get a development branch of RDF Gears and compile it using Maven. It does not hurt to understand the RDF Gears [project organization](ProjectOrganization.md), but it is not strictly necessary.

## Requirements ##
  * You have a fast internet connection (LAN preferred over wifi) or you are very patient
  * The following tools must be installed (lower versions may work, but not recommended)
    * Java SDK (>= 6)
    * Mercurial (>= 1.8) for version control - http://mercurial.selenic.com/downloads/
    * Maven (>= 3) for building RDF Gears - http://maven.apache.org/download.html

# Installing RDF Gears #
Mercurial is used for version control. This section describes how to use Mercurial to create a clone of the RDF Gears code, and how to retrieve updates to this code.

## Clone the project ##

To check out the whole thing, start your favorite terminal and run (preferably in your eclipse Workspace directory, if you want to hack on it later):

`[user@host ~/workspace/]$ hg clone https://code.google.com/p/rdfgears.master rdfgears`

Be patient, as this operation clones the entire version history and a progress indicator is lacking. It may take a while and requires >300MB of diskspace. It results in the following filesystem tree.

  * working directory
    * rdfgears/
      * .hg/
      * .hgsub/
      * .project/
      * pom.xml
      * rdfgears-core/
        * .project
        * pom.xml
        * .hg/
        * src/
        * ...
      * rdfgears-ui/
        * .project
        * pom.xml
        * .hg/
        * src/
        * ...
      * rdfgears-rest/
        * .project
        * pom.xml
        * .hg/
        * src/
        * ...
      * rdfgears-cli/
        * .project
        * pom.xml
        * .hg/
        * src/
        * ...
      * rdfgears-plugins/
        * .project
        * pom.xml
        * .hg/
        * src/
        * ...

## Build the project ##
Build the project by running

`[user@host ~/workspace/]$ cd rdfgears`

`[user@host ~/workspace/rdfgears]$ mvn clean package -DskipTests`

This will build RDF Gears and will omit executing the unittests. It results in the following files:

  * `rdfgears-core/target/rdfgears-exec-0.1-SNAPSHOT-shade.jar` - an executable jar used to [run the RDF Gears interpreter from the command line](RDFGearsCommandLine.md).
  * `rdfgears-core/target/rdfgears-core-0.1-SNAPSHOT.jar` - a library with RDF Gears classes.
  * `rdfgears-ui/target/rdfgears-ui-0.1-SNAPSHOT.war` - a web application to [run the RDF Gears User Interface](Run_the_GUI.md) for workflow editing.
  * `rdfgears-rest/target/rdfgears-rest-0.1-SNAPSHOT.war` - a web application to [run the RDF Gears RESTful HTTP interface](Install_RESTful_Interface.md) for workflow execution.


## Receive updates ##
You can incorporate updates from the RDF Gears repository with the command:

`[user@host ~/workspace/rdfgears]$ hg pull -u`

and then you can build the project again.