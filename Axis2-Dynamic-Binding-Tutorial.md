# Introduction

Using Axis and the old JAX-RPC approach is OK and I have already written a tutorial as to how to dynamically (no compilation time stubs but runtime class binding) consume SOAP web services. However JAX-WS is supposed to replace the now old JAX-RPC.

Axis2 supports JAX-WS and I will be using it to demonstrate how to build a dynamic client. Class org.apache.axis2.client.ServiceClient is the starting point. Be aware that at the time of this writing if the web service is returning multiRef elements then ServiceClient#sendReceive() is buggy as of version 1.4 ( https://issues.apache.org/jira/browse/AXIS2-2408 ).

The second important class is org.apache.axis2.databinding.utils.BeanUtil which allows to bind the SOAP response to a Java Object.

I will use Tomcat (Jetty lacks of out of the box default hot deployment which would force me to use maven for a quick proof of concept) this time to host the web service. In order to install Axis2 in JBoss some modifications are needed specially because axis2.war includes commons-logging and log4j and JBoss is not happy when a WAR includes them (making the long story short). If you go for JBoss I will recommend using version 5 and above. As a side note I will be using Tomcat runnning on port 8085. So you might have to change the URLs below to reflect your Tomcat listening port.

The whole source code is accessible from http://nestorurquiza.googlecode.com/svn/trunk/axis2-dynamic-binding/


# Details

 1. Follow the guide from http://ws.apache.org/axis2/1_4/quickstartguide.html to get Axis2 working. We will need some environment variables. In my case (working on linux)
```
export TOMCAT_HOME="/usr/local/apache-tomcat-6.0.18/"
export AXIS2_HOME="/home/nurquiza/libraries/axis2-1.4"
export PROJECT_HOME="/home/nurquiza/projects/axis2/dynamicws"
export SERVICE_PROJECT_HOME="$PROJECT_HOME/service"
export CLIENT_PROJECT_HOME="$PROJECT_HOME/client"
```
 1. We will need hotupdate enabled in Axis2 so edit $TOMCAT_HOME/webapps/axis2/WEB-INF/conf/axis2.xml:
```
<parameter name="hotupdate">true</parameter>
```
 1. Let us build a service using a POJO approach. Our classes are:
```
com.nestorurquiza.ws.axis2.service.DynamicServiceSample
com.nestorurquiza.ws.axis2.service.Person
com.nestorurquiza.ws.axis2.service.Address
```
  Be sure Person#address and its accessors are commented out. Also all references to Address from DynamicServiceSample should be commented out. We want to uncomment them later to prove our client will not break if a new complex type is included as part of the response.
 1. Copy the quickstartaxiom example as a scaffold for the new project, for example:
```
cp -r $AXIS2_HOME/samples/quickstartaxiom $SERVICE_PROJECT_HOME/service
```
 1. Edit $SERVICE_PROJECT_HOME/resources/META-INF/services.xml.
```
<service name="DynamicServiceSample" scope="application" targetNamespace="http://quickstart.samples/">

    <description>

        Dynamic Sample Service

    </description>

    <messageReceivers>

        <messageReceiver mep="http://www.w3.org/2004/08/wsdl/in-only"

                         class="org.apache.axis2.rpc.receivers.RPCInOnlyMessageReceiver"/>

        <messageReceiver mep="http://www.w3.org/2004/08/wsdl/in-out"

                         class="org.apache.axis2.rpc.receivers.RPCMessageReceiver"/>

    </messageReceivers>

    <schema schemaNamespace="http://dynamicws.nestorurquiza.com"/>

    <parameter name="ServiceClass">com.nestorurquiza.ws.axis2.service.DynamicServiceSample</parameter>

</service>
```
 1. Edit the ant build.xml script
