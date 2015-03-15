# Introduction

Apache Axis documentation comes with a 'samples/encoding' folder from where I have extracted most of the code for this tutorial.

I am going to show how a SOAP web service client can adapt to changes in the WDSL dynamically.

# Prerequisites

Java, Servlet Container (using JBoss application Server here), Basic Linux/Unix but if you are on Windows and you haven't done it install Cygwin and try the below, you might get introduces to basic Unix Shell (bash) as well ;-)

# Initial discussion

As far as I can say after a couple of days thinking on ways to get dynamic proxies working here are the options we have (Using Apache Axis)
1. Create custom serialization.
2. Modify on the fly the SOAP response leaving just the needed node (Low level XML manipulation needed)
3. Use responses as org.w3c.dom.Element (Low level XML manipulation needed)
4. Custom module to get Last-Modified from a HEAD request, rebuild static stubs through WSDL2Java, then reload classes (Way too much coding at the least)

The steps below should show clearly enough how a static stub generated via Java2WDSL will fail if the web service changes one of the complex types being exposed through the called method. It also demonstrate how this can be solved building a dynamic proxy through the use of custom Deserializers.

# The solution

1. Prepare environment:
```
JBOSS_HOME="/usr/local/jboss-4.0.4.GA/server/default"
JBOSS_DEPLOY="$JBOSS_HOME/deploy"
JBOSS_LIB="$JBOSS_HOME/lib"
AXIS_LIB="/home/nurquiza/Desktop/axis-1_4/lib"
JBOSS_AXIS_DEPLOY="$JBOSS_DEPLOY/axis.war"
JBOSS_AXIS_CLASSES="$JBOSS_AXIS_DEPLOY/WEB-INF/classes"
PROJECTPATH="/home/nurquiza/projects/axis/dynamicws"
SOURCEPATH="$PROJECTPATH/src"
BINPATH="$PROJECTPATH/bin"
CLIENTBINPATH="$BINPATH/client"
BUILDPATH="$PROJECTPATH/build"
CLIENTBUILDPATH="$BUILDPATH/client"
CLASSPATH="$AXIS_LIB/wsdl4j-1.5.1.jar:$AXIS_LIB/axis.jar:$AXIS_LIB/jaxrpc.jar:$AXIS_LIB/commons-logging-1.0.4.jar:$AXIS_LIB/commons-discovery-0.2.jar:$AXIS_LIB/saaj.jar:$JBOSS_LIB/activation.jar:$JBOSS_LIB/mail.jar"
mkdir $SOURCEPATH 
mkdir $BINPATH
mkdir $CLIENTBINPATH
mkdir $BUILDPATH
mkdir $CLIENTBUILDPATH
```
2. Install Axis in JBoss. Just download axis-1_4.zip, uncompress and:
```
cp "axis-1_4/webapps/axis" "$JBOSS_DEPLOY/axis.war" 
```
3. Get the sources for this tutorial from SVN.
  http://nestorurquiza.googlecode.com/svn/trunk/axis-dynamic-binding/dynamicws/
