The Liferay plugin SDK provides a quick way to build JSP portlets. It would be ideal future versions of it to come with Maven builds besides the ant builds it is shipped with. As of today Maven is still officially missing from Liferay distributions.

Here I show how to mavenize a JSP portlet created from the Liferay plugin SDK. I will also show how to access the Document Library API directly from JSP. Form submission is also shown.

There are several assumptions in terms of OS paths, so be aware you will need to review them or use the same paths I use. I will not talk about the configuration of the plugin SDK as I have written about that before.

The source code for this tutorial is available from http://nestorurquiza.googlecode.com/svn/trunk/mavenized-jsp-portlet

 1. Create the JSP Portlet
 {{{
PORTLET=/Users/nurquiza/liferay-components/liferay-plugins-sdk-5.2.3/portlets/to-be-mavenized-jsp-portlet
cd ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets
./create.sh to-be-mavenized-jsp "Mavenized JSP Portlet"
 }}}
 1. Mavenize the JSP Portlet
 {{{ 
mkdir ~/mavenized-jsp-portlet
cd ~/mavenized-jsp-portlet
mkdir -p src/main/java
mkdir -p src/main/resources
mkdir -p src/main/webapp
mkdir -p src/test/java
mkdir -p src/test/resources
cp -R $PORTLET/docroot/** src/main/webapp/
mv src/main/webapp/WEB-INF/src/** src/main/java/
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
<!-- Provided dependencies -->
		<dependency>
			<groupId>portlet-api</groupId>
			<artifactId>portlet-api</artifactId>
			<version>1.0</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>com.liferay</groupId>
			<artifactId>portal-service</artifactId>
			<version>5.2.3</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>com.liferay</groupId>
			<artifactId>portal-kernel</artifactId>
			<version>5.2.3</version>
			<scope>provided</scope>
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
			<name>SpringSource Enterprise Bundle Repository - Snapshot Releases</name>
			<url>http://repository.springsource.com/maven/bundles/snapshot</url>
		</repository>
	</repositories>
	<build>
		<defaultGoal>package</defaultGoal>
		<finalName>mavenized-jsp-portlet</finalName>
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
</project>
 }}}
 1. Install needed dependencies in your Maven repository (Showing commands for Archiva):
 {{{
cd ~/liferay-components/liferay-portal-ext-5.2.3/lib/global
mvn deploy:deploy-file \
	-DgroupId=com.liferay \
	-DartifactId=portal-service \
	-Dversion=5.2.3 \
	-Dpackaging=jar \
	-Dfile="portal-service.jar" \
	-Durl=http://archiva-repo-domain/archiva/repository/internal/ \
	-DrepositoryId=archiva.internal

mvn deploy:deploy-file \
	-DgroupId=com.liferay \
	-DartifactId=portal-kernel \
	-Dversion=5.2.3 \
	-Dpackaging=jar \
	-Dfile="portal-kernel.jar" \
	-Durl=http://archiva-repo-domain/archiva/repository/internal/ \
	-DrepositoryId=archiva.internal
 }}}
 1. If using Artifactory the above will look like: 
 {{{
 ...
 -Durl=http://artifactory-repo-domain/artifactory/libs-releases-local \
 -DrepositoryId=libs-releases-local
 ...
 }}}
 1. Alternatively install them locally (If you do not have a Maven repo which is strongly recommended)
 {{{
cd ~/liferay-components/liferay-portal-ext-5.2.3/lib/global
mvn install:install-file -Dfile=portal-service.jar \
  -DgroupId=com.liferay \
  -DartifactId=portal-service \
  -Dversion=5.2.3 \
  -Dpackaging=jar

mvn install:install-file -Dfile=portal-kernel.jar \
  -DgroupId=com.liferay \
  -DartifactId=portal-kernel \
  -Dversion=5.2.3 \
  -Dpackaging=jar
 }}}
 1. Build the project
 {{{
cd ~/mavenized-jsp-portlet/
mvn install
 }}}
 1. Deploy your portlet. In my case I use a remote Glassfish Server. You can read previous tutorials to get more information about Glassfish hosting Liferay.
 {{{
scp target/mavenized-jsp-portlet.war liferay@liferay-glassfish-server:~/liferay-plugin-deploy/
 }}}
 1. Login to Liferay and include the portlet "Mavenized JSP Portlet" to confirm the deployment went succesfully.
 1. Edit view.jsp to show some useful information
 {{{
#vi src/main/webapp/view.jsp 
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>
<%@ taglib uri="http://liferay.com/tld/theme" prefix="liferay-theme" %>
<%@ taglib uri="http://liferay.com/tld/ui" prefix="liferay-ui" %>
<%@ page import="com.liferay.portlet.documentlibrary.service.DLFolderServiceUtil" %>
<%@ page import="com.liferay.portlet.documentlibrary.model.DLFolder" %>
<%@ page import="com.liferay.portal.kernel.util.ParamUtil" %>

<portlet:defineObjects />
<liferay-theme:defineObjects />

<%
long groupId = layout.getGroupId();
java.util.List<DLFolder> folders = DLFolderServiceUtil.getFolders(groupId, 0);
String phrase = ParamUtil.getString(request, "phrase");
if(phrase == null || phrase.length() == 0){
  phrase = "Nothing";
}
%>

<h3>User Data</h3>
<div><liferay-ui:user-display userId="<%= user.getUserId() %>" /></div>
<div>Company: <%=company.getCompanyId()%> </div>
<div>Group: <%=groupId%> </div>
<div>Folders: <%=folders%> </div>

<h3>What you said</h3>
<%=phrase%>

<h3>Form Submission</h3>
<form name="testForm" action="<portlet:renderURL/>" method="post">
  <input type="text" name="phrase" value="<%=phrase%>">
  <input type="submit" name="submit" value="submit">
</form>
 }}}
 1. Build deploy and then refresh the page in Liferay to get the Data rendered.
 {{{
mvn install
scp target/mavenized-jsp-portlet.war liferay@liferay-glassfish-server:~/liferay-plugin-deploy/
 }}}

## Remarks

While developing just using JSP is not a good approach as it violates the MVC pattern it is a quick and dirty way to get things done with a minimum effort and little learning curve. 

I would like to keep on including features especially related to the Liferay Document Library API. Unfortunately my spare time is getting to an end for that. Some useful information about Document Library API:

 1. Liferay API can be accessed from plugin portlets (plugin SDK instead of Extension environment development). The way to do so is including and using the functionality included in two jar files (portal-service.jar and portal-kernel.jar). Those are the files this Maven project includes.
 1. Take a look at http://docs.liferay.com/portal/5.2/javadocs/portal-service/ and http://docs.liferay.com/portal/5.2/javadocs/portal-kernel/for available APIs
 1. portal-service has all we need to access the Document Library API (com.liferay.portlet.documentlibrary package).