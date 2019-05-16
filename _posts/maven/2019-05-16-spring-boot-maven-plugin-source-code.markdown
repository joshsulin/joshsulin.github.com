---
layout: post
title:  "spring-boot-maven-plugin插件源码分析"
date:   2019-05-16 15:35:15 +0800
categories: Maven
tag: Maven
---

初始化Spring-Boot项目, 在pom.xml文件里, 都需要一个插件.

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

在spring-boot-starter-parent里面, 能找到如下配置.

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>repackage</id>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <mainClass>${start-class}</mainClass>
    </configuration>
</plugin>
```

结合上一篇博客, 查看该插件目标对应的Mojo描述符, 可知, 该插件作用于 PACKAGE 阶段.

```java
// Mojo描述符
@Mojo(name = "repackage", defaultPhase = LifecyclePhase.PACKAGE, requiresProject = true,
		threadSafe = true,
		requiresDependencyResolution = ResolutionScope.COMPILE_PLUS_RUNTIME,
		requiresDependencyCollection = ResolutionScope.COMPILE_PLUS_RUNTIME)
public class RepackageMojo extends AbstractDependencyFilterMojo {
    ......
    ......
    ......
}
```

对于默认的Spring-Boot项目, 执行mvn install后, 需要由如下插件完成.

#### 针对 jar 打包方式

| 阶段                    | jar     |
| --------                | :-----  |
| validate                |         |
| initialize              |         |
| generate-sources        |         |
| process-sources         |         |
| generate-resources      |         |
| process-resources       | org.apache.maven.plugins:maven-resources-plugin:2.6:resources |
| compile                 | org.apache.maven.plugins:maven-compiler-plugin:3.1:compile |
| process-classes         |         |
| generate-test-sources   |         |
| process-test-sources    |         |
| generate-test-resources | org.apache.maven.plugins:maven-resources-plugin:2.6:testResources |
| process-test-resources  |         |
| test-compile            | org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile |
| process-test-classes    |         |
| test                    | org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test |
| prepare-package         |         |
| package                 | org.apache.maven.plugins:maven-jar-plugin:3.1.1:jar、`org.springframework.boot:spring-boot-maven-plugin:2.1.4.RELEASE:repackage {execution: repackage}` |
| pre-integration-test    |         |
| integration-test        |         |
| post-integration-test   |         |
| verify                  |         |
| install                 | org.apache.maven.plugins:maven-install-plugin:2.4:install |
| deploy                  | org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy   |

package接受编译好的代码，打包成可发布的格式，如JAR.

针对于同一个spring-boot项目, 引入插件 spring-boot-maven-plugin 与 不引入该插件, 打出来的jar包内容是不一样的. 其实最主要的区别就是 可执行jar包与不可执行jar包.

可执行jar包与不可执行jar包 唯一的区别就是jar包里面的MANIFEST.MF 里面有Main-Class指定。

引入插件 spring-boot-maven-plugin, MANIFEST.MF文件内容

```java
Manifest-Version: 1.0
Implementation-Title: plugin-study
Implementation-Version: 0.0.1-SNAPSHOT
Start-Class: com.josh.pluginstudy.PluginStudyApplication
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Build-Jdk-Spec: 1.8
Spring-Boot-Version: 2.1.5.RELEASE
Created-By: Maven Archiver 3.4.0
Main-Class: org.springframework.boot.loader.JarLauncher
```

不引入插件 spring-boot-maven-plugin, MANIFEST.MF文件内容

```java
Manifest-Version: 1.0
Implementation-Title: plugin-study
Implementation-Version: 0.0.1-SNAPSHOT
Build-Jdk-Spec: 1.8
Created-By: Maven Archiver 3.4.0
```

通过以上内容的区别, 大体应该知道 spring-boot-maven-plugin <goal>repackage</goal> 的作用了.

接下来我们来走一遍该插件源码

```java
@Mojo(name = "repackage", defaultPhase = LifecyclePhase.PACKAGE, requiresProject = true,
    threadSafe = true,
    requiresDependencyResolution = ResolutionScope.COMPILE_PLUS_RUNTIME,
    requiresDependencyCollection = ResolutionScope.COMPILE_PLUS_RUNTIME)
public class RepackageMojo extends AbstractDependencyFilterMojo {

    @Parameter(property = "spring-boot.repackage.skip", defaultValue = "false")
    private boolean skip;

