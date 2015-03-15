# Maven tutorial

I am planning to use Spring Web Flow to make stateful web services. I
was planning to illustrate once again how no-invasive Spring is, while
showing all current behavior from my Spring MVC Convention Over
Configuration tutorial plus new behavior based on Spring Web Flow.

I will show here how to migrate coctutorial project
from Ant build to Maven build. I will use Maven for my upcoming tutorial to be hosted in a separate project called "spring-web-flow-tutorial".

Maven is just a shell script that invokes a Java program. After you install it program you might need to configure an environment variable JAVA_OPTS. In my case (using MAC) I set in ~/,profile:
```
JAVA_OPTS=-Xms128m -Xmx256m -XX:MaxPermSize=256m
```
To get the changes applid in the current session you will need to export the variable:
```
export JAVA_OPTS
```
Look for your OS how to set up the environment variable.

If you use Eclipse go ahead and install Maven plugin for it (Add site http://m2eclipse.sonatype.org/update/)

 1. Below is the struture of the non-maven project which uses Ant for building purposes:
```
|-- build.properties
|-- build.xml
|-- src
|   `-- com
|       `-- nestorurquiza
|           |-- filters
|           |   `-- XssFilter.java
|           |-- spring
|           |   `-- mvc
|           |       |-- view
|           |       |   `-- JSONView.java
|           |       `-- web
|           |           |-- GreetingController.java
|           |           |-- JsonParamsController.java
|           |           |-- LocaleGreetingController.java
|           |           `-- ParamsController.java
|           `-- utils
|               |-- HTMLInputFilter.java
|               |-- Util.java
|               `-- XssRequestWrapper.java
`-- web-app
   |-- WEB-INF
   |   |-- coctutorial-servlet.xml
   |   |-- jboss-web.xml
   |   |-- jsp
   |   |   |-- messageView.jsp
   |   |   `-- paramsView.jsp
   |   |-- lib
   |   |   |-- commons-collections-3.2.jar
   |   |   |-- commons-lang-2.4.jar
   |   |   |-- ezmorph-1.0.6.jar
   |   |   |-- json-lib-2.2.3.jar
   |   |   |-- jstl.jar
   |   |   |-- spring-webmvc.jar
   |   |   |-- spring.jar
   |   |   `-- standard.jar
   |   `-- web.xml
   |-- index.jsp
   |-- messages.properties
   |-- messages_en_GB.properties
   |-- messages_en_US.properties
   |-- messages_es_ES.properties
   `-- static.html
```
 1. Maven has a predefined structure we must fill out with the above files. The target directory will be excluded with svn:ignore. There is where Maven builds the project and we want to store sources **only** on SVN.
 {{{
|-- pom.xml
|-- src
|   |-- main
|   |   |-- java
|   |   |-- resources
|   |   `-- webapp
|   `-- test
|       |-- java
|       `-- resources
`-- target
 }}}
 1. Let us refactor the project then:
```
rm build.properties
rm build.xml
mkdir -p src/main/java
mkdir -p src/main/resources
mkdir -p src/main/webapp
mkdir -p src/test/java
mkdir -p src/test/resources
mv src/com src/main/java/
mv web-app/** src/main/webapp/
rm -fR web-app/
rm -fR src/main/webapp/WEB-INF/lib/
mv src/main/webapp/messages** src/main/resources/
```
 1. Now our maven project has the right structure. Note that anything you put in resources will end up in the classpath under classes folder:
```
.
`-- src
   |-- main
   |   |-- java
   |   |   `-- com
   |   |       `-- nestorurquiza
   |   |           |-- filters
   |   |           |   `-- XssFilter.java
   |   |           |-- spring
   |   |           |   `-- mvc
   |   |           |       |-- view
   |   |           |       |   `-- JSONView.java
   |   |           |       `-- web
   |   |           |           |-- GreetingController.java
   |   |           |           |-- JsonParamsController.java
   |   |           |           |-- LocaleGreetingController.java
   |   |           |           `-- ParamsController.java
   |   |           `-- utils
   |   |               |-- HTMLInputFilter.java
   |   |               |-- Util.java
   |   |               `-- XssRequestWrapper.java
   |   |-- resources
   |   |   |-- messages.properties
   |   |   |-- messages_en_GB.properties
   |   |   |-- messages_en_US.properties
   |   |   `-- messages_es_ES.properties
   |   `-- webapp
   |       |-- WEB-INF
   |       |   |-- coctutorial-servlet.xml
   |       |   |-- jboss-web.xml
   |       |   |-- jsp
   |       |   |   |-- messageView.jsp
   |       |   |   `-- paramsView.jsp
   |       |   `-- web.xml
   |       |-- index.jsp
   |       `-- static.html
   `-- test
       |-- java
       `-- resources
```
 1. It is time to create our project file (pom.xml) just in the root of the project. I am not concern at this point with deployment or releasing tasks. That might be subject of another tutorial. The important things about this project is that we are creating a MANIFEST.MF file with lot of valuable information about the project and that we are managing dependencies from repositories instead of using SVN to store binary jar files. It is a good idea to install your own repository server. I currently use Archiva for that purpose but in this project I am pointing directly to Spring repo. Note how we are satisfying the same dependencies we had before. It might be confusing we are including Spring Web Flow here even though we were not needing that before but the jar comes with both MVC + SWF classes. Note the use of property jboss.deploy.dir from profiles dev-jboss-deploy and dev-jboss deploy-webapp. We will use them to agile deployment of the whole exploded WAR when there is java or webapp changes respectively.
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <groupId>com.nestorurquiza.tutorials</groupId>
       <artifactId>spring-web-flow-tutorial</artifactId>
       <packaging>war</packaging>
       <version>1.0.0-SNAPSHOT</version>
       <dependencies>
         <!--  For JSON -->
         <dependency>
                       <groupId>net.sf.json-lib</groupId>
                       <artifactId>json-lib</artifactId>
                       <version>2.2.3</version>
               </dependency>
               <dependency>
                       <groupId>net.sf.ezmorph</groupId>
                       <artifactId>ezmorph</artifactId>
                       <version>1.0.6</version>
               </dependency>
               <dependency>
                       <groupId>commons-collections</groupId>
                       <artifactId>commons-collections</artifactId>
                       <version>3.2</version>
               </dependency>
               <!-- For Spring MVC + WebFlow -->
               <!-- Compile dependencies -->
               <dependency>
                       <groupId>javax.servlet</groupId>
                       <artifactId>jstl</artifactId>
                       <version>1.2</version>
               </dependency>
               <dependency>
                       <groupId>log4j</groupId>
                       <artifactId>log4j</artifactId>
                       <version>1.2.14</version>
                       <scope>provided</scope>
               </dependency>
               <dependency>
                       <groupId>org.tuckey</groupId>
                       <artifactId>urlrewritefilter</artifactId>
                       <version>3.1.0</version>
               </dependency>
               <dependency>
                       <groupId>org.springframework</groupId>
                       <artifactId>spring-core</artifactId>
                       <version>${spring.version}</version>
               </dependency>
               <dependency>
                       <groupId>org.springframework</groupId>
                       <artifactId>spring-beans</artifactId>
                       <version>${spring.version}</version>
               </dependency>
               <dependency>
                       <groupId>org.springframework</groupId>
                       <artifactId>spring-context</artifactId>
                       <version>${spring.version}</version>
               </dependency>
               <dependency>
                       <groupId>org.springframework</groupId>
                       <artifactId>spring-web</artifactId>
                       <version>${spring.version}</version>
               </dependency>
               <dependency>
                       <groupId>org.springframework</groupId>
                       <artifactId>spring-webmvc</artifactId>
                       <version>${spring.version}</version>
               </dependency>
               <dependency>
                       <groupId>org.springframework.webflow</groupId>
                       <artifactId>spring-webflow</artifactId>
                       <version>${webflow.version}</version>
               </dependency>
               <dependency>
                       <groupId>ognl</groupId>
                       <artifactId>ognl</artifactId>
                       <version>2.6.9</version>
               </dependency>
               <!-- Container-provided dependencies-->
               <dependency>
                       <groupId>javax.servlet</groupId>
                       <artifactId>servlet-api</artifactId>
                       <version>2.4</version>
                       <scope>provided</scope>
               </dependency>
               <dependency>
                       <groupId>javax.servlet.jsp</groupId>
                       <artifactId>jsp-api</artifactId>
                       <version>2.1</version>
                       <scope>provided</scope>
               </dependency>
               <!-- Test dependencies -->
               <dependency>
                       <groupId>junit</groupId>
                       <artifactId>junit</artifactId>
                       <version>4.5</version>
                       <scope>test</scope>
               </dependency>
       </dependencies>
       <repositories>
               <repository>
                       <id>com.springsource.repository.bundles.release</id>
                       <name>SpringSource Enterprise Bundle Repository
- SpringSource
Releases</name>

<url>http://repository.springsource.com/maven/bundles/release</url>
               </repository>
               <repository>
                       <id>com.springsource.repository.bundles.external</id>
                       <name>SpringSource Enterprise Bundle Repository
- External Releases</name>

<url>http://repository.springsource.com/maven/bundles/external</url>
               </repository>
               <repository>
                       <id>com.springsource.repository.bundles.snapshot</id>
                       <name>SpringSource Enterprise Bundle Repository
- Snapshot Releases</name>

<url>http://repository.springsource.com/maven/bundles/snapshot</url>
               </repository>
       </repositories>
       <build>
               <defaultGoal>package</defaultGoal>
                       <finalName>spring-web-flow-tutorial</finalName>
               <extensions>
                       <extension>
                               <groupId>org.apache.maven.wagon</groupId>
                               <artifactId>wagon-webdav</artifactId>
                               <version>1.0-beta-2</version>
                       </extension>
               </extensions>
               <plugins>
                       <plugin>
                               <artifactId>maven-compiler-plugin</artifactId>
                               <version>2.0.2</version>
                               <configuration>
                                       <source>1.5</source>
                                       <target>1.5</target>
                               </configuration>
                       </plugin>
                       <plugin>
                               <groupId>org.codehaus.mojo</groupId>
                               <artifactId>buildnumber-maven-plugin</artifactId>
                               <executions>
                               <execution>
                                       <phase>validate</phase>
                                       <goals>
                                               <goal>create</goal>
                                       </goals>
                                       </execution>
                               </executions>
                               <configuration>
                                       <format>{0,date,yyyy-MM-dd
HH:mm:ss}</format>
                                       <items>
                                       <item>timestamp</item>
                                       </items>
                               </configuration>
                       </plugin>
                       <plugin>
                               <groupId>org.apache.maven.plugins</groupId>
                               <artifactId>maven-war-plugin</artifactId>
                               <version>2.0.2</version>
                               <configuration>
                                       <archive>
                                               <manifest>

<addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                                               </manifest>
                                               <manifestEntries>

<Implementation-Build>${buildNumber}</Implementation-Build>
                                               </manifestEntries>
                                       </archive>
                               </configuration>
                       </plugin>
               </plugins>
       </build>
       <profiles>
               <profile>
                       <id>dev-jboss-deploy-webapp</id>
                       <build>
                               <plugins>
                                       <plugin>

<artifactId>maven-antrun-plugin</artifactId>
                                               <executions>
                                                       <execution>
                                                               <id>e1</id>

<phase>process-resources</phase>
                                                               <goals>

<goal>run</goal>
                                                               </goals>
                                                               <configuration>
                                                                       <tasks>

        <echo message="Copying from ${basedir}/src/main/webapp to
${jboss.deploy.dir}/${pom.build.finalName}.war" />

        <copy todir="${jboss.deploy.dir}/${pom.build.finalName}.war">

                        <fileset includes="**/**.**"
dir="${basedir}/src/main/webapp" />

        </copy>
                                                                       </tasks>
                                                               </configuration>
                                                       </execution>
                                               </executions>
                                       </plugin>
                               </plugins>
                       </build>
               </profile>
               <profile>
                       <id>dev-jboss-deploy</id>
                       <build>
                               <plugins>
                                       <plugin>

<artifactId>maven-antrun-plugin</artifactId>
                                               <executions>
                                                       <execution>
                                                               <id>e1</id>

<phase>install</phase>
                                                               <goals>

<goal>run</goal>
                                                               </goals>
                                                               <configuration>
                                                                       <tasks>

        <echo>Deploying exploded WAR locally</echo>

        <!--    delete
dir="${jboss.deploy.dir}/${pom.build.finalName}.war" / -->

        <exec executable="rm">

                <arg value="-fR" />

                <arg
value="${jboss.deploy.dir}/${pom.build.finalName}.war" />

        </exec>

        <sleep seconds="5" />

        <unzip overwrite="false"
src="target/${pom.build.finalName}.war"
dest="${jboss.deploy.dir}/${pom.build.finalName}.war" />
                                                                       </tasks>
                                                               </configuration>
                                                       </execution>
                                               </executions>
                                       </plugin>
                               </plugins>
                       </build>
               </profile>
               <profile>
                       <id>dev-jboss-deploy-war</id>
                       <build>
                               <plugins>
                                       <plugin>

<artifactId>maven-antrun-plugin</artifactId>
                                               <executions>
                                                       <execution>
                                                               <id>e1</id>

<phase>install</phase>
                                                               <goals>

<goal>run</goal>
                                                               </goals>
                                                               <configuration>
                                                                       <tasks>

        <echo>Deploying WAR locally</echo>

        <copy todir="${jboss.deploy.dir}"
file="target/${pom.build.finalName}.war" />
                                                                       </tasks>
                                                               </configuration>
                                                       </execution>
                                               </executions>
                                       </plugin>
                               </plugins>
                       </build>
               </profile>
       </profiles>
       <properties>
    <spring.version>2.5.6</spring.version>
    <webflow.version>2.0.7.RELEASE</webflow.version>
    <jboss.deploy.dir>/usr/local/jboss-4.0.4.GA/server/default/deploy</jboss.deploy.dir>
 </properties>
</project>
```
 1. If you use Eclipse you want to create now the necessary Eclipse project files which must be svn:ignore later. These files are hidden (.classpath, .settings, .project)
```
mvn eclipse:clean
mvn eclipse:eclipse
```
 1. Build the project from the project root directory
```
mvn install
```
 1. Deploy in your app the resulting WAR file
```
cp cp target/spring-web-flow-tutorial.war
/usr/local/jboss-4.0.4.GA/server/default/deploy/
```
 1. I prefer to use exploded deployment as it allows changes to any file inside webapp to be easy deployed without the need of redeploying the application. So use this to deploy in exploded directory:
```
mvn clean install -Pdev-jboss-deploy antrun:run
```
 1. Use this to deploy just changed webapp components:
```
mvn process-resources -Pdev-jboss-deploy-webapp antrun:run
```
 1. Test same simple URLs as previous tutorial, now just changing the context path as we have changed the name of the project:
```
http://localhost:8080/spring-web-flow-tutorial/
http://localhost:8080/spring-web-flow-tutorial/spring/greeting/hello
http://localhost:8080/spring-web-flow-tutorial/spring/greeting/hi
```
 1. Test internationalization:
```
curl --header "Accept-Language: en-gb"
"http://localhost:8080/spring-web-flow-tutorial/spring/localeGreeting/hello"
Hello Mate !!!
```
 1. Test static pages:
```
http://localhost:8080/spring-web-flow-tutorial/static.html
```
 1. Test plain old JSP with not using Spring at all:
```
http://localhost:8080/spring-web-flow-tutorial/index.jsp
```
 1. Test the site is not XSS vulnerable. As a reminder a Javascript modal popup will be showing up if you remove the XSSFilter from web.xml and hit:
```
http://localhost:8080/spring-web-flow-tutorial/spring/params/write?par=XssAttack%3CSCRIPT%3Ealert(String.fromCharCode(88,83,83))%3C/SCRIPT%3E
```
 1. Test the simple JSON Service:
```
http://localhost:8080/spring-web-flow-tutorial/spring/jsonParams/write?p1=foo&p2=bar&callback
```

## SVN for maven projects

Be sure you follow the below steps when committing to an SVN repository
 1. Delete the target directory and any hidden file (those starting with a dot). Hidden directories would be Eclipse or other IDE related files which should not be committed to SVN. Commonly you should prepare your SVN before building with IDEs but if you don't just delete the files to commit and later you can run "mvn eclipse:eclipse" for example as explained before.
```
rm -fR <project-directory>/target
rm -fR <project-directory>/.**
```
 1. Import the project into the SVN repo:
```
svn import <project-directory> <svn-base-url>/<project-directory> -m "first import"
```
 1. Ignore the target and hidden directories
```
svn propset svn:ignore target .
svn propset svn:ignore ".**" .
```