4. Compile sources
```
javac -g $SOURCEPATH/com/nestorurquiza/ws/axis/service/DynamicServiceSampleImpl.java -sourcepath $SOURCEPATH -d $BINPATH
```
5. Create the wsdl file
```
java -cp $CLASSPATH:$BINPATH org.apache.axis.wsdl.Java2WSDL -i com.nestorurquiza.ws.axis.service.DynamicServiceSampleImpl -o $BUILDPATH/DynamicServiceSample.wsdl -l "http://localhost:8080/axis/services/DynamicServiceSample" -n "urn:DynamicServiceSample" -p"com.nestorurquiza.ws.axis.service" "urn:DynamicServiceSample" com.nestorurquiza.ws.axis.service.DynamicServiceSample
```
6. Create client stubs and  deploy/undeploy descriptor (wsdd files). Replace Person.java stub with  with the original since the stubs change the scope of public members to private. We need them to be public to use the class later for the dynamic proxy approach.
```
rm -R $BUILDPATH/com
java -cp $CLASSPATH org.apache.axis.wsdl.WSDL2Java -o $BUILDPATH -d Application -s -S true -Nurn:DynamicServiceSample com.nestorurquiza.ws.axis.service $BUILDPATH/DynamicServiceSample.wsdl
cp $SOURCEPATH/com/nestorurquiza/ws/axis/service/Person.java $BUILDPATH/com/nestorurquiza/ws/axis/service/
```
7. Change $BUILDPATH/com/nestorurquiza/ws/axis/service/deploy.wsdd
```
sed -i 's/DynamicServiceSampleSoapBindingSkeleton/DynamicServiceSampleImpl/g' $BUILDPATH/com/nestorurquiza/ws/axis/service/deploy.wsdd 
```
8. Deploy the service (undeploy first just in case it exists)
```
cp -r $BINPATH/com $JBOSS_AXIS_CLASSES
java -cp $CLASSPATH org.apache.axis.client.AdminClient $BUILDPATH/com/nestorurquiza/ws/axis/service/undeploy.wsdd
java -cp $CLASSPATH org.apache.axis.client.AdminClient $BUILDPATH/com/nestorurquiza/ws/axis/service/deploy.wsdd
touch "$JBOSS_AXIS_DEPLOY/WEB-INF/web.xml"
```
9. Write the client DynamicServiceClient. Build it using its own copy of stubs.
```
cp -r $BUILDPATH/com $CLIENTBUILDPATH
javac -cp $CLASSPATH $SOURCEPATH/com/nestorurquiza/ws/axis/client/DynamicServiceClient.java -sourcepath $CLIENTBUILDPATH -d $CLIENTBINPATH
```
10. Now let us create another client using a dynamic proxy technik to generate the stubs at runtime. So we create our new client and build both:
```
javac -cp $CLASSPATH $SOURCEPATH/com/nestorurquiza/ws/axis/client/**.java -sourcepath $CLIENTBUILDPATH -d $CLIENTBINPATH
```
11. Run the static stub client:
```
java -cp $CLASSPATH:$CLIENTBINPATH com.nestorurquiza.ws.axis.client.DynamicServiceClient
```
12. Run the dynamic proxy client:
```
java -cp $CLASSPATH:$CLIENTBINPATH com.nestorurquiza.ws.axis.client.DynamicServiceProxyClient
```
You should get for both cases:
  Response from DynamicServiceSample#getPerson()#getFirstName(): Fred

13. Let us add a new member 'address' for 'Person' class. 

14. Repeat from 4 to 8. Do not run 9 and 10. We want to use the same clients compiled when the webservice was not managing Person#address.

15. Run the clients again (11 and 12) As you see the first request fails this time with:
  org.xml.sax.SAXException: Invalid element in com.nestorurquiza.ws.axis.service.Person - address

# Useful Notes

 - At any time you can test if the webservice is available:
   {{{
   http://localhost:8080/axis then click on 'List' link 
   }}}
 - If you want to clean generated classes just run:
   {{{
   find $BINPATH -name "**.class"|xargs rm
   }}}
 - To undeploy the web service use:
   {{{
   java -cp $CLASSPATH org.apache.axis.client.AdminClient $BUILDPATH/com/nestorurquiza/ws/axis/service/undeploy.wsdd
   }}}
 - To see TCP traffic and so all SOAP messages going back and forth use 'tcpmon' utility. You can change the client code to connect to port 8081 for example and then run:
   {{{
   java -cp $CLASSPATH org.apache.axis.utils.tcpmon 8081 localhost 8080 &
   }}}
 - To play with responses from services from the command prompt use one of the provided samples (axis-1_4/samples/client/DynamicInvoker.java)
   {{{
   java -cp $CLASSPATH:$CLIENTBINPATH com.nestorurquiza.ws.axis.client.DynamicInvoker "http://localhost:8080/axis/services/DynamicServiceSample?wsdl" getPerson  "Right Said Fred")
   }}}
 - Look for custom serialization in the Axis User Guide.
 - Look for answers in axis-user mailing list