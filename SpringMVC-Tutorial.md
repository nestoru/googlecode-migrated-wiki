# CoC or Convention over Configuration in Spring MVC Framework

There are currently four URL-to-controller mapping mechanisms supported in SpringMVC:
```
org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping
org.springframework.web.servlet.handler.SimpleUrlHandlerMapping
org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping
Annotation-based mapping
```

The first one is the default mapping and it is explained in other beginner tutorials like http://maestric.com/en/doc/java/spring. In the official Spring distribution there is another useful tutorial using this mapping approach (file://${spring-root}/docs/MVC-step-by-step)

The second one allows for mapping of specific URls to controllers independent of the existence or not of specific beans. It has been covered in tutorials like http://mhimu.wordpress.com/2007/11/27/spring-mvc-tutorial/. In the official Spring distribution there is a sample application that uses this mapping technique (file://${spring-root}/samples/jpetstore)

The third one (only available in Spring 2.0 and up) while explained in several places is I think lacking still of a do-it-yourself example. 

The last is available starting from Spring 2.5 and is about Configuration by annotations. In the official Spring distribution there is a sample application that uses this mapping technique (file://${spring-root}/samples/petclinic)

This is my attempt to use third one as I am looking for Convention over Configuration (CoC). However as I have posted ( http://forum.springframework.org/showthread.php?t=60718 ) there is IMO a pitfall with the current implementation of ControllerClassNameHandlerMapping. Basically even though the routing can be configured to be done following conventions, controllers must be still annotated.

If you already went through the previously mentioned tutorials you are ready to understand the code you can find at http://nestorurquiza.googlecode.com/svn/trunk/coctutorial/. 

Assumming you have been using some other framework or basic Servlet + JSP you will love to be able to introduce Spring MVC without affecting your current application. In order to do so we use the below in web.xml:
```
  <servlet>  
    <description>Spring MVC Dispatcher Servlet</description>  
    <servlet-name>coctutorial</servlet-name>  
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
    <load-on-startup>1</load-on-startup>  
  </servlet>  
  <servlet-mapping>  
    <servlet-name>coctutorial</servlet-name>  
    <url-pattern>/spring/**</url-pattern>  
  </servlet-mapping>
```

The important part is 'url-pattern' which specifies urls starting with "/spring" will be managed by Spring.

Below is the annotation needed in com.nestorurquiza.spring.mvc.web.GreetingController controller. That will make the class automatically recognized as a valid Controller:
```
@Controller
public class GreetingController extends MultiActionController{
```

Take a look at coctutorial-servlet.xml. As you see below the above class will be found if it is in the base-package defined below.
```
    <!-- 
      Define Annotation-based mapping. 
      This is still needed when using ControllerClassNameHandlerMapping 
      if you want to omit declaring a bean per controller. Remember to annotate your controller
      using @Controller
    -->    
    <context:component-scan base-package="com.nestorurquiza.spring.mvc.controller"/>
    
    <!-- Define Convention Over Configuration Mapping -->
    <bean id="urlMapping"  class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping">
      <property name="caseSensitive" value="true"/>  
      <property name="order" value="0" />
      <property name="pathPrefix" value="/"/>
      <property name="basePackage" value="com.nestorurquiza.spring.mvc.web"/>
    </bean>
```

After running Ant in the root directory a war file will result which you can drop in a Servlet Container like Tomcat. Once you do so you should be able to request the below URLs:
```
http://localhost:8080/coctutorial/
http://localhost:8080/coctutorial/spring/greeting/hello
http://localhost:8080/coctutorial/spring/greeting/hi
```

The first one is a simple JSP file that is rendered without using SpringMVC. The last two URLs show Spring MVC "Convention Over Configuration" using a combination of Annotations (there is no way to get rid of it as for the time of this writing) and ControllerClassNameHandlerMapping.

Note that in my case I deployed a WAR file and the requests are contextualized to 'coctutorial'.

# Internationalization (i18n)

Basic and advanced Internationalization support can be accomplished several ways. Below is an example of basic one based on Browser sent HTTP Header "Accept-Language". Read the documentation to add for example Cookie support so a user can change her locale and that way ignore the supplied by the Browser locale.

First add the below bean definition in the application context xml file:
```
<!-- Resolves i18n messages from files of the form messages_<lowerCaseLanguageAbbreviation>_<upperCaseCountryAbbreviation -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
      <property name="basename">
        <value>messages</value>
      </property>
    </bean>
```
Then proceed including all files you will need per locale (language+country). Be sure they end up in the application root. The first file below has the lowest priority and will be parsed if no specific locale is found.
```
messages.properties        
messages_en_US.properties
messages_en_GB.properties  
messages_es_ES.properties
....
```
Edit the above files like in:
```
greeting.hello=Hello Pal
greeting.hi=Hi Pal
```
And use the internationalized messages in your Controller code like:
```
  private String findLocaleGreeting(HttpServletRequest request, String greetingI18n){
    ApplicationContext applicationContext = this.getApplicationContext();  
    RequestContext requestContext = new RequestContext(request);
    Locale myLocale = requestContext.getLocale();
    if(myLocale == null)
      myLocale = Locale.US;
    String greeting = applicationContext.getMessage(greetingI18n, null, myLocale); 
    return greeting;
  }
```
I have included in this tutorial a controller called LocaleGreetingController. You can test it using "curl" simulating that way different browser "Accept-Language" headers:
```
curl --header "Accept-Language: en-gb" "http://localhost:8080/coctutorial/spring/localeGreeting/hello"
Hello Mate !!!
curl --header "Accept-Language: en-us" "http://localhost:8080/coctutorial/spring/localeGreeting/hello"
Hello Pal !!!
curl --header "Accept-Language: es-es" "http://localhost:8080/coctutorial/spring/localeGreeting/hello"
Hola !!!
curl --header "Accept-Language: en-gb" "http://localhost:8080/coctutorial/spring/localeGreeting/hi"
Hi Mate !!!
```
# Serving static pages

Static pages should be served out of the Spring concern. In fact it is a good practice to have a Web Server like Apache serving static content instead of your application server or servlet container.
I have included an example of a page that is served directly without Spring intervention:
```
http://localhost:8080/coctutorial/static.html
```
# Serving plain old JSP/Servlets

As long as the mapping does not apply your Servlet/JSP will work as expected. Spring is not invasive. i have included a JSP page to show this:
```
http://localhost:8080/coctutorial/index.jsp
```
# XSS vulnerability protection

If you do not escape the HTTP request parameters you are taking a risk of XSS attack. You can find several XSS vulnerabilities examples at http://ha.ckers.org/xss.html.

How can we use a general method to protect a Spring application against XSS attacks?
Either using JSTL's <c:out> or plain scriptlets you will need to deal with this problem. Unfortunately XSS protection is not provided out of the box in Spring MVC. One could think about using Aspects for this but I do think this task is better for a filter. I came across the Stripes project http://mc4j.org/confluence/pages/viewpage.action?pageId=2593 and the HTMLInputFilter http://josephoconnell.com/java/xss-html-filter/ which is actually  a port from PHP (http://code.iamcal.com/php/lib_filter/). Combining them I have added XSS protection.

Try commenting out the below snippet from web.xml:
```
<!-- Avoiding XSS -->
  <filter>
    <filter-name>XssFilter</filter-name>
    <filter-class>com.nestorurquiza.filters.XssFilter</filter-class>
  </filter>   
  <filter-mapping>
    <filter-name>XssFilter</filter-name>
    <url-pattern>/**</url-pattern>		
  </filter-mapping>
```
And then you will see how the below URLs show XSS vulnerability.
```
http://localhost:8080/coctutorial/spring/params/write?par=XssAttack%3CSCRIPT%3Ealert(String.fromCharCode(88,83,83))%3C/SCRIPT%3E
http://localhost:8080/coctutorial/spring/params/write?<script >alert(document.cookie)</script>
```
Uncomment it back and the whole site is now protected agins XSS attacks.

# JSON Services

Say you want to use your Spring Controllers to expose JSON then you need to use a JSONView. The same will apply for other content types like XML. I have included a sample following guidelines from http://ffzhuang.blogspot.com/2008/03/spring-mvc-json-view.html
So hit:
```
http://localhost:8080/coctutorial/spring/jsonParams/write?p1=foo&p2=bar&callback
```
You will get a JSON response 
```
({"callback":[""],"p1":["foo"],"p2":["bar"]});
```
# Deploying

The resulting WAR file (after running "ant clean;ant") has been tested in tomcat 6/5 and in JBoss 4.0.4. You will notice some specific configurations on build.properties and build.xml.

# IDE

Files are provided for Eclipse using external dependencies to JBoss4.0.4 lib directory on a local Ubuntu Desktop. You might need to configure the project properties especially "Java Build Path"

# Maven integration

See my Maven Tutorial hosted here at Goggle Code for information to migrate this tutorial to Maven.