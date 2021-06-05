# Jersey

#### Background

JAX-RS is a Java specification for creating REST APIs using annotations on class methods and properties.

Jersey is the reference implementation of JAX-RS.  It lives in a third party dependency called org.glassfish.jersey (jersey-container-servlet-core)

JAX-B is a Java implementation for mapping objects to XML and vice-versa.  JaxB is included with Java and lives in the namespace javax.xml.bind.




#### Creating a REST API using Jersey, Maven, Netbeans and Docker

To run Netbeans in Docker:

	docker run -d --net="host" -it -e DISPLAY=localhost:1.0 -e "TZ=America/Chicago" --name netbeans psharkey/netbeans-8.1

We can create a working Jersey sample API project using the maven archetype called jersey-quickstart-webapp.  I think all of the Java IDEs have the capability to scaffold new projects using Maven archetypes.

* Choose File > New Project
* Choose Maven > Project from Archetype
* Input archetype name: jersey-quickstart-webapp
* Choose version 2.26.x (or current latest)
* Set the project name to something short.  This will become the project folder name and the WAR file name.
* Set the group ID to the desired namespace, e.g. com.marcs.servlet
* Click finish

Configure a server:

* Install tomcat8 and tomcat8-admin using your package manager
* Edit /etc/tomcat8/tomcat-users.xml and add a user to administer the server

	<tomcat-users>
	<role rolename="manager-gui" />
	<role rolename="manager-script" />
	<user username="admin" password="password" roles="manager-gui, manager-script" />
	</tomcat-users>

* Netbeans and Tomcat don't seem to quite agree on the file structure.  Create a symlink called /usr/share/tomcat8/conf pointing to /etc/tomcat8, to make Netbeans happy.

	ln -s /etc/tomcat8 /usr/share/tomcat8/conf

Add Tomcat to Netbeans:

* Click Tools->Plugins->Tomcat (Install)
* Click Tools->Servers->Add Server
* Choose Tomcat Server
* Set the Catalina Home: /usr/share/tomcat8
* Set the Catalina Base: /usr/share/tomcat8
* Set the username and password that you put into tomcat-users.xml

The project you created has class called MyResource which returns a string on /myresource.  Let's see if it works.

* Click Debug Project (Ctrl+F5)
* When prompted, select the Tomcat Server to run the project on and click Remember Permanently
* It will deploy.  Now visit: http://localhost:8080/manager and we should see a new webapp deployed into Tomcat
* Open the web app URL, which should be http://localhost:8080/(projectname)
* We should see the webapp running successfully.
* We can call the MyResource.java::getIt method using the URI http://localhost:8080/(your project name)/webapi/myresource

#### Changing the URI of the method

The web.xml class is specifying the root Uri that the resource classes appear under:

    <servlet-mapping>
        <servlet-name>Jersey Web Application</servlet-name>
        <url-pattern>/webapi/*</url-pattern>
    </servlet-mapping>

If we wish, we can change it to /* to remove the /webapi part of the URI.

#### Returning plain old Java objects as XML and JSON

Create a new class and decorate it with the attributes required for object mapping:

	package com.marcs.servlet.marcs5;

	import javax.xml.bind.annotation.XmlAttribute;
	import javax.xml.bind.annotation.XmlElement;
	import javax.xml.bind.annotation.XmlRootElement;

	@XmlRootElement
	public class MarcsObject {
	    
	    @XmlElement
	    public String marcsString;
	    
	    @XmlElement
	    public int marcsInt;
	}

Change the MyResource::getIt function to return the object.

    @GET
    @Produces(MediaType.TEXT_XML)
    public MarcsObject getIt() {
        
        MarcsObject marcsObject = new MarcsObject();
        marcsObject.marcsString = "What is the answer?";
        marcsObject.marcsInt = 42;
        
        return marcsObject;
    }

Now it should respond with XML:

	<marcsObject>
		<marcsString>What is the answer?</marcsString>
		<marcsInt>42</marcsInt>
	</marcsObject>

Now let's try returning JSON.  Change the GetIt function to specify that JSON is returned instead of XML.

    @Produces(MediaType.APPLICATION_JSON)

This doesn't work unless we add an additional dependency to pom.xml.

    <dependency>
        <groupId>org.glassfish.jersey.media</groupId>
        <artifactId>jersey-media-moxy</artifactId>
    </dependency>

Bingo, the result is now:

	{"marcsString":"What is the answer?","marcsInt":42}


#### Creating a REST API using Jersey and Maven on the command line

Maven is a tool for Java which does scaffolding of new projects, package management, and build.

Create the project from the archetype:

	mvn archetype:generate -DgroupId=com.marcs.servlet.four -DartifactId=marcs4 -DarchetypeGroupId=org.glassfish.jersey.archetypes -DarchetypeArtifactId=jersey-quickstart-webapp -DarchetypeVersion=2.26-b03 -DinteractiveMode=false 

Change into the new directory created for the project.  We should see pom.xml and the skeleton project source code.  The pom.xml tells maven how to build the project.  

To build the project:

	mvn install

We need to add a tomcat plugin for maven which knows how to deploy the WAR into tomcat.  

The file pom.xml inside the project is used to configure Maven for your project.  The code to add to pom.xml is:

	<plugin>
		<groupId>org.apache.tomcat.maven</groupId>
		<artifactId>tomcat7-maven-plugin</artifactId>
		<version>2.2</version>
		<configuration>
			<url>http://localhost:8080/manager/text</url>
			<server>TomcatServer</server>
			<path>/mkyongWebApp</path>
		</configuration>
	</plugin>

Then we need to set our tomcat password in %MAVEN_PATH%/conf/settings.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<settings ...>
		<servers>
			<server>
				<id>TomcatServer</id>
				<username>admin</username>
				<password>password</password>
			</server>
		</servers>
	</settings>

Finally, deploy the project:

	mvn tomcat7:deploy

At this point we should have a new web app running on ths server again, at http://localhost:8080/marcs4.

Maven can also undeploy or redeploy it.

	mvn tomcat7:undeploy
	mvn tomcat7:redeploy



#### References

* Getting Started with Jersey: https://jersey.java.net/documentation/latest/getting-started.html
* XML Example with Jersey + JAXB: https://examples.javacodegeeks.com/enterprise-java/rest/jersey/xml-example-with-jersey-jaxb/
* How to deploy Maven based war file to Tomcat: https://www.mkyong.com/maven/how-to-deploy-maven-based-war-file-to-tomcat/
* List of Maven archetypes: https://maven-repository.com/


