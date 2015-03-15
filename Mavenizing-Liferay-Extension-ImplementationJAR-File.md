In this tutorial I show how I mavenized the Liferay extension implementation JAR file (ext-impl.jar).

 1. Install necessary dependencies. Below I am showing the local installation. You should have your own company maven repository (Archiva or Artifactory can be used for example). Note that ext-service should be mavenized and make this project depending on it but for now I assumed you built it with ant.
```
cd ~/liferay-components/liferay-portal-ext-5.2.3/modules
mvn install:install-file -Dfile=portal-impl.jar -DgroupId=com.liferay -DartifactId=portal-impl -Dversion=5.2.3 -Dpackaging=jar
cd ~/liferay-components/liferay-portal-dependencies-5.2.3
mvn install:install-file -Dfile=portal-service.jar -DgroupId=com.liferay -DartifactId=portal-service -Dversion=5.2.3 -Dpackaging=jar
cd ~/liferay-components/liferay-portal-dependencies-5.2.3
mvn install:install-file -Dfile=portal-kernel.jar -DgroupId=com.liferay -DartifactId=portal-kernel -Dversion=5.2.3 -Dpackaging=jar
cd ~/liferay-components/liferay-portal-ext-5.2.3/ext-service
mvn install:install-file -Dfile=ext-service.jar -DgroupId=com.liferay -DartifactId=ext-service -Dversion=5.2.3 -Dpackaging=jar
```
 1. Mavenize project
```
mkdir ~/mavenized-ext-impl
cd ~/mavenized-ext-impl
mkdir -p src/main/java
mkdir -p src/main/resources
mkdir -p src/test/java
mkdir -p src/test/resources
cp -R ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/src/META-INF/ src/main/resources
cp -R ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/src/com src/main/java/
#vi pom.xml 
<?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.nestorurquiza.tutorials</groupId>
	<artifactId>mavenized-ext-impl</artifactId>
	<name>ext-impl</name>
	<version>1.0.0-SNAPSHOT</version>
	<description/>
	<dependencies>
<!-- Provided dependencies -->
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
		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.1.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>struts</groupId>
			<artifactId>struts</artifactId>
			<version>1.2.9</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.portlet</groupId>
			<artifactId>portlet-api</artifactId>
			<version>2.0</version>
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
		<dependency>
			<groupId>com.liferay</groupId>
			<artifactId>portal-impl</artifactId>
			<version>5.2.3</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>com.liferay</groupId>
			<artifactId>ext-service</artifactId>
			<version>5.2.3</version>
			<scope>provided</scope>
		</dependency>
	</dependencies>
	<build>
		<finalName>ext-impl</finalName>
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
					<encoding>UTF-8</encoding>
					<verbose>false</verbose>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-release-plugin</artifactId>
				<version>2.0-beta-7</version>
				<configuration>
					<tagBase>
						http://your.repository.domain/path/to/tags
		                        </tagBase>
				</configuration>
			</plugin>
		</plugins>
	</build>
	<scm>
		<connection>scm:svn:http://your.repository.domain/path/to/trunk-or-branch/directory</connection>
                <developerConnection>scm:svn:http://your.repository.domain/path/to/trunk-or-branch/directory</developerConnection>
                <url>http://your.repository.domain/path/to/trunk-or-branc/directory</url>
	</scm>
</project>
```
 1. See the final result here http://nestorurquiza.googlecode.com/svn/trunk/customizing-liferay/mavenized-ext-impl/
 1. Now you can deploy it to your server like:
```
cp ext-impl.jar ~/glassfishv3/glassfish/domains/liferay-domain/applications/liferay-portal-5.2.3/WEB-INF/lib
```
 1. Of course you will need to restart your server.