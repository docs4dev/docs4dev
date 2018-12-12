## 63.部署到Cloud

Spring Boot的可执行jar是现成的，适用于大多数流行的CloudPaaS（平台即服务）提供商.这些提供商往往要求您“自带容器”.它们管理应用程序进程（而不是Java应用程序），因此它们需要一个中间层，使您的应用程序适应Cloud的运行过程概念.

两个流行的Cloud提供商Heroku和Cloud Foundry采用“buildpack”方法. buildpack将您部署的代码包装在启动应用程序所需的任何内容中.它可能是一个JDK，也是对 `java` 的调用，一个嵌入式Web服务器，或一个成熟的应用服务器. buildpack是可插拔的，但理想情况下，您应该能够尽可能少地进行自定义.这减少了不受您控制的功能的占用空间.它最大限度地减少了开发和生产环境环境之间的差异.

理想情况下，您的应用程序（如Spring Boot可执行jar）具有在其中运行打包所需的所有内容.

在本节中，我们将了解如何在“入门”部分中启动[simple application that we developed](getting-started-first-application.html)并在Cloud中运行.

## 63.1 Cloud Foundry

如果未指定其他buildpack，Cloud Foundry将提供默认的构建包. Cloud Foundry [Java buildpack](https://github.com/cloudfoundry/java-buildpack)对Spring应用程序提供了出色的支持，包括Spring Boot.您可以部署独立的可执行jar应用程序以及传统的 `.war` 打包应用程序.

构建应用程序（例如，使用 `mvn clean package` ）并使用[installed the cf command line tool](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)后，使用 `cf push` 命令部署应用程序，将路径替换为已编译的 `.jar` .在推送应用程序之前一定要[logged in with your cf command line client](https://docs.cloudfoundry.org/cf-cli/getting-started.html#login).以下行显示使用 `cf push` 命令部署应用程序：

```java
$ cf push acloudyspringtime -p target/demo-0.0.1-SNAPSHOT.jar
```

> 在前面的示例中，我们将 `acloudyspringtime` 替换为您给出的任何值 `cf` 作为应用程序的名称.

有关更多选项，请参阅[cf push documentation](https://docs.cloudfoundry.org/cf-cli/getting-started.html#push).如果同一目录中存在Cloud Foundry [manifest.yml](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)文件，则会考虑该文件.

此时， `cf` 开始上传您的应用程序，生成类似于以下示例的输出：

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

恭喜！该应用程序现已上线！

应用程序运行后，您可以使用 `cf apps` 命令验证已部署应用程序的状态，如以下示例所示：

```java
$ cf apps
Getting applications in ...
OK

name                 requested state   instances   memory   disk   urls
...
acloudyspringtime    started           1/1         512M     1G     acloudyspringtime.cfapps.io
...
```

一旦Cloud Foundry确认您的应用程序已部署，您应该能够在给定的URI处找到该应用程序.在前面的示例中，您可以在 `http://acloudyspringtime.cfapps.io/` 找到它.

### 63.1.1绑定到服务

默认情况下，有关正在运行的应用程序的元数据以及服务连接信息将作为环境变量公开给应用程序（例如： `$VCAP_SERVICES` ）.此体系结构决策归功于Cloud Foundry的多语言（任何语言和平台都可以作为buildpack支持）.进程范围的环境变量与语言无关.

环境变量并不总是适用于最简单的API，因此Spring Boot会自动提取它们并将数据展平为可通过以下方式访问的属性Spring的 `Environment` 抽象，如下例所示：

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

所有Cloud Foundry属性都以 `vcap` 为前缀.您可以使用 `vcap` 属性来访问应用程序信息（例如应用程序的公共URL）和服务信息（例如数据库凭据）.有关完整的详细信息，请参阅[‘CloudFoundryVcapEnvironmentPostProcessor’](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/cloud/CloudFoundryVcapEnvironmentPostProcessor.html) Javadoc.

>  [Spring Cloud Connectors](https://cloud.spring.io/spring-cloud-connectors/)项目更适合配置DataSource等任务. Spring Boot包括自动配置支持和 `spring-boot-starter-cloud-connectors` 启动器.

## 63.2 Heroku

Heroku是另一个流行的PaaS平台.要自定义Heroku构建，请提供 `Procfile` ，它提供部署应用程序所需的咒语. Heroku为要使用的Java应用程序分配 `port` ，然后确保路由到外部URI工作.

您必须将应用程序配置为侦听正确的端口.以下示例显示了启动REST应用程序的 `Procfile` ：

```java
web: java -Dserver.port=$PORT -jar target/demo-0.0.1-SNAPSHOT.jar
```

Spring Boot使 `-D` 参数可用作可从Spring  `Environment` 实例访问的属性.  `server.port` 配置属性被馈送到嵌入式Tomcat，Jetty或Undertow实例，然后在启动时使用该端口. Heroku PaaS将 `$PORT` 环境变量分配给我们.

这应该是你需要的一切. Heroku部署的最常见部署工作流程是 `git push` 生产环境代码，如以下示例所示：

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

您的应用程序现在应该在Heroku上启动并运行.

## 63.3 OpenShift

[OpenShift](https://www.openshift.com/)是Kubernetes容器编排平台的Red Hat公共（和企业）扩展.与Kubernetes类似，OpenShift有许多选项可用于安装基于Spring Boot的应用程序.

OpenShift有许多资源描述如何部署Spring Boot应用程序，包括：

- [Using the S2I builder](https://blog.openshift.com/using-openshift-enterprise-grade-spring-boot-deployments/)

- [Architecture guide](https://access.redhat.com/documentation/en-us/reference_architectures/2017/html-single/spring_boot_microservices_on_red_hat_openshift_container_platform_3/)

- [Running as a traditional web application on Wildfly](https://blog.openshift.com/using-spring-boot-on-openshift/)

- [OpenShift Commons Briefing](https://blog.openshift.com/openshift-commons-briefing-96-cloud-native-applications-spring-rhoar/)

## 63.4亚马逊网络服务（AWS）

Amazon Web Services提供了多种方法来安装基于Spring Boot的应用程序，可以是传统的Web应用程序（war），也可以是带有嵌入式Web服务器的可执行jar文件.选项包括：

- AWS Elastic Beanstalk

- AWS代码部署

- AWS OPS工程

- AWSCloud形成

- AWS容器注册表

每个都有不同的功能和定价模型.在本文档中，我们仅描述了最简单的选项：AWS Elastic Beanstalk.

### 63.4.1 AWS Elastic Beanstalk

如官方[Elastic Beanstalk Java guide](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Java.html)中所述，部署Java应用程序有两个主要选项.您可以使用“Tomcat平台”或“Java SE平台”.

#### 使用Tomcat平台

此选项适用于生成war文件的Spring Boot项目.无需特殊配置.您只需遵循官方指南即可.

#### 使用Java SE平台

此选项适用于生成jar文件并运行嵌入式Web容器的Spring Boot项目. Elastic Beanstalk环境在端口80上运行nginx实例以代理在端口5000上运行的实际应用程序.要配置它，请将以下行添加到 `application.properties` 文件中：

```java
server.port=5000
```

> By默认情况下，Elastic Beanstalk上传源并在AWS中编译它们.但是，最好上传二进制文件.为此，请在 `.elasticbeanstalk/config.yml` 文件中添加与以下内容类似的行：

> By默认Elastic Beanstalk环境是负载平衡的.负载平衡器具有显着的成本.要避免该成本，请将环境类型设置为“单实例”，如[the Amazon documentation](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-create-wizard.html#environments-create-wizard-capacity)中所述.您还可以使用CLI和以下命令创建单实例环境：

### 63.4.2摘要

这是访问AWS的最简单方法之一，但还有更多内容需要涉及，例如如何将Elastic Beanstalk集成到任何CI / CD工具中，使用Elastic Beanstalk Maven插件而不是CLI等.有一个[blog post](https://exampledriven.wordpress.com/2017/01/09/spring-boot-aws-elastic-beanstalk-example/)更详细地介绍了这些主题.

## 63.5 Boxfuse和亚马逊网络服务

[Boxfuse](https://boxfuse.com/)的工作原理是将Spring Boot可执行jar或war转换为可在VirtualBox或AWS上无需部署的最小VM映像. Boxfuse附带了Spring Boot的深度集成，并使用Spring Boot配置文件中的信息自动配置端口和运行状况检查URL. Boxfuse利用这些信息来处理它产生的图像以及它提供的所有资源（实例，安全组，弹性负载平衡器等）.

创建[Boxfuse account](https://console.boxfuse.com)后，将其连接到您的AWS账户，安装最新版本的Boxfuse Client，并确保该应用程序是由Maven或Gradle构建的（例如，使用 `mvn clean package` ），您可以部署Spring使用类似于以下的命令将应用程序引导至AWS：

```java
$ boxfuse run myapp-1.0.jar -env=prod
```

有关更多选项，请参阅[boxfuse run documentation](https://boxfuse.com/docs/commandline/run.html).如果当前目录中存在[boxfuse.conf](https://boxfuse.com/docs/commandline/#configuration)文件，则会考虑该文件.

> By默认情况下，Boxfuse在启动时激活名为 `boxfuse` 的Spring配置文件.如果您的可执行jar或war包含[application-boxfuse.properties](https://boxfuse.com/docs/payloads/springboot.html#configuration)文件，Boxfuse将其配置基于它包含的属性.

此时， `boxfuse` 为您的应用程序创建一个映像，上传它，并在AWS上配置和启动必要的资源，从而产生类似于以下示例的输出：

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

您的应用程序现在应该在AWS上启动并运行.

请参阅[deploying Spring Boot apps on EC2](https://boxfuse.com/blog/spring-boot-ec2.html)上的博客文章以及[documentation for the Boxfuse Spring Boot integration](https://boxfuse.com/docs/payloads/springboot.html)以开始使用Maven构建来运行该应用程序.

## 63.6 Google Cloud

Google Cloud有几个选项可用于启动Spring Boot应用程序.最容易上手的可能是App Engine，但您也可以找到在带有Container Engine的容器中运行Spring Boot的方法，或者在具有Compute Engine的虚拟机上运行Spring Boot的方法.

要在App Engine中运行，您可以首先在UI中创建项目，该项目为您设置唯一标识符并设置HTTP路由.将Java应用程序添加到项目中并将其留空，然后使用[Google Cloud SDK](https://cloud.google.com/sdk/downloads)将Spring Boot应用程序从命令行或CI构建推送到该插槽.

App Engine Standard要求您使用WAR包装.按照[these steps](https://github.com/GoogleCloudPlatform/getting-started-java/blob/master/appengine-standard-java8/springboot-appengine-standard/README.md)将App Engine标准应用程序部署到Google Cloud.

或者，App Engine Flex要求您创建 `app.yaml` 文件以描述您的应用所需的资源.通常，您将此文件放在 `src/main/appengine` 中，它应该类似于以下文件：

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

您可以通过将项目ID添加到构建配置来部署应用程序（例如，使用Maven插件），如以下示例所示：

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

然后使用 `mvn appengine:deploy` 进行部署（如果需要先进行身份验证，则构建失败）.

