## 63. Deploying to the Cloud

Spring Boot’s executable jars are ready-made for most popular cloud PaaS (Platform-as-a-Service) providers. These providers tend to require that you “bring your own container”. They manage application processes (not Java applications specifically), so they need an intermediary layer that adapts your application to the cloud’s notion of a running process.

Two popular cloud providers, Heroku and Cloud Foundry, employ a “buildpack” approach. The buildpack wraps your deployed code in whatever is needed to start your application. It might be a JDK and a call to  `java` , an embedded web server, or a full-fledged application server. A buildpack is pluggable, but ideally you should be able to get by with as few customizations to it as possible. This reduces the footprint of functionality that is not under your control. It minimizes divergence between development and production environments.

Ideally, your application, like a Spring Boot executable jar, has everything that it needs to run packaged within it.

In this section, we look at what it takes to get the [simple application that we developed](getting-started-first-application.html) in the “Getting Started” section up and running in the Cloud.

## 63.1 Cloud Foundry

Cloud Foundry provides default buildpacks that come into play if no other buildpack is specified. The Cloud Foundry [Java buildpack](https://github.com/cloudfoundry/java-buildpack) has excellent support for Spring applications, including Spring Boot. You can deploy stand-alone executable jar applications as well as traditional  `.war`  packaged applications.

Once you have built your application (by using, for example,  `mvn clean package` ) and have [installed the cf command line tool](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html), deploy your application by using the  `cf push`  command, substituting the path to your compiled  `.jar` . Be sure to have [logged in with your cf command line client](https://docs.cloudfoundry.org/cf-cli/getting-started.html#login) before pushing an application. The following line shows using the  `cf push`  command to deploy an application:

```java
$ cf push acloudyspringtime -p target/demo-0.0.1-SNAPSHOT.jar
```

> In the preceding example, we substitute  `acloudyspringtime`  for whatever value you give  `cf`  as the name of your application.

See the [cf push documentation](https://docs.cloudfoundry.org/cf-cli/getting-started.html#push) for more options. If there is a Cloud Foundry [manifest.yml](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html) file present in the same directory, it is considered.

At this point,  `cf`  starts uploading your application, producing output similar to the following example:

```java
Uploading acloudyspringtime... OK
Preparing to start acloudyspringtime... OK
-----> Downloaded app package (8.9M)
-----> Java Buildpack Version: v3.12 (offline) | https://github.com/cloudfoundry/java-buildpack.git#6f25b7e
-----> Downloading Open Jdk JRE 1.8.0_121 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_121.tar.gz (found in cache)
Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.6s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
Memory Settings: -Xss349K -Xmx681574K -XX:MaxMetaspaceSize=104857K -Xms681574K -XX:MetaspaceSize=104857K
-----> Downloading Container Certificate Trust Store 1.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-certificate-trust-store/container-certificate-trust-store-1.0.0_RELEASE.jar (found in cache)
Adding certificates to .java-buildpack/container_certificate_trust_store/truststore.jks (0.6s)
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Checking status of app 'acloudyspringtime'...
0 of 1 instances running (1 starting)
...
0 of 1 instances running (1 starting)
...
0 of 1 instances running (1 starting)
...
1 of 1 instances running (1 running)

App started
```

Congratulations! The application is now live!

Once your application is live, you can verify the status of the deployed application by using the  `cf apps`  command, as shown in the following example:

```java
$ cf apps
Getting applications in ...
OK

name                 requested state   instances   memory   disk   urls
...
acloudyspringtime    started           1/1         512M     1G     acloudyspringtime.cfapps.io
...
```

Once Cloud Foundry acknowledges that your application has been deployed, you should be able to find the application at the URI given. In the preceding example, you could find it at  `http://acloudyspringtime.cfapps.io/` .

### 63.1.1 Binding to Services

By default, metadata about the running application as well as service connection information is exposed to the application as environment variables (for example:  `$VCAP_SERVICES` ). This architecture decision is due to Cloud Foundry’s polyglot (any language and platform can be supported as a buildpack) nature. Process-scoped environment variables are language agnostic.

Environment variables do not always make for the easiest API, so Spring Boot automatically extracts them and flattens the data into properties that can be accessed through Spring’s  `Environment`  abstraction, as shown in the following example:

```java
@Component
class MyBean implements EnvironmentAware {

	private String instanceId;

	@Override
	public void setEnvironment(Environment environment) {
		this.instanceId = environment.getProperty("vcap.application.instance_id");
	}

	// ...

}
```

All Cloud Foundry properties are prefixed with  `vcap` . You can use  `vcap`  properties to access application information (such as the public URL of the application) and service information (such as database credentials). See the [‘CloudFoundryVcapEnvironmentPostProcessor’](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/cloud/CloudFoundryVcapEnvironmentPostProcessor.html) Javadoc for complete details.

> The [Spring Cloud Connectors](https://cloud.spring.io/spring-cloud-connectors/) project is a better fit for tasks such as configuring a DataSource. Spring Boot includes auto-configuration support and a  `spring-boot-starter-cloud-connectors`  starter.

## 63.2 Heroku

Heroku is another popular PaaS platform. To customize Heroku builds, you provide a  `Procfile` , which provides the incantation required to deploy an application. Heroku assigns a  `port`  for the Java application to use and then ensures that routing to the external URI works.

You must configure your application to listen on the correct port. The following example shows the  `Procfile`  for our starter REST application:

```java
web: java -Dserver.port=$PORT -jar target/demo-0.0.1-SNAPSHOT.jar
```

Spring Boot makes  `-D`  arguments available as properties accessible from a Spring  `Environment`  instance. The  `server.port`  configuration property is fed to the embedded Tomcat, Jetty, or Undertow instance, which then uses the port when it starts up. The  `$PORT`  environment variable is assigned to us by the Heroku PaaS.

This should be everything you need. The most common deployment workflow for Heroku deployments is to  `git push`  the code to production, as shown in the following example:

```java
$ git push heroku master

Initializing repository, done.
Counting objects: 95, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (78/78), done.
Writing objects: 100% (95/95), 8.66 MiB | 606.00 KiB/s, done.
Total 95 (delta 31), reused 0 (delta 0)

-----> Java app detected
-----> Installing OpenJDK 1.8... done
-----> Installing Maven 3.3.1... done
-----> Installing settings.xml... done
-----> Executing: mvn -B -DskipTests=true clean install

[INFO] Scanning for projects...
Downloading: https://repo.spring.io/...
Downloaded: https://repo.spring.io/... (818 B at 1.8 KB/sec)
		....
Downloaded: http://s3pository.heroku.com/jvm/... (152 KB at 595.3 KB/sec)
[INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/target/...
[INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/pom.xml ...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 59.358s
[INFO] Finished at: Fri Mar 07 07:28:25 UTC 2014
[INFO] Final Memory: 20M/493M
[INFO] ------------------------------------------------------------------------

-----> Discovering process types
Procfile declares types -> web

-----> Compressing... done, 70.4MB
-----> Launching... done, v6
http://agile-sierra-1405.herokuapp.com/ deployed to Heroku

To [emailprotected]:agile-sierra-1405.git
* [new branch]      master -> master
```

Your application should now be up and running on Heroku.

## 63.3 OpenShift

[OpenShift](https://www.openshift.com/) is the Red Hat public (and enterprise) extension of the Kubernetes container orchestration platform. Similarly to Kubernetes, OpenShift has many options for installing Spring Boot based applications.

OpenShift has many resources describing how to deploy Spring Boot applications, including:

- [Using the S2I builder](https://blog.openshift.com/using-openshift-enterprise-grade-spring-boot-deployments/)

- [Architecture guide](https://access.redhat.com/documentation/en-us/reference_architectures/2017/html-single/spring_boot_microservices_on_red_hat_openshift_container_platform_3/)

- [Running as a traditional web application on Wildfly](https://blog.openshift.com/using-spring-boot-on-openshift/)

- [OpenShift Commons Briefing](https://blog.openshift.com/openshift-commons-briefing-96-cloud-native-applications-spring-rhoar/)

## 63.4 Amazon Web Services (AWS)

Amazon Web Services offers multiple ways to install Spring Boot-based applications, either as traditional web applications (war) or as executable jar files with an embedded web server. The options include:

- AWS Elastic Beanstalk

- AWS Code Deploy

- AWS OPS Works

- AWS Cloud Formation

- AWS Container Registry

Each has different features and pricing models. In this document, we describe only the simplest option: AWS Elastic Beanstalk.

### 63.4.1 AWS Elastic Beanstalk

As described in the official [Elastic Beanstalk Java guide](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Java.html), there are two main options to deploy a Java application. You can either use the “Tomcat Platform” or the “Java SE platform”.

#### Using the Tomcat Platform

This option applies to Spring Boot projects that produce a war file. No special configuration is required. You need only follow the official guide.

#### Using the Java SE Platform

This option applies to Spring Boot projects that produce a jar file and run an embedded web container. Elastic Beanstalk environments run an nginx instance on port 80 to proxy the actual application, running on port 5000. To configure it, add the following line to your  `application.properties`  file:

```java
server.port=5000
```

> By default, Elastic Beanstalk uploads sources and compiles them in AWS. However, it is best to upload the binaries instead. To do so, add lines similar to the following to your  `.elasticbeanstalk/config.yml`  file:

> By default an Elastic Beanstalk environment is load balanced. The load balancer has a significant cost. To avoid that cost, set the environment type to “Single instance”, as described in [the Amazon documentation](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-create-wizard.html#environments-create-wizard-capacity). You can also create single instance environments by using the CLI and the following command:

### 63.4.2 Summary

This is one of the easiest ways to get to AWS, but there are more things to cover, such as how to integrate Elastic Beanstalk into any CI / CD tool, use the Elastic Beanstalk Maven plugin instead of the CLI, and others. There is a [blog post](https://exampledriven.wordpress.com/2017/01/09/spring-boot-aws-elastic-beanstalk-example/) covering these topics more in detail.

## 63.5 Boxfuse and Amazon Web Services

[Boxfuse](https://boxfuse.com/) works by turning your Spring Boot executable jar or war into a minimal VM image that can be deployed unchanged either on VirtualBox or on AWS. Boxfuse comes with deep integration for Spring Boot and uses the information from your Spring Boot configuration file to automatically configure ports and health check URLs. Boxfuse leverages this information both for the images it produces as well as for all the resources it provisions (instances, security groups, elastic load balancers, and so on).

Once you have created a [Boxfuse account](https://console.boxfuse.com), connected it to your AWS account, installed the latest version of the Boxfuse Client, and ensured that the application has been built by Maven or Gradle (by using, for example,  `mvn clean package` ), you can deploy your Spring Boot application to AWS with a command similar to the following:

```java
$ boxfuse run myapp-1.0.jar -env=prod
```

See the [boxfuse run documentation](https://boxfuse.com/docs/commandline/run.html) for more options. If there is a [boxfuse.conf](https://boxfuse.com/docs/commandline/#configuration) file present in the current directory, it is considered.

> By default, Boxfuse activates a Spring profile named  `boxfuse`  on startup. If your executable jar or war contains an [application-boxfuse.properties](https://boxfuse.com/docs/payloads/springboot.html#configuration) file, Boxfuse bases its configuration on the properties it contains.

At this point,  `boxfuse`  creates an image for your application, uploads it, and configures and starts the necessary resources on AWS, resulting in output similar to the following example:

```java
Fusing Image for myapp-1.0.jar ...
Image fused in 00:06.838s (53937 K) -> axelfontaine/myapp:1.0
Creating axelfontaine/myapp ...
Pushing axelfontaine/myapp:1.0 ...
Verifying axelfontaine/myapp:1.0 ...
Creating Elastic IP ...
Mapping myapp-axelfontaine.boxfuse.io to 52.28.233.167 ...
Waiting for AWS to create an AMI for axelfontaine/myapp:1.0 in eu-central-1 (this may take up to 50 seconds) ...
AMI created in 00:23.557s -> ami-d23f38cf
Creating security group boxfuse-sg_axelfontaine/myapp:1.0 ...
Launching t2.micro instance of axelfontaine/myapp:1.0 (ami-d23f38cf) in eu-central-1 ...
Instance launched in 00:30.306s -> i-92ef9f53
Waiting for AWS to boot Instance i-92ef9f53 and Payload to start at http://52.28.235.61/ ...
Payload started in 00:29.266s -> http://52.28.235.61/
Remapping Elastic IP 52.28.233.167 to i-92ef9f53 ...
Waiting 15s for AWS to complete Elastic IP Zero Downtime transition ...
Deployment completed successfully. axelfontaine/myapp:1.0 is up and running at http://myapp-axelfontaine.boxfuse.io/
```

Your application should now be up and running on AWS.

See the blog post on [deploying Spring Boot apps on EC2](https://boxfuse.com/blog/spring-boot-ec2.html) as well as the [documentation for the Boxfuse Spring Boot integration](https://boxfuse.com/docs/payloads/springboot.html) to get started with a Maven build to run the app.

## 63.6 Google Cloud

Google Cloud has several options that can be used to launch Spring Boot applications. The easiest to get started with is probably App Engine, but you could also find ways to run Spring Boot in a container with Container Engine or on a virtual machine with Compute Engine.

To run in App Engine, you can create a project in the UI first, which sets up a unique identifier for you and also sets up HTTP routes. Add a Java app to the project and leave it empty and then use the [Google Cloud SDK](https://cloud.google.com/sdk/downloads) to push your Spring Boot app into that slot from the command line or CI build.

App Engine Standard requires you to use WAR packaging. Follow [these steps](https://github.com/GoogleCloudPlatform/getting-started-java/blob/master/appengine-standard-java8/springboot-appengine-standard/README.md) to deploy App Engine Standard application to Google Cloud.

Alternatively, App Engine Flex requires you to create an  `app.yaml`  file to describe the resources your app requires. Normally, you put this file in  `src/main/appengine` , and it should resemble the following file:

```java
service: default

runtime: java
env: flex

runtime_config:
jdk: openjdk8

handlers:
- url: /.*
script: this field is required, but ignored

manual_scaling:
instances: 1

health_check:
enable_health_check: False

env_variables:
ENCRYPT_KEY: your_encryption_key_here
```

You can deploy the app (for example, with a Maven plugin) by adding the project ID to the build configuration, as shown in the following example:

```xml
<plugin>
	<groupId>com.google.cloud.tools</groupId>
	<artifactId>appengine-maven-plugin</artifactId>
	<version>1.3.0</version>
	<configuration>
		<project>myproject</project>
	</configuration>
</plugin>
```

Then deploy with  `mvn appengine:deploy`  (if you need to authenticate first, the build fails).

