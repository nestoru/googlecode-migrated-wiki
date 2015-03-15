This project (proof of concept) is hosted at http://nestorurquiza.googlecode.com/svn/trunk/drools-data-transformation/


## Setting up your environment

Note the versions I have used for each package. I have tried with several combinations and the below does work for me at least on MAC OSX Snow Leopard.

 1. Install in your local or remote maven repository some needed dependencies. Showing local installation:
 {{{
mvn install:install-file -DgroupId=org.drools -DartifactId=drools-core -Dversion=5.1.0.M2 -Dpackaging=jar -Dfile=/Users/nestor/drools-5.1.0.M2-bin/drools-core-5.1.0.M2.jar 
mvn install:install-file -DgroupId=org.drools -DartifactId=drools-compiler -Dversion=5.1.0.M2 -Dpackaging=jar -Dfile=/Users/nestor/drools-5.1.0.M2-bin/drools-compiler-5.1.0.M2.jar 
mvn install:install-file -DgroupId=org.drools -DartifactId=drools-api -Dversion=5.1.0.M2 -Dpackaging=jar -Dfile=/Users/nestor/drools-5.1.0.M2-bin/drools-api-5.1.0.M2.jar 
 }}}

## Running the project

Below is how to run the two included examples as well as the output for them:
```
#simple
#mvn exec:java -Dexec.mainClass="com.nestorurquiza.drools.DroolsDataTransformationTest" -Dexec.classpathScope=runtime
[[exec:java {execution: default-cli}](INFO])
Feed [FeedRow [fields={b=35.5, c=5.325, a=The Crab House}]]](feedRows=[[Feed Row|[fields={b=25.6, c=3.84, a=Red Lobster}]],)

#complex
#mvn exec:java -Dexec.mainClass="com.nestorurquiza.drools.DroolsComplexDataTransformationTest" -Dexec.classpathScope=runtime
[[exec:java {execution: default-cli}](INFO])
Feed [FeedRow [fields={d=7.1000001057982445, b=35.5, c=Miami Beach, a=The Crab House}]]](feedRows=[[Feed Row|[fields={d=3.8400001525878906, b=25.6, c=Miami, a=Red Lobster}]],)
```

I am providing two Test classes. In the so called "complex" example you see I am applying the rules per row. The reason is I want to affect a Domain object (OutputFeed) using one or more rules that apply given the nature of the input feed fields. The nature will then determine what value we use for the destination feed fields. The "Any Location" rule is applied to all records, "Miami Beach tips" is applied only to restaurants located in Miami and then "Non Miami Beach Tips" will be applied to the rest.

The "Simple" example uses rules that apply same tip for all locations.
The "Complex" example uses rules that apply different tips depending on the location.

## Using Eclipse IDE

 1. Install Eclipse. I have used latest Eclipse release: Helios.
 1. Install Maven plugin. Install new software from Help menu and point to http://m2eclipse.sonatype.org/sites/m2e
 1. Download Drools Eclipse plugin. I have used Drools 5.1.0.M2 (http://download.jboss.org/drools/release/5.1.0.33002.M2/drools-5.1.0.M2-eclipse-all.zip). Uncompress the file and copy the features and plugins files into the corresponding directory in the Eclipse home directory.
 1. If you have not pick an Eclipse workspace then open Eclipse and create a new one. Close Eclipse.
 1. From command prompt prepare the project:
 {{{
svn co http://nestorurquiza.googlecode.com/svn/trunk/drools-data-transformation/
cd drools-data-transformation
mvn eclipse:clean eclipse:eclipse
 }}}
1. Set M2_REPO variable. Note you need to use your own eclipse workspace folder
 {{{
mvn -Declipse.workspace=/Users/nestor/eclipse-workspace-test eclipse:add-maven-repo
 }}}
 1. Open Eclipse. If it was opened close it and open it again.
 1. From Eclipse import the project into the workspace: Right click on left pane select Import | General | Existing Projects into workspace
 1. Right click on the project and select Maven | Enable Dependency Management
 1. Open the drl files in src/main/resources. Put breakpoints in the "then" sections.
 1. Right click on com.nestorurquiza.drools.DroolsDataTransformationTest. Choose "Debug As | Drools Project" and the program will stop at one of the breakpoints. Inspect variables as usual. Do the same for the other class com.nestorurquiza.drools.DroolsComplexDataTransformationTest. You will get the same result you got when running the classes from the command prompt.

## Non Maven projects

I prefer to use Maven of course and I have written before why.

If you are not using Maven your project will have a slightly different structure. You can always convert the project to a Drools project using right click on top of the project entry.
 
 1. Download Drools (hhttp://download.jboss.org/drools/release/5.1.0.33002.M2/drools-5.1.0.M2-bin.zip) and unzip the file let us say in ~/drools-5.1.0.M2-bin. 
 1. Start Eclipse and select New | Project | Drools. Enter a name for the project. Press "next" twice. Click on "Configure Workspace Settings" and provide the path for drools-5.1.0.M2-bin directory. Click on "finish"
 {{{
Name: drools-5.1.0.M2
Path: /Users/nestor/drools-5.0-bin
 }}}
 1. You will need to restart Eclipse after configuring the Drools runtime for changes to take effect.
 1. At this point you can step through the code as we have explained before. Of course your project will have too many dependency jars that will need to be kept in the version control system which is one of the main reasons I prefer to use Maven (I keep just source files in the repo)