```
<project name="dynamicws" basedir="." default="generate.service">



    <property environment="env"/>

    <property name="AXIS2_HOME" value="${env.AXIS2_HOME}"/>



    <property name="build.dir" value="build"/>



    <path id="axis2.classpath">

        <fileset dir="${AXIS2_HOME}/lib">

            <include name="**.jar"/>

        </fileset>

    </path>


    <path id="client.class.path">

		  <fileset dir="${AXIS2_HOME}/lib">

			  <include name="**.jar" />

		  </fileset>

		  <pathelement location="${build.dir}/classes" />

	  </path>
	  

    <target name="compile">

        <mkdir dir="${build.dir}"/>

        <mkdir dir="${build.dir}/classes"/>



        <!--First let's compile the classes-->

        <javac debug="on" 
               fork="true"
               destdir="${build.dir}/classes" 
               srcdir="${basedir}/src"
               classpathref="axis2.classpath">

        </javac>

    </target>
    

    <target name="generate.service" depends="compile">

        <!--aar them up -->

        <copy toDir="${build.dir}/classes" failonerror="false">

            <fileset dir="${basedir}/resources">

                <include name="**/*.xml"/>

            </fileset>

        </copy>

        <jar destfile="${build.dir}/DynamicServiceSample.aar">

            <fileset excludes="**/Test.class" dir="${build.dir}/classes"/>

        </jar>

    </target>


    <target name="run.client" depends="compile">

        <java classname="com.nestorurquiza.ws.axis2.client.DynamicServiceSampleClient">

            <classpath refid="client.class.path" />

        </java>

    </target>
    

    <target name="clean">

        <delete dir="${build.dir}"/>

    </target>
    
    <target name="generate.wsdl" depends="compile">

        <taskdef name="java2wsdl"

                 classname="org.apache.ws.java2wsdl.Java2WSDLTask"

                 classpathref="axis2.classpath"/>

        <java2wsdl className="com.nestorurquiza.ws.axis2.service.DynamicServiceSample"

                   outputLocation="${build.dir}"

                   targetNamespace="http://dynamicws.nestorurquiza.com/"

                   schemaTargetNamespace="http://dynamicws.nestorurquiza.com/xsd">

            <classpath>

                <pathelement path="${axis2.classpath}"/>

                <pathelement location="${build.dir}/classes"/>

            </classpath>

        </java2wsdl>

    </target>



</project>
```
 1. Build the web service
```
cd $SERVICE_PROJECT_HOME
ant clean generate.service
```
 1. Deploy the aar file (packaged web service deployment file) in Tomcat.
```
cp $SERVICE_PROJECT_HOME/build/DynamicSampleService.aar $TOMCAT_HOME/webapps/axis2/WEB-INF/services
```
 1. Test the web service. You will see firstName and lastName comming up.
```
http://localhost:8085/axis2/services/DynamicServiceSample/getPerson?userName=Right%20Said%20Fred
```
 
Next step is to build a client for which we will use the AXIOM method
 1. Copy server folder into client to use it as scaffolder. Remove  $CLIENT_PROJECT_HOME/resources folder from client. Remove everything below $CLIENT_PROJECT_HOME/build/classes/.
 1. Use the samples.quickstart.service.pojo.StockQuoteService sample class as scaffold to build the client (com.nestorurquiza.ws.axis2.client.DynamicServiceSampleClient). Our client service depends on Person POJO. Note this Person class is the same we used when building the service. They will be different though in next steps. It will never use the Address class, so our Client classes are:
```
com.nestorurquiza.ws.axis2.client.DynamicServiceSampleClient
com.nestorurquiza.ws.axis2.service.Person
```
 1.Run the client. The result is the firstName extracted from a 'Person' response that contains firstName and lastName:
```
cd $CLIENT_PROJECT_HOME
ant run.client
```

Let us now modify the service
 1. Edit $SERVICE_PROJECT_HOME/com/nestorurquiza/ws/axis2/service/Person.java to include the Address field and accessors.
 1. Edit $SERVICE_PROJECT_HOME/comne/nestorurquiza/ws/axis2/service/DynamicServiceSample.java to include references to 'Address'.
 1. Build and deploy the service:
```
cd $SERVICE_PROJECT_HOME
ant clean generate.service
cp $SERVICE_PROJECT_HOME/build/DynamicServiceSample.aar $TOMCAT_HOME/webapps/axis2/WEB-INF/services
```
 1. Test the web service. You will see firstName, lastName and address comming up.
```
http://localhost:8085/axis2/services/DynamicServiceSample/getPerson?userName=Right%20Said%20Fred
```
 1. Run the below. The client should be still succesfully pulling the information we care about even though the web service is returning new information (Address) as well.
```
cd $CLIENT_PROJECT_HOME
ant run.client
```

Here are some of the advantages of using a Dynamic binding mechanism like in this case we do, using straight AXIOM:
 1. The service can return new nodes as part of the response without need of client recompilation.
 1. The client can add new members (for example middleName) even though the service does not supply a response including them.
 1. If you need just raw XML use the below:
 {{{
 String rawXML = result.getFirstElement().toString()
 }}}
 1. You can always post process/parse the raw XML applying custom rules making even more dynamic the behavior of your clients.

## Consuming dot NET Services

If the End Point is served from a dot NET webservice you might want to take a look at http://thinkinginsoftware.blogspot.com/2009/03/dotnet-web-services-from-axis2.html.

You will need to use JAXB to generate Web Service related class stubs (using jaxb-xjc.jar). The reason is a problem that AXIS2 AXIOM API as I explain in the above post. Basically you use JAXB for deserialization/unmarshalling. 

To generate the JAXB stubs you can use SOAPUI tool (The JAXB artifacts generator can be reached from tools menu) which uses anyway jaxb. You can of course use the command prompt:
```
$JAXB_HOME/bin/xjc.sh -d /a/destination/folder/in/your/computer -p your.package.stub.namespace -wsdl http://your.WSDL.url
```


# References

http://ws.apache.org/axis2/1_4/quickstartguide.html