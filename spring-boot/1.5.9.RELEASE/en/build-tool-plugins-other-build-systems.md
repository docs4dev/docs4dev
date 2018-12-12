## 74. Supporting Other Build Systems

If you want to use a build tool other than Maven, Gradle, or Ant, you likely need to develop your own plugin. Executable jars need to follow a specific format and certain entries need to be written in an uncompressed form (see the “[executable jar format](executable-jar.html)” section in the appendix for details).

The Spring Boot Maven and Gradle plugins both make use of  `spring-boot-loader-tools`  to actually generate jars. If you need to, you may use this library directly.

## 74.1 Repackaging Archives

To repackage an existing archive so that it becomes a self-contained executable archive, use  `org.springframework.boot.loader.tools.Repackager` . The  `Repackager`  class takes a single constructor argument that refers to an existing jar or war archive. Use one of the two available  `repackage()`  methods to either replace the original file or write to a new destination. Various settings can also be configured on the repackager before it is run.

## 74.2 Nested Libraries

When repackaging an archive, you can include references to dependency files by using the  `org.springframework.boot.loader.tools.Libraries`  interface. We do not provide any concrete implementations of  `Libraries`  here as they are usually build-system-specific.

If your archive already includes libraries, you can use  `Libraries.NONE` .

## 74.3 Finding a Main Class

If you do not use  `Repackager.setMainClass()`  to specify a main class, the repackager uses [ASM](http://asm.ow2.org/) to read class files and tries to find a suitable class with a  `public static void main(String[] args)`  method. An exception is thrown if more than one candidate is found.

## 74.4 Example Repackage Implementation

The following example shows a typical repackage implementation:

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

