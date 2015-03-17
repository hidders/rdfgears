# Introduction #
This page describes how to install U-Sem. This include the front-end (rdfgears-ui), as well as the REST services (rdfgears-rest).

There are two ways to install it: binary releases and from source.

# Installing U-Sem from binary releases #

## Requirements ##
  * Apache Tomcat (>= 6.0) web server - http://tomcat.apache.org/
  * MYSQL server (any recent version, you need to be admin to run a script). We advise to use tools such as WAMP (for Windows, http://www.wampserver.com) or MAMP (for Mac, http://www.mamp.info) that offer also phpMyadmin interfaces.

Binary releases are at https://drive.google.com/folderview?id=0BxPtN0IVRdDlelFFNFFabmtPVE0&usp=sharing. Please note that binaries might not correspond to the latest release of the source code. The release is contained in the name of the binary.

The binaries that you need to install are:

  * `rdfgears-rest-<version>.war`

  * `rdfgears-ui-<version>.war`

  * `rdfgears-<version>.zip`

  * `uuid_create_script-<version>.sql`


After downloading the files, please follow the instructions from the **Deploy U-Sem** section.

# Installing U-Sem from sources #

You need to compile the source code. You can get the development branch of the project and compile it using Maven.

## Requirements ##
  * Preferably a fast internet connection (LAN preferred over wifi) or you are very patient
  * The following tools must be installed (lower versions may work, but not recommended)
    * Java SDK (>= 6)
    * Mercurial (>= 1.8) for version control - http://mercurial.selenic.com/downloads/
    * Maven (>= 3) for building RDF Gears - http://maven.apache.org/download.html
    * Apache Tomcat (>= 6.0) web server - http://tomcat.apache.org/
    * MYSQL server (any recent version, you need to be admin to run a script). We advise to use tools such as WAMP (for Windows, http://www.wampserver.com) or MAMP (for Mac, http://www.mamp.info) that offer also phpMyadmin interfaces.

## Clone the project ##

Mercurial is used for version control. This section describes how to use Mercurial to create a clone of the U-Sem code, and how to retrieve updates to this code.

To check out the whole thing, start your favorite terminal and run (preferably in your eclipse Workspace directory, if you want to hack on it later):

Stable version:
`[user@host ~/workspace/]$ hg clone https://code.google.com/p/rdfgears.master`

Developer version:
`[user@host ~/workspace/]$ hg clone https://code.google.com/p/rdfgears.u-sem`

Be patient, as this operation clones the entire version history and a progress indicator is lacking. It may take a while and requires >300MB of diskspace.

You need to install manually the lib jsr173-ri-1.0.jar.

Download it from:

`http://www.mmbase.org/maven2/com/bea/xml/jsr173-ri/1.0/jsr173-ri-1.0.jar`

and then run:

`mvn install:install-file -DgroupId=com.bea.xml -DartifactId=jsr173-ri -Dversion=1.0 -Dpackaging=jar -Dfile=<path\_to\_jsr173-ri-1.0.jar> -DgeneratePom=true'


## Receive updates ##
You can incorporate updates from the U-Sem repository with the command:

`[user@host ~/workspace/usem]$ hg pull -u`

## Configuration before code build ##

You need to specify some settings before you run U-Sem. You can do it at this stage or once the project is deployed. You might want to do it now if you have not built the code yet, and after if you already have built it and you just need to change a parameter in the config file (for the latter you can follow the instructions at **Configuration after code build**).

The example configuration file is in `rdfgears.u-sem/rdfgears-rest/src/main/webapp/WEB-INF/rdfgears.config.example`. Rename it to rdfgears.config and edit it.

In this file you need to specify the database location and name (default localhost/imreal), the username and password. To set up the database correctly, you must run the sql script `rdfgears.u-sem/rdfgears-rest/src/main/webapp/sql/uuid_create_script.sql` on the server where MySQL is running. This script creates the necessary databases and table for a default user imreal with password imreal. You should change this and set it to same values you set in rdfgears.config.

In rdfgears.config you also need to set your Flickr API key and your Twitter API settings. The other parameters do not need to be changed, since they are used for performance tweaking.


## Build the project ##
Build the project by running

`[user@host ~/workspace/]$ cd rdfgears.u-sem`

`[user@host ~/workspace/usem]$ mvn clean package -DskipTests`

This will build U-Sem and will omit executing the unit-tests.

# Deploy U-Sem #

## When installing from Sources ##

In order to deploy the project to the Tomcat web server the following files have to be copied to the "webapps" folder of the server:

  * `usem\rdfgears-rest\target\rdfgears-rest-0.1-SNAPSHOT.war` as `rdfgears-rest.war`

  * `usem\rdfgears-ui\target\rdfgears-ui-0.1-SNAPSHOT.war` as `rdfgears-ui.war``

The directory `rdfgears.u-sem\rdfgears-plugins\runtimeResources\rdfgears` has to be copied to tomcat's home directory `<TOMCAT>` (the directory that contains webapps and bin), keeping the same name (rdfgears).

## When installing from Binaries ##

In order to deploy the project to the Tomcat web server the following files have to be copied to the "webapps" folder of the server:

  * `rdfgears-rest-<version>.war` as `rdfgears-rest.war`

  * `rdfgears-ui-<version>.war` as `rdfgears-ui.war``

The zip file `rdfgears-<version>.zip` has to be renamed to `rdfgears`, unzipped and copied to tomcat's home directory `<TOMCAT>` (the directory that contains webapps and bin).

## Configuration after code build ##

If you have not done the configuration in section **Configuration before code buid** (if you are installing from binaries you have not), you need to specify some settings in the configuration file rdfgears.config. The example file rdfgears.config.example is in `<TOMCAT>/webapps/rdfgears-rest/WEB-INF` if the rdfgears-rest.war has been exploded by Tomcat. If not, you can either make Tomcat explode it for you by stopping and restarting it, or you can run:

`jar -xvf rdfgears-rest.war`

Rename rdfgears.config.example to rdfgears.config. In this file you need to specify the database location and name (default `localhost/imreal`), the username and password. To set up the database correctly, you must run the sql script `<TOMCAT>/webapps/rdfgears-rest/sql/uuid_create_script.sql` if you are installing from sources, or `uuid_create_script-<version>.sql` if you are installing from binaries. This script creates the necessary databases and table for a default user `imreal` with password `imreal`. You should change this and set it to same values you set in rdfgears.config.

In rdfgears.config you also need to set your Flickr API key and Twitter API settings. The other parameters do not need to be changed, since they are used for performance tweaking.

Finally, the web server has to be re/started.

# Run U-Sem #

Once the project is successfully deployed and the server is running U-Sem can be accessed on the following URL (depending on the configuration of the web server):

http://localhost:8080/rdfgears-ui/