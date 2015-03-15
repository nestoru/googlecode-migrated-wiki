Spring Portlet MVC Framework is shipped with the official Spring distro. From the samples directory you can access the "petportal" project which uses ant for building purposes. The portal is composed of several portlets =, take a look at portlet.xml there.

I will mavenize, build and deploy in Liferay-on-Glassfish the PetPortal project.

No need to explain why Maven.

I have said before that I like separation of concerns and I believe an IDE should be just to write and debug code. Running a Server inside an IDE makes at least myself slower. I will be using then a remote Liferay Portal running on Glassfish. I will be deploying there the results of the Maven build.

## Mavenize Pet Portal

 1. Download Spring Framework. I will be using 2.5.6.
 1. Run these commands. You need to change the location of the original project sample.
```
PET=/Users/nurquiza/Downloads/spring-framework-2.5.6.SEC01/samples/petportal/
mkdir ~/mavenized-pet-portal
cd ~/mavenized-pet-portal
mkdir -p src/main/java
mkdir -p src/main/resources
mkdir -p src/main/webapp
mkdir -p src/test/java
mkdir -p src/test/resources
cp -R $PET/src/** src/main/java/
cp -R $PET/war/** src/main/webapp/
#vi pom.xml
<?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.nestorurquiza.tutorials</groupId>
	<artifactId>mavenized-pet-portal</artifactId>
	<packaging>war</packaging>
	<version>1.0.0-SNAPSHOT</version>
	<dependencies>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
		    <groupId>commons-fileupload</groupId>
		    <artifactId>commons-fileupload</artifactId>
		    <version>1.2.1</version>
		</dependency>
		<dependency>
		    <groupId>org.springframework</groupId>
		    <artifactId>spring-webmvc-portlet</artifactId>
		    <version>2.5.6</version>
		</dependency>
<!-- Provided dependencies -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.14</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
		    <groupId>portlet-api</groupId>
		    <artifactId>portlet-api</artifactId>
			<scope>provided</scope>
		    <version>1.0</version>
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
			<name>SpringSource Enterprise Bundle Repository - SpringSource Releases</name>
			<url>http://repository.springsource.com/maven/bundles/release</url>
		</repository>
		<repository>
			<id>com.springsource.repository.bundles.external</id>
			<name>SpringSource Enterprise Bundle Repository - External Releases</name>
			<url>http://repository.springsource.com/maven/bundles/external</url>
		</repository>
		<repository>
			<id>com.springsource.repository.bundles.snapshot</id>
			<name>SpringSource Enterprise Bundle Repository	- Snapshot Releases</name>
			<url>http://repository.springsource.com/maven/bundles/snapshot</url>
		</repository>
	</repositories>
	<build>
		<defaultGoal>package</defaultGoal>
		<finalName>mavenized-pet-portal</finalName>
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
					<source>1.6</source>
					<target>1.6</target>
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
					<format>{0,date,yyyy-MM-dd HH:mm:ss}</format>
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
	<scm>
			<connection>scm:svn:http://your.repository.domain/path/to/tags/directory</connection>
			<developerConnection>scm:svn:http://your.repository.domain/path/to/tags/directory</developerConnection>
			<url>scm:svn:http://your.repository.domain/path/to/tags/directory</url>
	</scm>
	<properties>
		<spring.version>2.5.6</spring.version>
	</properties>
</project>
```
 1. Build the project
```
cd ~/mavenized-pet-portal
mvn install
```
 1. Confirm your remote liferay server is up and running. I use an external VirtualBox Ubuntu 64 bits VM with a Glassfish server running Liferay. In my hosts file I have pointed to the at server IP using alias liferay-glassfish-server:
```
http://liferay-glassfish-server:8080
```
 1. Deploy the pet portal (as I said it contains several portlets). I am using scp command to deploy. The Ubuntu-Liferay-Glassfish username is liferay and the remote portlet deployment folder is ~/portlet:
```
scp target/mavenized-pet-portal.war liferay@liferay-glassfish-server:~/liferay-plugin-deploy/
```
 1. Login to Liferay and include one of the portlets for example "Pet Sites" to confirm the deployment went succesfully.
 1. Import or commit your project. I use subversion. Note the target directory and all hidden files are excluded. You do not want specific IDE files for example to be committed to SVN.
 1. Generate IDE specific project files. I will show Eclipse specifics here but you can use any IDE of course.
```
cd ~/mavenized-pet-portal
mvn eclipse:clean
mvn eclipse:eclipse
```
 1. Install the appropriate maven plugin in your IDE. Here are the specifics for Eclipse ... 
 1. Import the project as a Maven project. Be sure option "Menu | Project | Build automatically is set"
 1. From the admin console (http://liferay-glassfish-server:4848/)  Go to Configuration | JVM Settings and check the "Debug Enabled" checkbox. Hit Save. Alternatively from command prompt start as:
```
asadmin start-domain --debug
```
 1. ssh into the Glassfish box and restart the server
 1. Connect to the remote debugging port from Eclipse following these steps: Menu; Run; Debug Configurations; Right click on Remote Java Application; New; name it as liferay-glassfish-server; Host=liferay-glassfish-server; Port=9090. Apply and save. From now on you can click on Debug icon and select liferay-glassfish-server option to connect to Galssfish JVM debug port. Alternatively use command prompt ;-)
```
jdb -attach liferay-glassfish-server:9009
```
 1. Put a breakpoint in line 52 of PetSitesEditController.
 1. From Liferay interface go to the Pet Sites Portlet and click on "Add Site". Eclipse stops execution at the breakpoint.

## Some common tasks

 1. Undeploy the portlet: Login to http://liferay-glassfish-server:4848 and from Applications menu select the portlet to be removed, then press undeploy.


tags:Maven spring mvc portlet tutorial