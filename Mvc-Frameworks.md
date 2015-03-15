After several years of programming for the Web I am still looking for the ideal WEB framework ;-) I even built my own one (http://thinkinginsoftware.blogspot.com/). While evaluating Spring Framework, Rails and Grails some time ago I wrote these notes which at some point I would review just to see mainly if Grails is now in better shape.


# Grails

No much to be said. The WEB is plenty of information and Google is your friend but I feel like http://jonasfagundes.com/blog/2008/01/grails-the-good-the-ugly-and-the-bad can give you a good starting point at least to know the origins (I have found history is sometimes the best starting point when you want to learn something new)

To check in Grails projects into svn read this article http://docs.codehaus.org/display/GRAILS/Checking+Projects+into+SVN

## collab-todo

I have started with "Beginning Groovy and Grails: From Novice to Professional" which you can buy in http://www.amazon.com/gp/product/1430210451/ref=cm_rdp_product. While learning from this book I have been playing with vim, cream and groovy Eclipse plugin so far. Debugging has been a challenge BTW (http://jira.codehaus.org/browse/GROOVY-2208)

### Application code

For the Authentication chapter (7) I found explanations incomplete basically but thanks to that I was forced to play and learn about Spring Security (aka Acegi). I recommend to  follow directions from http://docs.codehaus.org/display/GRAILS/AcegiSecurity+Plugin+-+Basic+Tutorial configuring request maps from SecurityConfig.groovy
```
requestMapString = """
		CONVERT_URL_TO_LOWERCASE_BEFORE_COMPARISON
		PATTERN_TYPE_APACHE_ANT


		
		/todo/**=IS_AUTHENTICATED_FULLY
		/category/**=IS_AUTHENTICATED_FULLY
		/register/**=IS_AUTHENTICATED_FULLY
		/user/**=ROLE_ADMIN
        /role/**=ROLE_ADMIN
        /login/**=IS_AUTHENTICATED_ANONYMOUSLY
		/logout/**=IS_AUTHENTICATED_ANONYMOUSLY
		/**=IS_AUTHENTICATED_ANONYMOUSLY
		
	"""
```
I added firstName and lastName intead of userRealName and for that I had to update all gsp files below User View.

I also use <g:loggedInUserInfo> to ouput first and last name in the topbar.

The book assumes in chapter 7 and 8 that you work by yourself analyzing existing code that you download. I understand this is done to present a full featured application for which details could take way more than a book, but at the same time I would rather pick as a beginner a book that push me to do everything without downloading code. I find that approach better for learning. The downloadable code is good to have it as well though.

Chapters 12 and 13 I enjoyed reading but felt like I better go back to them when needed rather than considering them important to be included in the code I use to learn Grails basics.

There are some erratas I have reported which I am including here anyway:

  1. Mixing pure HTML and GSP in a form like below:
  {{{
  <form action="handleLogin">
  <g:actionSubmit	value="Login" />
  }}}
  results after submission in a URL like:
  {{{
  http://localhost:8080/collab-todo/user/handleLogin?userName=fred&_action_Login=Login
  }}}
  because the second line is translated to:
  {{{
  <input type="submit" name="_action_Login" value="Login" />
  }}}
  UserController will have to decide from two actions. Use either:
  {{{
  <form action="handleLogin">
  <input type="submit" value="Login"/>
  }}}
  or:
  {{{
  <g:form>
  <g:actionSubmit	action="handleLogin" value="Login" />
  }}}
  2. When running tests you get:
  {{{
  testHandleLoginInvalidUser	Error	Cannot send redirect - response is already committed
  }}}
  UserController.groovy is missing an else as below:
  {{{
  if (!user) {
    flash.message = "User not found for userName: ${params.userName}"
    redirect(action:'login')
  }else{
    session.user = user
    redirect(controller:'todo')
  }	
  }}}
  3. The below code:
  {{{
  if (session.user.id != params.id) {
  }}}
  should be:
  {{{
  if (session.user.id != params.id.toInteger()) {
  }}}
  4. The recommended entry in UrlMappings.groovy for REST is:
  {{{
  "/$rest/$controller/$id?"{
  }}}
  But it will cause a NullPointerException from RestController because domainClassName will be "Rest" **always**
  {{{
  domainClassName = capitalize(params.controller)
  }}}
  Just change "controller" by "realcontroller" for example.
  
I did not play with Jasper Reports from Chapter 10 neither Quartz Batch processing from Chapter 11. So far there is no code reflecting that chapter in this repository.

Finally the book does not mention Maven and instead guides the developer through the use of GANT. Using ANT or for groovy GANT has several disadvantages in comparison to Maven http://graemerocher.blogspot.com/2008/01/why-grails-doesnt-use-maven.html but the one that worries me the most is the dependency management. By the time of the book writting and even as of today (09/10/2008) Maven faces some challenges to support Grails project. The most important part for me is that developers shouldn't be forced to commit jar files (dependencies) into SVN and Maven provides a clean solution for it. Some efforts are on their way ( http://forge.octo.com/maven/sites/mtg/grails-maven-plugin/index.html ) but still requested features in JIRA are showing Maven cannot be used to checkout and build a whole Grails project just based on sources and a POM which specifies dependencies.

Disclaimer: I am not hosting the source code that comes with the book but instead I am just reading the book and while making changes to the code I keep updating it on SVN so I can check out in different machines. If this action is illegal by any means please send me an email and I will remove the project.

### mysql

I have worked for development using MySQL instead of HSQLDB:
 1. Download the driver from http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.6.tar.gz/from/pick#mirrors and put the jar file (mysql-connector-java-5.1.6-bin.jar) it in the Grails lib folder.
 1. Edit DataSource.groovy:
 {{{
 development {
	dataSource {			
		driverClassName = "com.mysql.jdbc.Driver"
	        username = "root"
	        password = ""
	        dbCreate = "update"
	        url = "jdbc:mysql://localhost:3306/collab_todo_dev"
	}
 }
 }}}
 Note we use "create-drop" and so if you started coding with HSQLDB you will end up with all your domain changes in place in MySQL after a "grails run-app"
 1. Create the dev db
 {{{
 create database collab_todo_dev 
 }}}
 
# SpringMVC

 While Grails does a wonderful job trying to vreate a DSL for WEB development on top of Spring Framework by the time of this writting it is tough to get all the pieces (edition, syntax highlighting, autocompletion, debugging) in just one IDE. On top of that for existing projects where someone wants to create a migration path to existing applications and the whole rearchitecture of them is prohibited Spring comes to your rescue.

 I have followed some tutorials and I am including here the code for the only purpose of keeping track of what I have done and be able to recreate the projects in any of my three development environments (Mac, Windows and Ubuntu)

## fasttutorial

This is a quick, straight and very useful beginner tutorial that can be found at http://maestric.com/en/doc/java/spring
 
## mhimustutorial

Again, another useful tutorial for beginners that can be found at http://mhimu.wordpress.com/2007/11/27/spring-mvc-tutorial/

## coctutorial

I have decided to document the Convention Over Configuartion (CoC) solution SpringMVC currently provides. The post can be found at () and as pointed out there the source code is available from http://nestorurquiza.googlecode.com/svn/trunk/coctutorial/