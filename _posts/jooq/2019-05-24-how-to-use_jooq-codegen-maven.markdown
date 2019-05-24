---
layout: post
title:  "jooq-codegen-maven插件入门"
date:   2019-05-24 16:19:22 +0800
categories: Jooq
tag: Jooq
---

非常全的插件文档 <a href="https://www.jooq.org/doc/3.11/manual/code-generation/" target="_blank">请查看官方文档</a>

There is no substantial difference between running the code generator with Maven or in standalone mode.
使用Maven运行代码生成器或在独立模式下没有实质性区别。

Both modes use the exact same <configuration/> element. The Maven plugin configuration adds some additional boilerplate around that:
两种模式都使用完全相同的<configuration />元素。 Maven插件配置增加了一些额外的样板：

```java
<plugin>
  <!-- Specify the maven code generator plugin -->
  <!-- Use org.jooq            for the Open Source Edition
           org.jooq.pro        for commercial editions, 
           org.jooq.pro-java-6 for commercial editions with Java 6 support,
           org.jooq.trial      for the free trial edition 
         
       Note: Only the Open Source Edition is hosted on Maven Central. 
             Import the others manually from your distribution -->
  <groupId>org.jooq</groupId>
  <artifactId>jooq-codegen-maven</artifactId>
  <version>3.11.11</version>

  <executions>
    <execution>
      <id>jooq-codegen</id>
      <phase>generate-sources</phase>
      <goals>
        <goal>generate</goal>
      </goals>
      <configuration>
        ...
      </configuration>
    </execution>
  </executions>
</plugin>
```

** Additional Maven-specific flags **

There are, however, some additional, Maven-specific flags that can be specified with the jooq-codegen-maven plugin only:
但是，还有一些额外的Maven特定标志，只能使用jooq-codegen-maven插件指定：

```java
<plugin>
  ...
	<configuration>

		<!-- A boolean property (or constant) can be specified here to tell the plugin not to do anything -->
		<!-- 这里可以指定一个布尔属性（或常量）来告诉插件不要做任何事情 -->
		<skip>${skip.jooq.generation}</skip>

		<!-- Instead of providing an inline configuration here, you can specify an external XML configuration file here -->
		<!-- 您可以在此处指定外部XML配置文件，而不是在此处提供内联配置 -->
		<configurationFile>${externalfile}</configurationFile>
	</configuration>
  ...
</plugin>
```

** 插件实现的源码大体轮廓 **

```java

......
@Parameter(
		property = "jooq.codegen.configurationFile"
)
private String configurationFile;

@Parameter(
		property = "jooq.codegen.skip"
)
private boolean skip;

@Parameter
private org.jooq.meta.jaxb.Generator generator;

public void execute() throws MojoExecutionException {
		if (skip) {
			getLog().info("Skipping jOOQ code generation");
			return;
		}

		if (configurationFile != null) {
			......
			generator = configuration.getGenerator();
			......
		}

		if (generator == null) {
			// jOOQ代码生成工具的配置不正确 
			getLog().error("Incorrect configuration of jOOQ code generation tool");
			/**
			jOOQ-codegen-maven模块的生成器配置未正确设置。
			这可能有多种原因，其中包括：
			 - 根据jooq-codegen-3.11.0.xsd，您的pom.xml的<configuration>包含无效的XML
			 - 您的pom.xml和命令行之间存在版本或工件不匹配
			**/
			getLog().error(
					"\n"
					+ "The jOOQ-codegen-maven module's generator configuration is not set up correctly.\n"
					+ "This can have a variety of reasons, among which:\n"
					+ "- Your pom.xml's <configuration> contains invalid XML according to " + XSD_CODEGEN + "\n"
					+ "- There is a version or artifact mismatch between your pom.xml and your commandline");

			throw new MojoExecutionException("Incorrect configuration of jOOQ code generation tool. See error above for details.");
		}
}
......
```

通过上述插件源码, 可以清楚, 没有不配置 configurationFile, 插件是要报错的.

