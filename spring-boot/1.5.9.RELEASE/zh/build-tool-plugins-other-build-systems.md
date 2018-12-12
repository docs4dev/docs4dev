## 74.支持其他构建系统

如果您想使用Maven，Gradle或Ant以外的构建工具，您可能需要开发自己的插件.可执行jar需要遵循特定的格式，并且某些条目需要以未压缩的形式写入（有关详细信息，请参阅附录中的“[executable jar format](executable-jar.html)”部分）.

Spring Boot Maven和Gradle插件都使用 `spring-boot-loader-tools` 来实际生成jar.如果需要，您可以直接使用此库.

## 74.1重新包装档案

要重新打包现有存档以使其成为自包含的可执行存档，请使用 `org.springframework.boot.loader.tools.Repackager` .  `Repackager` 类采用一个构造函数参数，该参数引用现有的jar或war存档.使用两种可用的 `repackage()` 方法之一替换原始文件或写入新目标.在重新打包程序运行之前，还可以配置各种设置.

## 74.2嵌套库

重新打包存档时，可以使用 `org.springframework.boot.loader.tools.Libraries` 接口包含对依赖项文件的引用.我们在这里没有提供 `Libraries` 的任何具体实现，因为它们通常是特定于构建系统的.

如果您的存档已包含库，则可以使用 `Libraries.NONE` .

## 74.3寻找主类

如果不使用 `Repackager.setMainClass()` 指定主类，则repackager使用[ASM](http://asm.ow2.org/)读取类文件并尝试使用 `public static void main(String[] args)` 方法查找合适的类.如果找到多个候选项，则抛出异常.

## 74.4重新包装实施示例

以下示例显示了典型的重新打包实现：

```java
Repackager repackager = new Repackager(sourceJarFile);
repackager.setBackupSource(false);
repackager.repackage(new Libraries() {
			@Override
			public void doWithLibraries(LibraryCallback callback) throws IOException {
				// Build system specific implementation, callback for each dependency
				// callback.library(new Library(nestedFile, LibraryScope.COMPILE));
			}
		});
```

