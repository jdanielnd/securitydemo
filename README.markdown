### How to create a GWT application with Spring Security from the scratch

We will start by creating a sample application from GWT. If want to add Spring Security to your own web application, you can try starting on 2nd step. However, since I don't know what's going on your web application, it may not work. I recommend you try on a new GWT sample application to understand the steps, and then follow to your own application.

## 1. Creating a sample application

So, assuming that you have GWT SKD and Eclipse installed, let's create a new Web Application Project on Eclipe. Just go to File > New > Web Application Project, give it the name `securitydemo`, and click Finish. Try to run it to see if everything it's alright.

Let's start the proccess of adding Spring Security. It can be divided in four steps:

  - Add Spring libraries to your buildpath
  - Implement a Spring Security XML configuration file
  - Add the Spring DelegatingFilterProxy to your web.xml file
  - Add the Spring Security XML configuration file reference to web.xml
  
So, let's do it:

***

## 2. Add Spring libraries to your buildpath

You can download the latest release of Spring Security on this link: [http://static.springsource.org/spring-security/site/downloads.html](http://static.springsource.org/spring-security/site/downloads.html)
You also need to download Spring Framework, that can be found on this link: [http://www.springsource.org/download/community?project=Spring%2520Framework](http://www.springsource.org/download/community?project=Spring%2520Framework)

We will need the following jar files:

  - spring-security-config-X.X.X.RELEASE.jar - deals with XML configuration file
  - spring-security-core-X.X.X.RELEASE.jar - deals with all the authentication and access-control classes and interfaces
  - spring-security-web-X.X.X.RELEASE.jar - deals with web authentication and URL-based access control

You also need those jar files from Spring Framework:

  - org.springframework.aop-X.X.X.RELEASE.jar
  - org.springframework.asm-X.X.X.RELEASE.jar
  - org.springframework.beans-X.X.X.RELEASE.jar
  - org.springframework.context-X.X.X.RELEASE.jar
  - org.springframework.core-X.X.X.RELEASE.jar
  - org.springframework.expression-X.X.X.RELEASE.jar
  - org.springframework.web-X.X.X.RELEASE.jar
  - org.springframework.web.servlet-X.X.X.RELEASE.jar

You can find them on the `dist` folder of the files you have downloaded. Copy them to your app `lib` folder: `war/WEB-INF/lib` and add them to your buildpath. To do that you need to right-click your applications folder on Eclipse, select Build Path > Configure Built Path. Click on button Add JARs, and select the files you just copied to there. (If you can't find the files there, press F5 on Eclipse Package Explorer in order to refresh it)

***

## 3. Implement a Spring Security XML configuration file

Create a XML file called `securitydemo-security.xml` in your WEB-INF directory. It should follow the pattern `[your application's name]-security.xml`. Add the following code to it:

    <?xml version="1.0" encoding="UTF-8"?>
    <beans:beans xmlns="http://www.springframework.org/schema/security"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:beans="http://www.springframework.org/schema/beans"
        xsi:schemaLocation="
                http://www.springframework.org/schema/beans 
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/security 
                http://www.springframework.org/schema/security/spring-security-3.1.xsd
        ">

        <http auto-config="true">
          <intercept-url pattern="/*l" access="ROLE_USER"/>
        </http>
        
        <authentication-manager alias="authenticationManager">
          <authentication-provider>
            <user-service>
              <user authorities="ROLE_USER" name="guest" password="guest"/>
            </user-service>
          </authentication-provider>
        </authentication-manager>
    </beans:beans>

The `<http auto-config="true">` tag will take care of configurating everything for us. The URL pattern to be intercepted, i.e., to be secured is defined by the `<intercept-url>` tag.

Then we're setting our Authentication Manager. In this example we'll be using a single user defined on the XML file itself. So we define the Authentication Provider, and populate it with a User Service, that contains a user with "ROLE_USER" authority and "guest" as its name and password.

***

## 4. Add the Spring DelegatingFilterProxy to your web.xml file

Add the following snippet to your `web.xml` file

	<!-- Spring Security Filter Chain -->
	<filter>
		<filter-name>springSecurityFilterChain</filter-name>
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>springSecurityFilterChain</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
By doing this we're applying a ServletRequest filter and configuring it to handle requests matching the given URL pattern - "/*" - for this wildcard, the filter will be applied to all requests. Notice that this is not connected to the configuration we did on XML file yet.

***

## 5. Add Spring ApplicationContext XML file and listener

Create a XML file called securitydemo-base.xml. This file will define our Application Context to Spring. Add the following code to it:

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:context="http://www.springframework.org/schema/context"

	xsi:schemaLocation="
			 http://www.springframework.org/schema/beans
			 http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
			 http://www.springframework.org/schema/jdbc
			 http://www.springframework.org/schema/jdbc/spring-jdbc-3.1.xsd
			 http://www.springframework.org/schema/context
			 http://www.springframework.org/schema/context/spring-context-3.1.xsd">

	<context:annotation-config />

    </beans>

Now we need to reference it in our `web.xml` file. Add the following snippet of code that will tell Spring where our application context is defined.

	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			/WEB-INF/securitydemo-base.xml
		</param-value>
	</context-param>

	<listener>
		<listener-class>
			org.springframework.web.context.ContextLoaderListener
	 	</listener-class>
	</listener>

***

## 6. Add the Spring Security XML configuration file reference to `web.xml`

Now we need to tell Spring to also read our securitydemo-security.xml configuration file. To do this, we need to add its path to the contextConfigLocation we have created on the previous step:

	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			/WEB-INF/securitydemo-security.xml
			/WEB-INF/securitydemo-base.xml
	  </param-value>
	</context-param>
	
That's it! We just need to run our server and try to access it. We will be redirected to a login page!
