 1. Here is the software to download:
```
http://www.eng.lsu.edu/mirrors/apache/tomcat/tomcat-connectors/jk/binaries/win32/jk-1.2.30/mod_jk-1.2.30-httpd-2.2.3.so
http://mirror.olnevhost.net/pub/apache/httpd/binaries/win32/httpd-2.2.15-win32-x86-openssl-0.9.8m-r2.msi
```
 1. Install Apache and your java application server.
 1. Copy mod_jk-1.2.30-httpd-2.2.3.so to C:\Program Files\Apache Software Foundation\Apache2.2\modules
 1. Create workers.properties in C:\Program Files\Apache Software Foundation\Apache2.2\conf
 {{{
#default (standalone workers)
worker.default1.type=ajp13
worker.default1.host=localhost
worker.default1.port=8009

worker.maintain=20
worker.list=default1
 }}}
 1. Edit httpd.conf from C:\Program Files\Apache Software Foundation\Apache2.2\conf 
 {{{
...
NameVirtualHost **
LoadModule rewrite_module modules\mod_rewrite.so
Include conf/mod-jk.conf
Include conf/vhosts.conf
 }}}
 1. Create mod-jk.conf in C:\Program Files\Apache Software Foundation\Apache2.2\conf 
 {{{
LoadModule jk_module modules\mod_jk-1.2.30-httpd-2.2.3.so

#Workers file location
JkWorkersFile conf\workers.properties

#mod log location
JkLogFile logs\mod-jk.log

#Set log level
JkLogLevel trace 

# Add shared memory.
# This directive is present with 1.2.10 and
# later versions of mod_jk, and is needed for
# for load balancing to work properly
JkShmFile logs\apache2\jk.shm 
 }}}
 1. Create vhosts.conf in C:\Program Files\Apache Software Foundation\Apache2.2\conf
 {{{
<VirtualHost **>
    ServerName sample.com

    DocumentRoot "c:\docroot\sample.war"

    <Directory "c:\docroot\sample.war">
        Options +FollowSymLinks
        AllowOverride All
        Order deny,allow
        Allow from all
        RewriteEngine on
        RewriteRule ^(.**)$ /sample/$1
    </Directory>

     ErrorLog logs\sample.com.log
     CustomLog logs\sample.com.log common

     JkMount "/sample" default1
     JkMount "/sample/**" default1

 </VirtualHost>
 }}}
 1. Verify the syntax is OK and correct any reported errors"
 {{{
C:\Program Files\Apache Software Foundation\Apache2.2\bin>httpd.exe -t
 }}}
 1. Restart apache. Find errors if any at C:\Program Files\Apache Software Foundation\Apache2.2\logs and correct them.
 1. At this point if the application the application server is running, the context exists and the AJP port is configured as 8009 you should get your home page on a browser after hitting http://sample.com/ 