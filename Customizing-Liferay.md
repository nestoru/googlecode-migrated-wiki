Here is a playground with some tips about customizing Liferay. The resulting code can be seen from the "Source" link above. See also [[Liferay Tips And Tricks|Liferay Tips And Tricks]]

 1. Change favicon.ico: In reality version 5.2.3 and before use liferay.ico instead. Change it from your theme, deploy it, take a look at the bottom section named "Disabling cache in development environments" and struggle with your browser cache ;-)
```
cp favicon.ico ~/liferay-components/liferay-plugins-sdk-5.2.3/themes/005E82-theme/docroot/_diffs/images/liferay.ico
```
 1. Change the Liferay Logo by your company logo: Just go to Control Pannel; Portal; Settings; Display Settings. Click on Change and point to your png image; Save and click again on Display settings to verify the logo was changed. You can allow community administrators to change the new logo or not from that page.
 1. Create a custom theme project. I am creating one that will customize Liferay to show a predominant blue (005E82) color
```
cd liferay-plugins-sdk-5.2.3
./create.sh 005E82 "Predominant 005E82 r0g94b130"
cd  005E82-theme
```
 1. Copy necessary folders form an existing theme. Below I use the classic theme.
```
cp -R /Users/nestor/liferay/src/portal-web/docroot/html/themes/classic/_diffs/* docroot/_diffs/
```
 1. Make sure absolutely all changes are done in the _diffs directory
 1. Update custom.css to adapt to desired background and border colors (in this case 005E82)
 1. Update navigation.css to adjust the menu look and feel. For example you might want to get rid of the dock menu button image. For that matter you will need create a transparent 1px square png and drop it in _diffs/images/dock/menu_bar.png
 1. You for sure need a footer and the code for it can be inserted in _diffs/templates/portal_normal.vm
 1. You can always compare current theme with the one you started it from:
 {{{
diff -r /Users/nestor/liferay//src/portal-web/docroot/html/themes/classic/_diffs/css docroot/_diffs/css
 }}}
 1. Take a look at my results below. I have changed the base color and eliminated some background images:
 {{{
***********
custom.css
***********
15d14
< 	background: transparent url(../images/common/body_bg.png) repeat-x 0 0;
26d24
< 	background: transparent url(../images/common/banner_bg.jpg) no-repeat 20% 0;
51,53c49,50
< 	background: url(../images/dock/my_places_public.png) no-repeat 15px 50%;
< 	border-left: 1px solid #D3DADD;
< 	color: #D3DADD;
---
> 	border-left: 1px solid #005E82;
> 	color: #000;
68d64
< 	background-image: url(../images/dock/my_places_private.png);
74,75c70,71
< 	background: #020509;
< 	border-top: 1px solid #304049;
---
> 	background: #005E82;
> 	border-top: 1px solid #005E82;
87c83
< 	border: 1px solid #304049;
---
> 	border: 1px solid #005E82;
94c90
< 	color: #D3DADD;
---
> 	color: #fff;
104a101
>         color: #fff;
108c105
< 	background-color: #1E2529;
---
> 	background-color: #005E82;
118c115
< 	background: #1E2529;
---
> 	background: #000;
126c123
< 	background: #1E2529;
---
> 	background: #005E82;
149c146
< 	background-color: #020509;
---
> 	background-color: #005E82;
153c150
< 	background: #020509 url(../images/navigation/bullet_selected.png) no-repeat 5px 50%;
---
> 	background: #005E82 url(../images/navigation/bullet_selected.png) no-repeat 5px 50%;
159c156
< 	background-color: #1E2529;
---
> 	background-color: #005E82;
201c198
< 	background: #020509;
---
> 	background: #005E82;
237c234
< 	border: 2px solid #828F95;
---
> 	border: 2px solid #005E82;
243,244c240,241
< 	background: #D3DADD;
< 	border-bottom: 1px solid #AEB8BC;
---
> 	background: #005E82;
> 	border-bottom: 1px solid #005E82;
275c272
< }
\ No newline at end of file
---
> }

***************
navigation.css
***************
48c48
< 	background: url(../images/dock/welcome_message.png) no-repeat 0 50%;
---
> 	background: #005E82;
122c122
< 	background: #020509 url(../images/dock/center_bg.png) repeat-x;
---
> 	background: #005E82;
129d128
< 	background: url(../images/dock/right_bg.png) no-repeat 100% 0;
138,139c137
< 	background: url(../images/dock/left_bg.png) no-repeat 0 0;
< 	border-right: 1px solid #34404F;
---
> 	border-right: 1px solid #005E82;
156c154
< 	border-color: #DEDEDE #BFBFBF #BFBFBF #DEDEDE;
---
> 	border-color: #005E82;
162c160
< 	border-top: 1px solid #DEDEDE;
---
> 	border-top: 1px solid #005E82;
206c204
< 	background-color: #D3DADD;
---
> 	background-color: #005E82;
238c236
< 	background-color: #DFF4FF;
---
> 	background-color: #005E82;
248c246
< 	background-color: #828F95;
---
> 	background-color: #005E82;
252c250
< 	background-color: #828F95;
---
> 	background-color: #005E82;
257c255
< 	background-color: #828F95;
---
> 	background-color: #005E82;
266c264
< 	border-bottom: 2px solid #DEDEDE;
---
> 	border-bottom: 2px solid #005E82;
 }}}
 1. Deploy the theme. As I am using an external server to deploy I need to use scp and while the ant script could be changed to do that in one step I put it here separately because still I use my local server instance as well and so I need the ant script to work locally.