While optional, source code generation is one of jOOQ's main assets if you wish to increase developer productivity. jOOQ's code generator takes your database schema and reverse-engineers it into a set of Java classes modelling tables, records, sequences, POJOs, DAOs, stored procedures, user-defined types and many more.
虽然可选，但如果您希望提高开发人员的工作效率，源代码生成是jOOQ的主要资产之一。 jOOQ的代码生成器采用您的数据库模式并将其反向工程为一组Java类建模表，记录，序列，POJO，DAO，存储过程，用户定义类型等等。

The essential ideas behind source code generation are these:
源代码生成背后的基本思想是：

Increased IDE support: Type your Java code directly against your database schema, with all type information available
Type-safety: When your database schema changes, your generated code will change as well. Removing columns will lead to compilation errors, which you can detect early.
The following chapters will show how to configure the code generator and how to generate various artefacts.
增加IDE支持：直接针对数据库模式键入Java代码，并提供所有类型信息
类型安全：当您的数据库架构发生更改时，生成的代码也会更改。 删除列将导致编译错误，您可以尽早检测到。

以下章节将介绍如何配置代码生成器以及如何生成各种伪像。

### 1.1 Configuration and setup of the generator(生成器配置与安装)

#### Configure jOOQ's code generator

You need to tell jOOQ some things about your database connection. Here's an example of how to do it for an Oracle database
你需要告诉jOOQ你的数据库连接的一些事情。 以下是如何为Oracle数据库执行此操作的示例

