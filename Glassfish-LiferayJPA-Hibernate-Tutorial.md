## Before you start

If you are using like I am a VM with 64bits Ubuntu Server or any other GUI-less Server you will need to ssh into the box using X11 forwarding:
```
ssh -X ubuntu-server
```

That way you will be able to get the GUI back on your local XServer. And you definitely need the GUI for glassfish installation and for using updatetool command.

Of course to be able to interact with the server you will need to use Bridging or any other technique from your Virtualization software (I use Virtual Box) that will allow the VM to run with its own IP.

In addition all http://locahost references below should be replaced by http://liferay where liferay will be an entry in your hosts file pointing to the Server VM IP.

JSP pre-compilation from Glassfish Liferay deployment has worked for me on MAC OSX (32 and 64 bits) and on Ubuntu 64 bits so far. Ubuntu 32 bits will work but you better leave (as I recommend below) the "compile jsp" option unchecked.

At least on MAC OSX and Ubuntu Jaunty the zip file (CLI) will fail to accept Liferay WAR throwing actually different exceptions on both OS. 

Apparently there are switches to make the binary installer work with an external configuration file which I have not tested. 

If you need to install Glassfish in a remote Linux server you can use VNC or SSH X11 forwarding. If you are connecting from Windows you can use XMing as XServer and Putty with X11 enabled.

In any case this tutorial is about installing from the binary file which so far I have tested **only** from a GUI installation.

## Putting all together

 1. Download Glassfish binary (GUI). For Windows and Mac here it is: http://download.java.net/glassfish/v3/release/glassfish-v3-unix.sh 
 1. Install Glassfish running the below command and select as destination directory ~/glassfishv3v3. Below is the GUI command for the release version.
```
chmod +x glassfish-v3-unix.sh
./glassfish-v3-unix.sh
```
 1. Locate bin/updatetool program (GUI) inside the installation directory, run it and select "Available Add-ons". Install Hibernate JPA:
```
~/glassfishv3/bin/updatetool
```
 1. Create liferay domain. You will be prompted for master and admin password.
```
~/glassfishv3/bin/asadmin create-domain --adminport 4848 liferay-domain
```
 1. Get liferay components. For this tutorial I am using Liferay version 5.2.3:
```
mkdir ~/liferay-components
cd ~/liferay-components
wget http://downloads.sourceforge.net/lportal/liferay-portal-ext-5.2.3.zip
wget http://downloads.sourceforge.net/lportal/liferay-portal-5.2.3.war
wget http://downloads.sourceforge.net/lportal/liferay-portal-dependencies-5.2.3.zip
wget http://downloads.sourceforge.net/lportal/liferay-plugins-sdk-5.2.3.zip
wget http://downloads.sourceforge.net/lportal/liferay-portal-src-5.2.3.zip
unzip liferay-portal-ext-5.2.3.zip
unzip liferay-portal-src-5.2.3.zip
unzip liferay-plugins-sdk-5.2.3.zip -d liferay-plugins-sdk-5.2.3
unzip liferay-portal-dependencies-5.2.3.zip
```
 1. Copy dependencies to Glassfish Liferay Domain
```
cp ~/liferay-components/liferay-portal-dependencies-5.2.3/** ~/glassfishv3/glassfish/domains/liferay-domain/lib/
cp ~/liferay-components/liferay-plugins-sdk-5.2.3/lib/** ~/glassfishv3/glassfish/domains/liferay-domain/lib/
cp ~/liferay-components/liferay-portal-ext-5.2.3/lib/development/mysql.jar ~/glassfishv3/glassfish/domains/liferay-domain/lib/
```
 1. Ensure the JVM is running with enough Perm Space (You might need more than the 192 enough in my case)
```
#vi ~/glassfishv3/glassfish/domains/liferay-domain/config/domain.xml
<jvm-options>-XX:MaxPermSize=192m</jvm-options>
<jvm-options>-Xmx512m</jvm-options>
```
 1. Start the server. The sample page (domains/liferay-domain/docroot/index.html) should be accessible from http://localhost:8080/