```
ant clean deploy
scp ~/liferay-plugin-deploy/005E82-theme-5.2.3.1.war liferay@liferay-glassfish-server:~/liferay-plugin-deploy
```
 1. Now go to Liferay and from the Dock Menu select "Manage Pages", click on "Look An Feel" and select the new created theme. See how the creative has changed.
 1. Commit to the repository the whole plugin folder but exclude the below folders:
```
docroot/css		
docroot/images		
docroot/javascript	
docroot/templates
```
 1. Remember to change the theme per community using "Manage Pages" option. Note that for private and public pages you can specify different themes.
 1. Configure extension environment to use custom landing page adding the two lines below.
```
#vi ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/src/portal-ext.properties 
auth.forward.by.last.path=true
login.events.post=com.liferay.portal.events.LoginPostAction,com.ext.portal.events.CustomDefaultLandingPageAction
```
 1. Deploy the properties file
```
cd ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/
ant deploy-properties
```
 1. Create the needed CustomDefaultLandingPageAction class
```
mkdir -p ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/src/com/ext/portal/events
cp ~/liferay-components/liferay-portal-src-5.2.3/portal-impl/src/com/liferay/portal/events/DefaultLandingPageAction.java ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/src/com/ext/portal/events/CustomDefaultLandingPageAction.java
```
 1. Start by editing the java class changing its namespace to com.ext.portal.events.CustomDefaultLandingPageAction.java
```
#vi ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/src/com/ext/portal/events/CustomDefaultLandingPageAction.java 
```
 1. Include specific business rules to redirect to the proper page according to your needs. You might want to visit http://www.liferay.com/community/forums/-/message_boards/message/2111793 to get an idea of the needed code. In my case I modified the code there to redirect the user to his first available community.
 1. Deploy the newly created class. The whole ext-impl.jar will be actually deployed:
```
cd ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl
rm  classes/META-INF/*
ant deploy
```
 1. I have mavenized the ext-impl environment and you can find it at http://nestorurquiza.googlecode.com/svn/trunk/customizing-liferay/mavenized-ext-impl/. Going that path no more ant tasks will be needed to create the ext-impl.jar.
 1. Configure email. Edit and deploy portal-ext.properties 
```
#Email Config
mail.session.mail.store.protocol=smtp
mail.session.mail.transport.protocol=smtp
mail.session.mail.smtp.host=smtp.gmail.com
mail.session.mail.smtp.password=yourpassword
mail.session.mail.smtp.user=yourgmailaddress
mail.session.mail.smtp.port=465
mail.session.mail.smtp.auth=true
mail.session.mail.smtp.starttls.enable=true
mail.session.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory
```
 1. Send an email
```
InternetAddress from = new InternetAddress("nestor.urquiza@gmail.com", "myself");
InternetAddress to = new InternetAddress("nestor.urquiza@gmail.com", "myself");
MailMessage message = new MailMessage(from, to, "test from liferay", "This is a test message from a Liferay portlet", true);
log.info("Test Email Message: "+ message);
MailServiceUtil.sendEmail(message);
```
 1. Remove the Dock (Welcome tab) for non-admin users. In your theme edit docroot/_diffs/templates/portal_normal.vm as shown below. Do not forget to reinstall the theme ( "ant clean deploy" and make sure the theme is droped into the liferay plugin folder. 
```
#if($permissionChecker.isOmniadmin())
 #parse ("$full_templates_path/dock.vm")
#end
```
## Disabling cache in development environments

 1. Take a look at liferay WAR or exploded directory WEB-INF/classes/portal-developer.properties file. Do not copy that content in ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl/src/portal-ext.properties and if you do you must use a different file for production. Liferay has a better way to deal with it, in order to pull the properties from the developer file just use a special JVM system property. For example in tomcat:
```
JAVA_OPTS="-Dexternal-properties=portal-developer.properties $JAVA_OPTS"
```
 1. If you insist on using the portal-ext.properties (read previous point) then here is a copy of the developer properties:
```
#
# Eliminating Caching for development environment. Please put true in production systems
#
theme.css.fast.load=false
theme.images.fast.load=false
javascript.fast.load=false
javascript.log.enabled=true
layout.template.cache.enabled=false
browser.launcher.url=
last.modified.check=false
openoffice.cache.enabled=false
velocity.engine.resource.manager.cache.enabled=false
com.liferay.portal.servlet.filters.cache.CacheFilter=false
com.liferay.portal.servlet.filters.theme.ThemePreviewFilter=true
```
 1. Only if you are actually changing the properties (again I suggest you use the system properties instead) you will need to deploy properties
```
cd ~/liferay-components/liferay-portal-ext-5.2.3/ext-impl
ant deploy-properties
```
