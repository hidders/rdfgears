# Introduction #

rdfgears-rest is a web-application providing a RESTful interface for execution of RDF Gears workflows. The installation process is very similar to that of the rdfgears-ui application.

Make sure you first [installed and built](Installing_RDFGears.md) RDF Gears correctly. The build process create a jar file `rdfgears-rest/target/rdfgears-rest-0.1-SNAPSHOT.war`.

This file must be deployed under the path http://host/rdfgears-rest in your favourite web application server, if it is to be used in combination with rdfgears-ui. The application must have read-access to /tmp/rdfgears/ (this path can be changed in the file rdfgears/rdfgears-rest/src/main/webapp/WEB-INF/classes/rdfgears.config that is included in the .war upon packaging).

# Details #
Install rdfgears-rest on jetty with the following commands:

```
 # config
 RDFGEARS_DIR=~/workspace/rdfgears # change this: dir where your rdfgears is
 JETTY_LOCATION=~ # change this: dir where you want jetty

 # build RDF Gears
 cd $RDFGEARS_DIR
 mvn clean package -DskipTests

 # get Jetty
 cd $JETTY_LOCATION
 JETTY_NAME=jetty-7.2.0
 wget http://apsthree.st.ewi.tudelft.nl/~af09017/${JETTY_NAME}.tgz
 tar -xzvf ${JETTY_NAME}.tgz

 # deploy the file
 cp $RDFGEARS_DIR/rdfgears-rest/target/rdfgears-rest-0.1-SNAPSHOT.war $JETTY_LOCATION/${JETTY_NAME}/webapps/rdfgears-rest.war
 
 # start jetty
 cd $JETTY_LOCATION/${JETTY_NAME}
 java -jar start.jar
```

You should now be able to execute the workflow MY/WORKFLOW/NAME the GUI at http://localhost:8080/rdfgears-rest/user/input/MY/WORKFLOW/NAME


# Input parameters #
detail this:
  * Workflow inputs can be given as GET parameters
    * e.g. http://localhost:8080/rdfgears-rest/user/input/MY/WORKFLOW/NAME?lit="hi"@en&uri=<http://example.com>&ref=_0982kj39sfjsdf
    * URI's and literals are encoded N-Triple style.
    * a '`_`'-prefix for a value is used to refer to an internally stored value
    * a workflow output can be stored as an internal value by setting the '-store' GET parameter, after which a generated id will be returned.

For example, a HTTP GET on the URI

`http://localhost:8080/rdfgears-rest/user/execute/my/workflow?v1=_2929234&v2="apple"`

will execute workflow `my/workflow` with input '`v1`' the value stored under id 2929234 and input '`v2`' the plain literal "apple".