```
~/glassfishv3/bin/asadmin start-domain liferay-domain
```
 1. Login to the admin console (hit http://localhost:4848)
 1. Navigate to Applications/Web Applications and hit deploy. Point to absolute path, in my case /Users/nurquiza/liferay-components/liferay-portal-5.2.3.war.  Use local packaged file option if the file is local to the server. Use "liferay-portal-5.2.3" as application name. Select root (/) as "Context Root". Do **not** select precompile JSPs option. If you pick JSP pre-compilation you might get missing jar files errors. In my case running OSX 64 bits it compiled without problems however in a Ubuntu Jaunty 32 bits box it was impossible to get it working with pre-compilation enabled.
 1. After the deployment finishes hitting http://localhost:8080 should bring Liferay login page.
 1. If you take a look at the log file you will see Hypersonic Database is being used. Let us change that for MySQL:
```
Using dialect org.hibernate.dialect.HSQLDialect
```
 1. I had a local MySQL Liferay Database in place but if you do not then download the sql scripts and use sql/create-minimal or sql/create scripts.
```
wget http://downloads.sourceforge.net/lportal/liferay-portal-sql-5.2.3.zip
unzip liferay-portal-sql-5.2.3.zip
mysql -u root -p<liferay-password> < liferay-portal-sql-5.2.3/create-minimal/create-minimal-mysql.sql
mysql> CREATE USER 'liferay'@'localhost' IDENTIFIED BY '<liferay-password>';
mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP ON lportal.** TO 'liferay'@'localhost';
```
 1. Navigate to Resources/JDBC/Connection Pools and create a new one. In Prelude there is a bug that will cause a 500 if "Database Vendor" is selected so leave it blank:
```
name: LiferayPool
Resource Type: javax.sql.ConnectionPoolDataSource
Datasource Classname: com.mysql.jdbc.jdbc2.optional.MysqlConnectionPoolDataSource
#Add properties
url: jdbc:mysql://localhost:3306/lportal?useUnicode=true&amp;characterEncoding=UTF-8&amp;useFastDateParsing=false
Url: jdbc:mysql://localhost:3306/lportal?useUnicode=true&amp;characterEncoding=UTF-8&amp;useFastDateParsing=false
#Add additional Properties
username: liferay
password: <liferay-password>
#Click Finish
```
 1. Click on the LiferayPool link. Click on "ping" to be sure the connection works.
 1. Navigate to Go to Resources/JDBC/JDBC Resources and create a new one:
```
JNDI Name: jdbc/LiferayPool
Pool Name: LiferayPool
```
 1. Configure the extension environment. Change the below for your local directory structure. Note that we create a new file in this step instead of modifying app.server.properties directly (in my case app.server.nurquiza.properties). Note you will need to change app.server.parent.dir.
```
#vi ~/liferay-components/liferay-portal-ext-5.2.3/app.server.`whoami`.properties
app.server.type=glassfish
app.server.parent.dir=/Users/nurquiza/glassfishv3
app.server.glassfish.instance.dir=${app.server.glassfish.dir}/domains/liferay-domain
app.server.glassfish.portal.dir=${app.server.parent.dir}/glassfish/domains/liferay-domain/applications/liferay-portal-5.2.3
app.server.glassfish.version=3
app.server.glassfish.zip.name=glassfish-v3.zip
app.server.glassfish.zip.url=http://download.java.net/glassfish/v3/release/${app.server.glassfish.zip.name}
```
 1. Edit extension properties to point to LiferayPool
```
#vi ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/src/portal-ext.properties
jdbc.default.jndi.name=jdbc/LiferayPool
```
 1. Deploy properties
```
cd ~/liferay-components/liferay-portal-ext-5.2.3/
ant deploy-properties
```
 1. Stop the server:
```
~/glassfishv3/bin/asadmin stop-domain liferay-domain
```
 1. If the stop command fails make sure the login information is stored locally.
```
~/glassfishv3/bin/asadmin login -p 4848
```
 1. Start Glassfish
```
~/glassfishv3/bin/asadmin start-domain liferay-domain
```
 1. Hit http://localhost:8080. Login with test@liferay.com/test (default Liferay admin account).
 1. From logs you should be able to see how previous Hypersonic DB is now replaced by MySQL:
```
$ grep org.hibernate.dialect /home/liferay/glassfishv3/glassfish/domains/liferay-domain/logs/server.log 
[INFO  [DialectDetector:97](#|2010-03-18T07:45:10.507-0400|INFO|glassfishv3.0|javax.enterprise.system.std.com.sun.enterprise.v3.services.impl|_ThreadID=29;_ThreadName=Thread-1;|07:45:10,506) Using dialect org.hibernate.dialect.HSQLDialect
[INFO  [DialectDetector:97](#|2010-03-18T09:11:09.459-0400|INFO|glassfishv3.0|javax.enterprise.system.std.com.sun.enterprise.v3.services.impl|_ThreadID=11;_ThreadName=Thread-1;|09:11:09,458) Using dialect org.hibernate.dialect.MySQLDialect
```
 1. Select from the dock menu "Add Application", then click on "Install more Applications". The "Plugin Installer" page should show up. Click on "Configuration tab and set your preferred plugin deployment diretory, in my case I picked "/Users/nurquiza/liferay-plugin-deploy". Click on "Save".
 1. Configure the plugin environment. Note that a specific file for your user name is recomended instead of editing directly build.properties:
```
#vi ~/liferay-components/liferay-plugins-sdk-5.2.3/build.`whoami`.properties
app.server.dir=/Users/nurquiza/glassfishv3/glassfish
app.server.lib.global.dir=${app.server.dir}/domains/liferay-domain/lib
app.server.lib.portal.dir=${app.server.lib.global.dir}
app.server.portal.dir=${app.server.dir}/domains/liferay-domain/applications/liferay-portal-5.2.3
auto.deploy.dir=/Users/nurquiza/liferay-plugin-deploy
```
 1. Create a simple plugin to test jndi datasources are working:
```
cd ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets/
./create.sh jsp_jndi_test "JSP JNDI Tester Portlet"
```
 1. Edit view.jsp:
```
#vi jsp_jndi_test-portlet/docroot/view.jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%@ page import="java.sql.**" %>
<%@ page import="javax.naming.**" %>
<%@ page import="javax.sql.**" %>


<%
Connection conn = null;
try{
       String jndi = "jdbc/LiferayPool";
       out.println("<br/>jndi: " + jndi);
       InitialContext initialContext = new InitialContext();
       DataSource datasource = (DataSource) initialContext.lookup(jndi);
       conn = datasource.getConnection();
       out.println("<br/>Catalogue : " + conn.getCatalog());
       out.println("<br/>Product Name: " +
conn.getMetaData().getDatabaseProductName());
} catch (Exception e) {
       out.println("<br/>" + e.getStackTrace().toString());
} finally {
       if (conn != null && !conn.isClosed()) {
               conn.close();
       }
}
%>
```
 1. Deploy the portlet
```
cd ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets/jsp_jndi_test-portlet
ant deploy
```
 1. Go to Liferay and include the portlet ("Add Application", search for "jndi" and click on "Add" for "JSP JNDI Tester Portlet"
 1. If you run into troubles examine the logs, correct any issues, redeploy and refresh the home page (http://localhost:8080/web/guest/home):
```
tail -100f ~/glassfishv3/glassfish/domains/liferay-domain/logs/server.log
```
 1. Create another portlet to test JPA with Hibernate
```
cd ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets/
./create.sh jsp_jpa_hibernate_test "JPA Hibernate Tester Portlet"
```
 1. Edit web.xml:
```
#vi ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets/jsp_jpa_hibernate_test-portlet/docroot/WEB-INF/web.xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/20
01/XMLSchema-instance" xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http:
//java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" version="2.4">

    <persistence-unit-ref>
    <persistence-unit-ref-name>liferayJPAHibernatePersistenceUnit/pu</persistence-unit-ref-name>
    <persistence-unit-name>liferayJPAHibernatePersistenceUnit</persistence-unit-name>
  </persistence-unit-ref>
</web-app>
```
 1. Create persistence.xml as per JPA spec
```
#mkdir -p ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets/jsp_jpa_hibernate_test-portlet/docroot/WEB-INF/classes/META-INF
#vi ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets/jsp_jpa_hibernate_test-portlet/docroot/WEB-INF/classes/META-INF/persistence.xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_1_0.xsd"
   version="1.0">
	<persistence-unit name="liferayJPAHibernatePersistenceUnit" transaction-type="RESOURCE_LOCAL">
	<provider>org.hibernate.ejb.HibernatePersistence</provider>
	<jta-data-source>jdbc/LiferayPool</jta-data-source>
        <properties>
		<property name="hibernate.connection.datasource" value="jdbc/LiferayPool" />
          	<property name="hibernate.connection.release_mode" value="after_transaction" />
           	<property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect" />
           	<property name="hibernate.hbm2ddl.auto" value="true" />
           	<property name="hibernate.cache.use_query_cache" value="false" />
           	<property name="hibernate.show_sql" value="true" />
		<property name="hibernate.format_sql" value="false" />
        </properties>
    </persistence-unit>
</persistence>
```
 1. Edit ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets/jsp_jpa_hibernate_test-portlet/docroot/view.jsp
```
#vi jsp_jpa_hibernate_test-portlet/docroot/view.jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="javax.persistence.EntityManager" %>
<%@ page import="javax.persistence.EntityManagerFactory" %>
<%@ page import="javax.persistence.Persistence" %>

<%
EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("liferayJPAHibernatePersistenceUnit");
EntityManager entityManager = entityManagerFactory.createEntityManager();
out.println("<br/>entityManager : " + entityManager);
%>
```
 1. Deploy the portlet
```
cd ~/liferay-components/liferay-plugins-sdk-5.2.3/portlets/jsp_jpa_hibernate_test-portlet/
ant deploy
```
 1. Go to Liferay and include the portlet ("Add Application", search for "jpa" and click on "Add" for "JPA Hibernate Tester Portlet"