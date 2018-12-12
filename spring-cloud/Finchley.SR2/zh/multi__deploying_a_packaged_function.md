## 129.部署打包功能

Spring Cloud Function提供了一个"deployer"库，允许您使用隔离的类加载器启动jar文件（或展开的存档或jar文件集），并公开其中定义的函数.这是一个非常强大的工具，允许您在不更改目标jar文件的情况下，将函数调整为一系列不同的输入输出适配器.无服务器平台通常具有内置的这种功能，因此您可以将其视为此类平台中函数调用程序的构建块（实际上[Riff](https://projectriff.io) Java函数调用程序使用此库）.

API的标准入口点是Spring配置注释 `@EnableFunctionDeployer` .如果在Spring Boot应用程序中使用它，则部署者会启动并查找某些配置以告诉它在哪里找到函数jar.用户必须至少提供 `function.location` ，这是包含这些功能的存档的URL或资源位置.它可以选择使用 `maven:` 前缀通过依赖项查找来定位工件（有关完整的详细信息，请参阅 `FunctionProperties` ）. Spring引导应用程序是从jar文件引导的，使用 `MANIFEST.MF` 来定位一个启动类，以便标准的Spring Boot胖jar工作正常.如果目标jar可以成功启动，那么结果是在主应用程序的 `FunctionCatalog` 中注册的函数.注册函数可以通过主应用程序中的代码应用，即使它是在隔离的类加载器中创建的（通过deault）.
