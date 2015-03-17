# Introduction #

The RDF Gears project is managed with different tools for different tasks.
  * Mercurial is used for version control
  * Maven is used for build- and dependency management
  * Eclipse can be used for development

# Mercurial #
RDF Gears consists of  Mercurial projects hosted on the same server. The path layout on the server is as follows:

  * Server-path
    * rdfgears  (container project)
      * rdfgears-cli (run from command line)
      * rdfgears-rest (rest interface)
      * rdfgears-plugins (services)
      * rdfgears.core (core functionality, workflow interpreter)
      * rdfgears.ui (web-based GUI)

The rdfgears project references the other projects as sub-projects. So on your filesystem, the rdfgears-core and rdfgears-ui directories will be nested in the rdfgears container project.
The .hg files are for Mercurial. Don't touch them. The rdfgears/.hgsub file defines the subprojects. It maps the local paths to server paths.

# Maven #
The pom.xml files are Maven files to manage the build- and dependency process.
There are four maven 'modules' or 'artifacts': a parent (container) and three child modules.
  * rdfgears
    * rdfgears-core
    * rdfgears-ui
    * rdfgears-rest
    * rdfgears-cli
    * rdfgears-plugins

Some of the rdfgears-... modules depend on the rdfgears-core module.

# Eclipse #
To hack on RDF Gears using Eclipse, you most probably want to m2e Eclipse plugin (http://www.eclipse.org/m2e/) to make it correctly build. Then you can import the parent project and the sub project that you care about. The import will create the Eclipse .project files.