```java
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration xmlns="http://www.jooq.org/xsd/jooq-codegen-3.11.0.xsd">
  <!-- Configure the database connection here -->
	<!-- 在此处配置数据库连接 -->
  <jdbc>
    <driver>oracle.jdbc.OracleDriver</driver>
    <url>jdbc:oracle:thin:@[your jdbc connection parameters]</url>
    <user>[your database user]</user>
    <password>[your database password]</password>
    
    <!-- You can also pass user/password and other JDBC properties in the optional properties tag: -->
		<!-- 您还可以在可选属性标记中传递用户/密码和其他JDBC属性 -->
    <properties>
      <property><key>user</key><value>[db-user]</value></property>
      <property><key>password</key><value>[db-password]</value></property>
    </properties>
  </jdbc>

  <generator>
    <database>
      <!-- The database dialect from jooq-meta. Available dialects are named org.jooq.meta.[database].[database]Database.
					 来自jooq-meta的数据库方言。 可用的方言名为org.jooq.meta.[database].[database]数据库
            
           Natively supported values are:
					 支持的值是
    
               org.jooq.meta.ase.ASEDatabase
               org.jooq.meta.auroramysql.AuroraMySQLDatabase
               org.jooq.meta.aurorapostgres.AuroraPostgresDatabase
               org.jooq.meta.cubrid.CUBRIDDatabase
               org.jooq.meta.db2.DB2Database
               org.jooq.meta.derby.DerbyDatabase
               org.jooq.meta.firebird.FirebirdDatabase
               org.jooq.meta.h2.H2Database
               org.jooq.meta.hana.HANADatabase
               org.jooq.meta.hsqldb.HSQLDBDatabase
               org.jooq.meta.informix.InformixDatabase
               org.jooq.meta.ingres.IngresDatabase
               org.jooq.meta.mariadb.MariaDBDatabase
               org.jooq.meta.mysql.MySQLDatabase
               org.jooq.meta.oracle.OracleDatabase
               org.jooq.meta.postgres.PostgresDatabase
               org.jooq.meta.redshift.RedshiftDatabase
               org.jooq.meta.sqldatawarehouse.SQLDataWarehouseDatabase
               org.jooq.meta.sqlite.SQLiteDatabase
               org.jooq.meta.sqlserver.SQLServerDatabase
               org.jooq.meta.sybase.SybaseDatabase
               org.jooq.meta.teradata.TeradataDatabase
               org.jooq.meta.vertica.VerticaDatabase
            
           This value can be used to reverse-engineer generic JDBC DatabaseMetaData (e.g. for MS Access)
           
               org.jooq.meta.jdbc.JDBCDatabase
                
           This value can be used to reverse-engineer standard jOOQ-meta XML formats
            
               org.jooq.meta.xml.XMLDatabase
    
           This value can be used to reverse-engineer schemas defined by SQL files (requires jooq-meta-extensions dependency)

               org.jooq.meta.extensions.ddl.DDLDatabase
                   
           This value can be used to reverse-engineer schemas defined by JPA annotated entities (requires jooq-meta-extensions dependency)
                   
               org.jooq.meta.extensions.jpa.JPADatabase

           You can also provide your own org.jooq.meta.Database implementation
           here, if your database is currently not supported -->
      <name>org.jooq.meta.oracle.OracleDatabase</name>

      <!-- All elements that are generated from your schema (A Java regular expression.
           Use the pipe to separate several expressions) Watch out for
           case-sensitivity. Depending on your database, this might be
           important!
					 从模式生成的所有元素（Java正则表达式。使用管道分隔多个表达式）注意区分大小写。 根据您的数据库，这可能很重要！
           
           You can create case-insensitive regular expressions using this syntax: (?i:expr)
					 您可以使用以下语法创建不区分大小写的正则表达式：（？i：expr）
           
           Whitespace is ignored and comments are possible.
					 空格被忽略并且可以进行注释。
           -->
      <includes>.*</includes>

      <!-- All elements that are excluded from your schema (A Java regular expression.
           Use the pipe to separate several expressions). Excludes match before
           includes, i.e. excludes have a higher priority -->
      <excludes>
           UNUSED_TABLE                # This table (unqualified name) should not be generated 不应生成此表（非限定名称）
         | PREFIX_.*                   # Objects with a given prefix should not be generated 不应生成具有给定前缀的对象
         | SECRET_SCHEMA\.SECRET_TABLE # This table (qualified name) should not be generated 不应生成此表（限定名称）
         | SECRET_ROUTINE              # This routine (unqualified name) ... 这个例程（不合格的名字）
      </excludes>

      <!-- The schema that is used locally as a source for meta information.
           This could be your development schema or the production schema, etc
           This cannot be combined with the schemata element.

           If left empty, jOOQ will generate all available schemata. See the
           manual's next section to learn how to generate several schemata
					 如果留空，jOOQ将生成所有可用的模式。 请参阅手册的下一部分以了解如何生成多个模式
			-->
      <inputSchema>[your database schema / owner / name]</inputSchema>
    </database>

    <generate>
      <!-- Generation flags: See advanced configuration properties -->
    </generate>

    <target>
      <!-- The destination package of your generated classes (within the
           destination directory)
					 生成的类的目标包（在目标目录中）
           
           jOOQ may append the schema name to this package if generating multiple schemas,
           e.g. org.jooq.your.packagename.schema1
                org.jooq.your.packagename.schema2 -->
      <packageName>[org.jooq.your.packagename]</packageName>

      <!-- The destination directory of your generated classes -->
      <!-- 生成的类的目标目录 -->
      <directory>[/path/to/your/dir]</directory>
    </target>
  </generator>
</configuration>
```

There are also lots of advanced configuration parameters, which will be treated in the manual's section about advanced code generation features Note, you can find the official XSD file for a formal specification at:
http://www.jooq.org/xsd/jooq-codegen-3.11.0.xsd

还有许多高级配置参数，将在手册的高级代码生成功能部分中进行处理。注意，您可以在以下位置找到正式规范的官方XSD文件：
http://www.jooq.org/xsd/jooq-codegen-3.11.0.xsd

通过上面的配置, 就能使用了, 后续高级应用, 请看后面的文档

### 1.2 Advanced generator configuration(生成器高级配置)

In the previous section we have seen how jOOQ's source code generator is configured and run within a few steps. In this chapter we'll cover some advanced settings, individually.

在上一节中，我们已经了解了jOOQ的源代码生成器是如何配置并在几个步骤中运行的。 在本章中，我们将分别介绍一些高级设置。

关于插件更多使用文档, 请查看插件官方文档: https://www.jooq.org/doc/3.11/manual/code-generation/