    @Override
    public void execute() throws MojoExecutionException, MojoFailureException {
        if (this.project.getPackaging().equals("pom")) {
            getLog().debug("repackage goal could not be applied to pom project.");
            return;
        }
        if (this.skip) {
            getLog().debug("skipping repackaging as per configuration.");
            return;
        }
        repackage();
    }

}
```

核心逻辑在repackage()方法里面

```java
private void repackage() throws MojoExecutionException {
    // source的值->com.josh:plugin-study:jar:0.0.1-SNAPSHOT
    Artifact source = getSourceArtifact();
    // target的值->项目target下面的jar包 /Users/sulin/project/java/study/plugin-study/target/plugin-study-0.0.1-SNAPSHOT.jar
    File target = getTargetFile();
    Repackager repackager = getRepackager(source.getFile());
    Set<Artifact> artifacts = filterDependencies(this.project.getArtifacts(), getFilters(getAdditionalFilters()));
    Libraries libraries = new ArtifactsLibraries(artifacts, this.requiresUnpack, getLog());
    try {
        LaunchScript launchScript = getLaunchScript();
        repackager.repackage(target, libraries, launchScript);
    }
    catch (IOException ex) {
        throw new MojoExecutionException(ex.getMessage(), ex);
    }
    updateArtifact(source, target, repackager.getBackupFile());
}
```

核心逻辑为 repackager.repackage(target, libraries, launchScript);

```java
public void repackage(File destination, Libraries libraries, LaunchScript launchScript) throws IOException {
    // 重新打包进行条件判断 start
    if (destination == null || destination.isDirectory()) {
      throw new IllegalArgumentException("Invalid destination");
    }
    if (libraries == null) {
      throw new IllegalArgumentException("Libraries must not be null");
    }
    if (this.layout == null) {
      this.layout = getLayoutFactory().getLayout(this.source);
    }
    destination = destination.getAbsoluteFile();
    File workingSource = this.source;
    if (alreadyRepackaged() && this.source.equals(destination)) {
      return;
    }
    // 重新打包进行条件判断 end

    // 这段代码, 解释了为什么spring-boot项目, 有 .jar.original 和 .jar两个文件
    // .jar.original 是备份文件, 备份了运行该插件前, 由maven-jar-plugin插件生成的jar包
    if (this.source.equals(destination)) {
        workingSource = getBackupFile();
        workingSource.delete();
        renameFile(this.source, workingSource);
    }

    destination.delete();
    try {
        try (JarFile jarFileSource = new JarFile(workingSource)) {
            // 核心逻辑
            repackage(jarFileSource, destination, libraries, launchScript);
        }
    }
    finally {
        if (!this.backupSource && !this.source.equals(workingSource)) {
            deleteFile(workingSource);
        }
    }
}
```

核心逻辑 repackage(jarFileSource, destination, libraries, launchScript);

```java
private void repackage(JarFile sourceJar, File destination, Libraries libraries, LaunchScript launchScript) throws IOException {
    WritableLibraries writeableLibraries = new WritableLibraries(libraries);
    try (JarWriter writer = new JarWriter(destination, launchScript)) {
        // 核心代码, 将新的MANIFEST.MF写入jar包
        writer.writeManifest(buildManifest(sourceJar));
        writeLoaderClasses(writer);
        if (this.layout instanceof RepackagingLayout) {
          writer.writeEntries(sourceJar, new RenamingEntryTransformer(
              ((RepackagingLayout) this.layout).getRepackagedClassesLocation()),
              writeableLibraries);
        }
        else {
          writer.writeEntries(sourceJar, writeableLibraries);
        }
        writeableLibraries.write(writer);
    }
}
```

接下来, 代码细节, 不再贴出, 主要对MANIFEST.MF文件中, Main-Class、Start-Class两个属性的值进行说明.

Main-Class: return "org.springframework.boot.loader.JarLauncher"; 这是写死的

Start-Class: 逻辑稍微有点复杂

```java
manifest = new Manifest(manifest);
String startClass = this.mainClass;
if (startClass == null) {
    // MAIN_CLASS_ATTRIBUTE -> "Main-Class"
    startClass = manifest.getMainAttributes().getValue(MAIN_CLASS_ATTRIBUTE);
}
if (startClass == null) {
    // 在给定的jar文件中查找单个主类。 使用给定{@code annotationName}的注释的主类将优先于没有此类注释的主类。
    // annotationName -> org.springframework.boot.autoconfigure.SpringBootApplication
    startClass = findMainMethodWithTimeoutWarning(source);
}
```
