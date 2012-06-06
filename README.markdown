# How to create a GWT application with Spring Security from the scratch

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

# How to read users from a database

In the last example we could configure Spring Security to protect our application, but username were hardcoded on the XML file. What if we want to read users from a database? Let's see how to do it!

## 1. Creating the database

First, let's create a database to store our `users` table and `user_roles` table.

    $ mysqladmin -u root -p create securitydemo
    
Then let's create a table to store the users

    CREATE TABLE `users` (
      `USER_ID` INT(10) UNSIGNED NOT NULL,
      `USERNAME` VARCHAR(64) NOT NULL,
      `PASSWORD` VARCHAR(64) NOT NULL,
      `ENABLED` tinyint(1) NOT NULL,
      PRIMARY KEY (`USER_ID`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

And now let's create a table to store user roles

    CREATE TABLE `user_roles` (
      `USER_ROLE_ID` INT(10) UNSIGNED NOT NULL,
      `USER_ID` INT(10) UNSIGNED NOT NULL,
      `AUTHORITY` VARCHAR(64) NOT NULL,
      PRIMARY KEY (`USER_ROLE_ID`),
      KEY `FK_user_roles` (`USER_ID`),
      CONSTRAINT `FK_user_roles` FOREIGN KEY (`USER_ID`) REFERENCES `users` (`USER_ID`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
That's it, our tables are ready, so let's populate them

    INSERT INTO securitydemo.users (USER_ID, USERNAME,PASSWORD, ENABLED)
    VALUES (1, 'user', '123456', TRUE);
     
    INSERT INTO securitydemo.user_roles (USER_ROLE_ID, USER_ID, AUTHORITY)
    VALUES (1, 1, 'ROLE_USER');
    
## 2. Add database support to Spring

Now we need to tell Spring how to connect to this database. In order to do that we need to create a DataSource JDBC bean on our ApplicationContext. And so Spring can read it, we need the following libraries:

  - org.springframework.jdbc-X.X.X.RELEASE.jar
  - org.springframework.transaction-X.X.X.RELEASE.jar
  - mysql-connector-java-X.X.X-bin.jar
  
The last library can be downloaded on http://dev.mysql.com/downloads/connector/j/

Having added those libraries we can describe the DataSource bean on our `securitydemo-base.xml` file. Add the following snippet of code before `</beans>` tag:

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name="driverClassName" value="com.mysql.jdbc.Driver" />
      <property name="url" value="jdbc:mysql://localhost:3306/securitydemo" />
      <property name="username" value="root" />
      <property name="password" value="password" />
    </bean>
    
We have set a bean with the id `dataSource` and configured it to read from our database.
    
## 3. Add database support to Spring Security

Alright, now Spring can read this database, so we need to make Spring Security look up for users from this database. To do that we need to add a JdbcUserService into AuthenticationProvider. This is done by replacing `<user-service>` tag by the following snippet of code:

    <authentication-manager alias="authenticationManager">
      <authentication-provider>
        <jdbc-user-service data-source-ref="dataSource"
          users-by-username-query="
            select username,password, enabled 
            from users where username=?" 
	             
          authorities-by-username-query="
            select u.username, ur.authority from users u, user_roles ur 
            where u.user_id = ur.user_id and u.username =?  "
        />
      </authentication-provider>
    </authentication-manager>
    
What we have done is defined that Spring Security should look for users from the `dataSource` bean, that we have defined on the last step. We also defined the queries to look for users and authorities - both by username.

That's it, we can now try to log in using the username and password we created on step 1. However, the password is being stored as plain text on the database. It's a good practice to encrypt it before storing. Spring Security will have to encrypt the password you time when you try to login, and compare it with the encrypted password to check if its correct. Spring Security automatically deals with it if we set a password encoder on our AuthenticationProvider. To do that, just add the following code before the tag `<jdbc-user-service>`.

    <password-encoder hash="sha-256" />

And create a new user on the database with an encrypted password. To generate an encrypted password type the following on the bash:

    $ echo -n password | sha256sum
    5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8  -
    
where `password` is the password you want.

    INSERT INTO securitydemo.users (USER_ID, USERNAME,PASSWORD, ENABLED)
    VALUES (2, 'user2', '5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8', TRUE);
     
    INSERT INTO securitydemo.user_roles (USER_ROLE_ID, USER_ID, AUTHORITY)
    VALUES (2, 2, 'ROLE_USER');

And we're done! Try to login with the username `user2` and password `password`. Spring Security will automatically encrypt the typed password and compare to the database entry.
