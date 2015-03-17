# Introduction #

The RDF Gears GUI is a web-application that can be run in a java application server.

Make sure you first [installed and built](Installing_RDFGears.md) RDF Gears correctly. The build process create a jar file `rdfgears-ui/target/rdfgears-ui-0.1-SNAPSHOT.war`.

This file must be deployed under the path http://host/rdfgears-ui in your favorite web application server, and it should have access to the $TEMP/rdfgears/ directory (the $TEMP dir depends on the configuration of your Web application server but it may just be /tmp/ ).

# Details #
The build process gives you a war file that you can deploy in your application server of choice. If you don't have such a server running and/or you don't know what to do, then you can follow these instructions to set up the Jetty Webserver.

make sure the following directories exist:
  * /tmp/rdfgears/workflows/
  * /tmp/rdfgears/valuestore/ (if you want to use the value store option for HTTP)

Install rdfgears-ui on jetty with the following commands:

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
 cp $RDFGEARS_DIR/rdfgears-ui/target/rdfgears-ui-0.1-SNAPSHOT.war $JETTY_LOCATION/${JETTY_NAME}/webapps/rdfgears-ui.war
 
 # start jetty
 cd $JETTY_LOCATION/${JETTY_NAME}
 java -jar start.jar
```

You should now be able to access the GUI at http://localhost:8080/rdfgears-ui . Note that if you want the workflows to be executable, you must [install the RESTful HTTP interface](Install_RESTful_Interface.md).