---
layout: post
title:  "JOOQ入门"
date:   2019-05-17 22:19:22 +0800
categories: Jooq
tag: Jooq
---

本篇博客对应<a href="https://www.jooq.org/doc/latest/manual-pdf/jOOQ-manual-3.11.pdf" target="_blank">官方英文文档</a>, 只是原封不动的进行了翻译, 基本上是借助于Google翻译, 如果这篇博客被你不小心看见, 翻译不流畅的地方, 就直接看官方文档.

以下内容包含如何开始使用本手册和jOOQ的快速概述. 虽然后续章节包含大量参考信息，但本章仅包含基本内容.

### 3.1. 如何阅读本手册

本节帮助您在jOOQ的上下文中正确理解本手册.

**代码块**

以下是代码块:

```java
-- A SQL code block
SELECT 1 FROM DUAL
```

```java
// A Java code block
for(int i=0; i<10; i++);
```

```java
<!-- An XML code block -->
<hello what="world"></hello>
```

```java
# A config file code block
org.jooq.property=value
```

这些对于在代码中提供示例很有用。通常，使用jOOQ，将SQL代码与其对应的Java / jOOQ代码进行比较甚至更有用。完成此操作后，这些块并排排列，SQL通常位于左侧，而Java中的等效jOOQ DSL查询通常位于右侧：

<a href='/img/post/20190517/jooq_1.png' target="_blank"><img src='/img/post/20190517/jooq_1.png' /></a>

**代码块内容**

代码块的内容也遵循惯例。如果在任何给定代码块旁边没有提到任何其他内容，则可以假设以下内容：

```SQL
-- SQL假设
------------------

-- 如果未指定其他内容，则假定使用了Oracle语法
SELECT 1 FROM DUAL
```

```Java
// Java假设
// ----------------

// 每当您看到“standalone functions”时，假设它们是从org.jooq.impl.DSL导入的静态函数
// “DSL”是静态查询DSL的入口点
exists(); max(); min(); val(); inline(); // 对应DSL.exists（）;DSL.max（）;DSL.min（）; 等等...

// 每当您看到BOOK / Book，AUTHOR / Author和类似实体时，假设它们是从生成的模式中导入的（静态的）
BOOK.TITLE，AUTHOR.LAST_NAME // 对应于com.example.generated.Tables.BOOK.TITLE，com.example.generated.Tables.BOOK.TITLE
FK_BOOK_AUTHOR  // 对应于 to com.example.generated.Keys.FK_BOOK_AUTHOR

// 每当您在Java代码中看到“create”时，假设这是org.jooq.DSLContext的一个实例。
// 它被称为“create”的原因是，从DSL对象创建了一个jOOQ QueryPart
// 因此，“create”是非静态查询DSL的入口点

DSLContext create = DSL.using(connection, SQLDialect.ORACLE);
```

当然，您的命名可能有所不同。 例如，您可以将“create”实例命名为“db”。

**执行**

当您编写PL / SQL，T-SQL或其他一些过程SQL语言时，SQL语句总是立即在分号处执行。在jOOQ中不是这种情况，因为作为内部DSL，在调用fetch（）或execute（）之前，jOOQ永远无法确定您的语句是否完整。手册尝试尽可能彻底地应用fetch（）和execute（）。 如果没有，暗示：

<a href='/img/post/20190517/jooq_2.png' target="_blank"><img src='/img/post/20190517/jooq_2.png' /></a>

**Degree(arity)**

jOOQ records (and many other API elements) have a degree N between 1 and 22. The variable degree of an API element is denoted as [N], e.g. Row[N] or Record[N]. The term "degree" is preferred over arity, as "degree" is the term used in the SQL standard, whereas "arity" is used more often in mathematics and relational theory.

**Settings**

jOOQ允许使用org.jooq.conf.Settings覆盖运行时行为。如果未指定任何内容，则假定使用默认运行时设置。

**样本数据库**

jOOQ查询示例针对样本数据库运行。有关样本数据库的详细信息，请参阅本手册中使用的示例数据库的手册部分。

### 3.2. 本手册中使用的样本数据库

对于本手册中的示例，将始终引用相同的数据库。它基本上由使用Oracle方言创建的这些实体组成

```SQL
CREATE TABLE language (
 id NUMBER(7) NOT NULL PRIMARY KEY,
 cd CHAR(2) NOT NULL,
 description VARCHAR2(50)
);
CREATE TABLE author (
 id NUMBER(7) NOT NULL PRIMARY KEY,
 first_name VARCHAR2(50),
 last_name VARCHAR2(50) NOT NULL,
 date_of_birth DATE,
 year_of_birth NUMBER(7),
 distinguished NUMBER(1)
);
CREATE TABLE book (
 id NUMBER(7) NOT NULL PRIMARY KEY,
 author_id NUMBER(7) NOT NULL,
 title VARCHAR2(400) NOT NULL,
 published_in NUMBER(7) NOT NULL,
 language_id NUMBER(7) NOT NULL,

 CONSTRAINT fk_book_author FOREIGN KEY (author_id) REFERENCES author(id),
 CONSTRAINT fk_book_language FOREIGN KEY (language_id) REFERENCES language(id)
);
CREATE TABLE book_store (
 name VARCHAR2(400) NOT NULL UNIQUE
);
CREATE TABLE book_to_book_store (
 name VARCHAR2(400) NOT NULL,
 book_id INTEGER NOT NULL,
 stock INTEGER,

 PRIMARY KEY(name, book_id),
 CONSTRAINT fk_b2bs_book_store FOREIGN KEY (name) REFERENCES book_store (name) ON DELETE CASCADE,
 CONSTRAINT fk_b2bs_book FOREIGN KEY (book_id) REFERENCES book (id) ON DELETE CASCADE
);
```

为特定示例引入了更多实体，类型（例如UDT，ARRAY类型，ENUM类型等），存储过程和包

除上述内容外，您还可以假设以下示例数据：

```SQL
INSERT INTO language (id, cd, description) VALUES (1, 'en', 'English');
INSERT INTO language (id, cd, description) VALUES (2, 'de', 'Deutsch');
INSERT INTO language (id, cd, description) VALUES (3, 'fr', 'Français');
INSERT INTO language (id, cd, description) VALUES (4, 'pt', 'Português');
INSERT INTO author (id, first_name, last_name, date_of_birth , year_of_birth)
 VALUES (1 , 'George' , 'Orwell' , DATE '1903-06-26', 1903 );
INSERT INTO author (id, first_name, last_name, date_of_birth , year_of_birth)
 VALUES (2 , 'Paulo' , 'Coelho' , DATE '1947-08-24', 1947 );
INSERT INTO book (id, author_id, title , published_in, language_id)
 VALUES (1 , 1 , '1984' , 1948 , 1 );
INSERT INTO book (id, author_id, title , published_in, language_id)
 VALUES (2 , 1 , 'Animal Farm' , 1945 , 1 );
INSERT INTO book (id, author_id, title , published_in, language_id)
 VALUES (3 , 2 , 'O Alquimista', 1988 , 4 );
INSERT INTO book (id, author_id, title , published_in, language_id)
 VALUES (4 , 2 , 'Brida' , 1990 , 2 );
INSERT INTO book_store VALUES ('Orell Füssli');
INSERT INTO book_store VALUES ('Ex Libris');
INSERT INTO book_store VALUES ('Buchhandlung im Volkshaus');
INSERT INTO book_to_book_store VALUES ('Orell Füssli' , 1, 10);
INSERT INTO book_to_book_store VALUES ('Orell Füssli' , 2, 10);
INSERT INTO book_to_book_store VALUES ('Orell Füssli' , 3, 10);
INSERT INTO book_to_book_store VALUES ('Ex Libris' , 1, 1 );
INSERT INTO book_to_book_store VALUES ('Ex Libris' , 3, 2 );
INSERT INTO book_to_book_store VALUES ('Buchhandlung im Volkshaus', 3, 1 );
```

### 3.3. jOOQ的不同用例

jOOQ最初被创建为一个库，用于完全抽象JDBC和所有数据库交互。在预先存在的软件产品中经常遇到的各种最佳实践应用于该库。这包括：

- 通过生成的模式，表，列，记录，过程，类型，dao，pojo伪像引用的Typesafe数据库对象（请参阅有关代码生成的章节）

- 通过完整的查询DSL API对Typesafe SQL构建/ SQL构建建模SQL作为Java中的域特定语言（请参阅有关查询DSL API的章节）

- 通过改进的API获取结果，方便查询执行（参见有关各种数据获取类型的章节）

-  SQL方言抽象和SQL子句仿真，以提高跨数据库兼容性并在更简单的数据库中启用缺少的功能（请参阅有关SQL方言的章节）

- 使用jOOQ作为开发过程不可或缺的一部分进行SQL日志记录和调试（请参阅有关日志记录的章节）

实际上，jOOQ最初设计用于替换任何其他数据库抽象框架，而不是处理连接池（以及更复杂的事务管理）的框架。

**以您喜欢的方式使用jOOQ**

但开源是社区驱动的。 社区已经展示了使用与原始意图背道而驰的jOOQ的各种方式。 遇到的一些用例是：

- 使用Hibernate进行70％的查询（即CRUD），使用jOOQ进行剩余的30％，其中确实需要SQL
- 使用jOOQ进行SQL构建，使用JDBC进行SQL执行
- 使用jOOQ进行SQL构建，使用Spring Data进行SQL执行
- 使用不带源代码生成器的jOOQ构建动态SQL执行框架的基础。

以下部分介绍了在您的应用程序中使用jOOQ的各种用例。

#### 3.3.1. jOOQ作为SQL构建器

这是所有用例中最简单的，允许为任何数据库构建有效的SQL。 在这个用例中，您将不使用jOOQ的代码生成器，甚至可能不使用jOOQ的查询执行工具。 相反，您将使用jOOQ的查询DSL API将字符串，文字和其他用户定义的对象包装到面向对象，类型安全的AST中，为您的SQL语句建模。

```java
// 从jOOQ查询中获取SQL字符串，以便使用其他工具手动执行它。
// 为简单起见，我们在这里使用API来构造不区分大小写的对象引用。
String sql = create.select(field("BOOK.TITLE"), field("AUTHOR.FIRST_NAME"), field("AUTHOR.LAST_NAME"))
									 .from(table("BOOK"))
									 .join(table("AUTHOR"))
									 .on(field("BOOK.AUTHOR_ID").eq(field("AUTHOR.ID")))
									 .where(field("BOOK.PUBLISHED_IN").eq(1948))
									 .getSQL();
```

然后可以使用Spring的JdbcTemplate，使用Apache DbUtils和许多其他工具直接使用JDBC执行使用jOOQ查询DSL构建的SQL字符串（请注意，由于jOOQ默认使用PreparedStatement，因此将生成“1948”的绑定变量。 更多关于绑定变量的信息）。

如果您只想将jOOQ用作SQL构建器，那么您将对以下手册部分感兴趣:

- SQL构建：本节包含有关使用jOOQ API创建SQL语句的大量信息
- 普通SQL：本节包含的信息特别适用于那些想要将表表达式，列表达式等作为纯SQL提供给jOOQ而不是通过生成的伪像提供的信息。

#### 3.3.2。jOOQ作为具有代码生成的SQL构建器

除了使用jOOQ作为独立的SQL构建器之外，您还可以使用jOOQ的代码生成功能，以便使用Java编译器针对实际数据库编译SQL语句
架构。这为使用查询DSL和自定义字符串和文字简单地构建SQL增加了很多力量和表现力，因为您可以确保所有数据库工件实际存在于
数据库，他们的类型是正确的。这里给出一个例子：

```Java
// 从jOOQ查询中获取SQL字符串，以便使用其他工具手动执行它。
String sql = create.select(BOOK.TITLE, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
									 .from(BOOK)
									 .join(AUTHOR)
									 .on(BOOK.AUTHOR_ID.eq(AUTHOR.ID))
									 .where(BOOK.PUBLISHED_IN.eq(1948))
									 .getSQL();
```

您可以直接使用JDBC，使用Spring的JdbcTemplate，使用Apache DbUtils和许多其他工具来执行您可以生成的SQL字符串。

如果您希望仅将jOOQ用作具有代码生成的SQL构建器，则您将对以下手册部分感兴趣：

-  SQL构建：本节包含有关使用jOOQ API创建SQL语句的大量信息
- 代码生成：本节包含针对开发人员数据库运行jOOQ代码生成器的必要信息

#### 3.3.3. jOOQ作为SQL执行器

您可以直接使用jOOQ来执行jOOQ生成的SQL语句，而不是前面章节中提到的任何工具。 当您可以重新使用生成的类中的信息来获取记录和自定义数据类型时，这将为之前讨论的类型安全SQL构造API添加许多便利。这里给出一个例子：

```java
// 直接用jOOQ执行SQL语句
Result<Record3<String, String, String>> result =
create.select(BOOK.TITLE, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
			 .from(BOOK)
			 .join(AUTHOR)
			 .on(BOOK.AUTHOR_ID.eq(AUTHOR.ID))
			 .where(BOOK.PUBLISHED_IN.eq(1948))
			 .fetch();
```

通过让jOOQ执行您的SQL，jOOQ查询DSL成为真正的嵌入式SQL。

不过，jOOQ并不止于此！ 您可以使用jOOQ执行任何SQL。 换句话说，您可以使用任何其他SQL构建工具并使用jOOQ运行SQL语句。 这里给出一个例子：

```java
// 使用您喜欢的工具来构造SQL字符串:
String sql = "SELECT title, first_name, last_name FROM book JOIN author ON book.author_id = author.id " + "WHERE book.published_in = 1984";
// 使用jOOQ获取结果
Result<Record> result = create.fetch(sql);
// 或者使用JDBC执行该SQL，使用jOOQ获取ResultSet：
ResultSet rs = connection.createStatement().executeQuery(sql);
Result<Record> result = create.fetch(rs);
```

如果您希望将jOOQ用作具有（或不生成）代码生成的SQL执行器，则您将对以下手册部分感兴趣：

- SQL构建：本节包含有关使用jOOQ API创建SQL语句的大量信息
- 代码生成：本节包含针对开发人员数据库运行jOOQ代码生成器的必要信息
- SQL执行：本节包含有关使用jOOQ API执行SQL语句的大量信息
- 获取：本节包含一些有关使用jOOQ获取数据的各种方法的有用信息

#### 3.3.4. jOOQ for CRUD

除了jOOQ用于查询构建的流畅API之外，jOOQ还可以帮助您执行日常CRUD操作。 这里给出一个例子：

```java
// Fetch an author
AuthorRecord author : create.fetchOne(AUTHOR, AUTHOR.ID.eq(1));
// Create a new author, if it doesn't exist yet
if (author == null) {
 author = create.newRecord(AUTHOR);
 author.setId(1);
 author.setFirstName("Dan");
 author.setLastName("Brown");
}
// Mark the author as a "distinguished" author and store it
author.setDistinguished(1);
// Executes an update on existing authors, or insert on new ones
author.store();
```

如果您希望使用jOOQ的所有功能，您将对本手册的以下部分感兴趣（包括所有小节）：

- SQL构建：本节包含有关使用jOOQ API创建SQL语句的大量信息
- 代码生成：本节包含针对开发人员数据库运行jOOQ代码生成器的必要信息
- SQL执行：本节包含有关使用执行SQL语句的大量信息jOOQ API

#### 3.3.5. jOOQ for PROs

jOOQ不仅仅是一个帮助您根据生成的可编译模式构建和执行SQL的库。 jOOQ附带了很多工具。 以下是jOOQ附带的一些最重要的工具：

- jOOQ的执行监听器：jOOQ允许您将自定义执行监听器挂钩到jOOQ的SQL语句执行生命周期中，以集中协调对正在执行的SQL执行的任何操作。用于记录，标识生成，SQL跟踪，性能测量等。
- 日志记录：jOOQ内置一个标准的DEBUG记录器，用于记录和跟踪所有已执行的SQL语句和获取的结果集
- 存储过程：jOOQ支持您喜欢的数据库的存储过程和功能。 生成所有例程和用户定义类型，并且可以作为函数引用包含在jOOQ的SQL构建API中。
- 批量执行：执行大量SQL语句时，批量执行很重要。 与JDBC相比，jOOQ简化了这些操作
- 导出和导入：jOOQ附带API，可以轻松导出/导入各种格式的数据

如果您是自己喜欢的功能丰富的数据库的高级用户，jOOQ将帮助您访问所有数据库的供应商特定功能，例如OLAP功能，存储过程，用户定义类型，特定于供应商的SQL，函数，本手册中给出了一些例子。

### 3.4. 教程

没时间阅读完整的手册？ 以下是一些教程，可以让您尽快进入jOOQ最基本的部分。

#### 3.4.1. jOQQ in 7 easy steps

本手册部分面向新用户，帮助他们快速获得jOOQ正在运行的应用程序。

** 3.4.1.1. Step 1: 准备 **

如果您还没有下载它，请下载jOOQ：http://www.jooq.org/download 或者，您可以创建Maven依赖项来下载jOOQ artefacts：

使用 开源版

```java
<dependency>
		<groupId>org.jooq</groupId>
		<artifactId>jooq</artifactId>
		<version>3.11.11</version>
</dependency>
<dependency>
		<groupId>org.jooq</groupId>
		<artifactId>jooq-meta</artifactId>
		<version>3.11.11</version>
</dependency>
<dependency>
		<groupId>org.jooq</groupId>
		<artifactId>jooq-codegen</artifactId>
		<version>3.11.11</version>
</dependency>
```

使用商业版(Java 8+)

```java
<!-- Note: These aren't hosted on Maven Central. Import them manually from your distribution -->
<!-- 这些不在Maven Central上托管。 从您的分发中手动导入它们 -->
<dependency>
		<groupId>org.jooq.pro</groupId>
		<artifactId>jooq</artifactId>
		<version>3.11.11</version>
</dependency>
<dependency>
		<groupId>org.jooq.pro</groupId>
		<artifactId>jooq-meta</artifactId>
		<version>3.11.11</version>
</dependency>
<dependency>
		<groupId>org.jooq.pro</groupId>
		<artifactId>jooq-codegen</artifactId>
		<version>3.11.11</version>
</dependency>
```

使用商业版(Java 6+)

```java
<!-- Note: These aren't hosted on Maven Central. Import them manually from your distribution -->
<dependency>
		<groupId>org.jooq.pro-java-6</groupId>
		<artifactId>jooq</artifactId>
		<version>3.11.11</version>
</dependency>
<dependency>
		<groupId>org.jooq.pro-java-6</groupId>
		<artifactId>jooq-meta</artifactId>
		<version>3.11.11</version>
</dependency>
<dependency>
		<groupId>org.jooq.pro-java-6</groupId>
		<artifactId>jooq-codegen</artifactId>
		<version>3.11.11</version>
</dependency>
```

使用商业版（免费试用版）

```java
<!-- Note: These aren't hosted on Maven Central. Import them manually from your distribution -->
<dependency>
		<groupId>org.jooq.trial</groupId>
		<artifactId>jooq</artifactId>
		<version>3.11.11</version>
</dependency>
<dependency>
		<groupId>org.jooq.trial</groupId>
		<artifactId>jooq-meta</artifactId>
		<version>3.11.11</version>
</dependency>
<dependency>
		<groupId>org.jooq.trial</groupId>
		<artifactId>jooq-codegen</artifactId>
		<version>3.11.11</version>
</dependency>
```

请注意，Maven Central仅提供jOOQ开源版。如果您使用的是jOOQ专业版或jOOQ企业版，则必须在本地Nexus或本地Maven缓存中手动安装jOOQ。 有关更多信息，请参阅许可页面。

请参阅手册中有关代码生成配置的部分，以了解如何在Maven中使用jOOQ的代码生成器。

对于这个例子，我们将使用MySQL。 如果您尚未下载MySQL Connector / J，请在此处下载：http://dev.mysql.com/downloads/connector/j/

如果您尚未启动并运行MySQL实例，请立即获取XAMPP！ XAMPP是Apache，MySQL，PHP和Perl的简单安装包

** 3.4.1.2. Step 2: Your database **

我们将创建一个名为“library”的数据库和一个相应的“author”表。 通过命令行客户端连接到MySQL并键入以下内容：

```SQL
CREATE DATABASE `library`;
USE `library`;
CREATE TABLE `author` (
 `id` int NOT NULL,
 `first_name` varchar(255) DEFAULT NULL,
 `last_name` varchar(255) DEFAULT NULL,
 PRIMARY KEY (`id`)
);
```

** 3.4.1.3. Step 3: Code generation

在这一步中，我们将使用jOOQ的命令行工具生成映射到我们刚刚创建的Author表的类。有关如何设置jOOQ代码生成器的更多详细信息，请访问：jOOQ manual pages about setting up the code generator

关于设置代码生成器的jOOQ手册页生成模式的最简单方法是将jOOQ jar文件（应该有3个）和MySQL Connector jar文件复制到临时目录。 然后，创建一个如下所示的library.xml：

```XML
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<configuration xmlns="http://www.jooq.org/xsd/jooq-codegen-3.11.0.xsd">
	<!-- Configure the database connection here -->
	<jdbc>
			<driver>com.mysql.jdbc.Driver</driver>
 			<url>jdbc:mysql://localhost:3306/library</url>
 			<user>root</user>
 			<password></password>
 	</jdbc>
 	<generator>
 <!-- The default code generator. You can override this one, to generate your own code style.
			默认代码生成器。 您可以覆盖此项，以生成您自己的代码样式。
 			Supported generators:
 			- org.jooq.codegen.JavaGenerator
 			- org.jooq.codegen.ScalaGenerator
 			Defaults to org.jooq.codegen.JavaGenerator -->
	<name>org.jooq.codegen.JavaGenerator</name>
 	<database>
 	    <!-- The database type. The format here is: org.jooq.meta.[database].[database]Database -->
			<name>org.jooq.meta.mysql.MySQLDatabase</name>
 			<!-- The database schema (or in the absence of schema support, in your RDBMS this can be the owner, user, database name) to be generated -->
		  <inputSchema>library</inputSchema>
 			<!-- All elements that are generated from your schema (A Java regular expression. Use the pipe to separate several expressions) Watch out for case-sensitivity. Depending on your database, this might be important! -->
		  <includes>.*</includes>
 			<!-- All elements that are excluded from your schema (A Java regular expression. Use the pipe to separate several expressions). Excludes match before includes, i.e. excludes have a higher priority -->
		  <excludes></excludes>
	 </database>
   <target>
       <!-- The destination package of your generated classes (within the destination directory) -->
 			<packageName>test.generated</packageName>
 			<!-- The destination directory of your generated classes. Using Maven directory layout here -->
 			<directory>C:/workspace/MySQLTest/src/main/java</directory>
 		</target>
   </generator>
</configuration>
```

用具有相应权限的任何用户替换用户名以查询数据库元数据。 您还需要查看其他值并根据需要进行替换。 以下是两个有趣的属性：

generator.target.package  - 将此属性设置为要为生成的类创建的父包。
test.generated的设置将导致test.generated.Author和test.generated.AuthorRecord被创建

generator.target.directory  - 要输出的目录。

在临时目录中有JAR文件和library.xml后，在Windows机器上键入：

```shell
java -classpath jooq-3.11.11.jar;jooq-meta-3.11.11.jar;jooq-codegen-3.11.11.jar;mysql-connector-java-5.1.18-bin.jar; org.jooq.codegen.GenerationTool library.xml
```

or type this on a UNIX / Linux / Mac system (colons instead of semi-colons):

```shell
java -classpath jooq-3.11.11.jar:jooq-meta-3.11.11.jar:jooq-codegen-3.11.11.jar:mysql-connector-java-5.1.18-bin.jar:. org.jooq.codegen.GenerationTool library.xml
```

注意：jOOQ将尝试从类路径加载library.xml。 这也是类路径上有一个尾随句点（.）的原因。 如果在类路径上找不到该文件，jOOQ将从当前工作目录查看文件系统。

用您的实际文件名替换文件名。 在此示例中，正在使用jOOQ 3.11.11。 如果一切正常，您应该在控制台输出中看到：

** 3.4.1.4. Step 4: Connect to your database **

让我们在包含生成的类的项目中编写一个主类：

```java
public class MainOne {
    public static void main(String[] args) {
        String userName = "root";
        String password = "";
        String url = "jdbc:mysql://localhost:3306/library";

        // Connection is the only JDBC resource that we need
        // Connection是我们需要的唯一JDBC资源
        // PreparedStatement and ResultSet are handled by jOOQ, internally
        // PreparedStatement和ResultSet由内部的jOOQ处理
        try (Connection conn = DriverManager.getConnection(url, userName, password)) {
            // ...
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

这是用于建立MySQL连接的非常标准的代码。

** 3.4.1.5. Step 5: Querying **

让我们添加一个用jOOQ的查询DSL构建的简单查询：

```java
DSLContext create = DSL.using(conn, SQLDialect.MYSQL);
Result<Record> result = create.select().from(AUTHOR).fetch();
```

首先获取DSLContext的一个实例，这样我们就可以编写一个简单的SELECT查询。
我们将MySQL连接的实例传递给DSL。 请注意，DSLContext不会关闭连接。 我们必须自己做。
然后我们使用jOOQ的查询DSL来返回Result的实例。 我们将在下一步中使用此结果。

** 3.4.1.6. Step 6: Iterating **

在我们检索结果的行之后，让我们迭代结果并打印出数据：

完整的程序现在应该如下所示：

```java
public class MainOne {
    public static void main(String[] args) {
        String userName = "root";
        String password = "jkxsl12369";
        String url = "jdbc:mysql://localhost:3306/library";

        // Connection is the only JDBC resource that we need
        // Connection是我们需要的唯一JDBC资源
        // PreparedStatement and ResultSet are handled by jOOQ, internally
        // PreparedStatement和ResultSet由内部的jOOQ处理
        try (Connection conn = DriverManager.getConnection(url, userName, password)) {
            // ...
            DSLContext create = DSL.using(conn, SQLDialect.MYSQL);
            Result<Record> result = create.select().from(AUTHOR).fetch();

            for (Record r : result) {
                Integer id = r.getValue(AUTHOR.ID);
                String firstName = r.get(AUTHOR.FIRST_NAME);
                String lastName = r.getValue(AUTHOR.LAST_NAME);
                System.out.println("ID: " + id + " first name: " + firstName + " last name: " + lastName);
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

** 3.4.1.7. Step 7: Explore! **

jOOQ已经发展成为一个综合的SQL库。 有关更多信息，请考虑以下文档：

http://www.jooq.org/learn
... explore the Javadoc:
http://www.jooq.org/javadoc/latest/
... or join the news group:
https://groups.google.com/forum/#!forum/jooq-user
This tutorial is the courtesy of Ikai Lan. See the original source here:
http://ikaisays.com/2011/11/01/getting-started-with-jooq-a-tutorial/

#### 3.4.2. Using jOOQ in modern IDEs

Feel free to contribute a tutorial!

#### 3.4.3. Using jOOQ with Spring and Apache DBCP

jOOQ和Spring易于集成。 在这个例子中，我们将整合：

-  Apache DBCP（但您也可以使用其他一些连接池，如BoneCP，C3P0，HikariCP和其他各种连接池）。
-  Spring TX作为事务管理库。
- 作为SQL构建和执行库的jOOQ。

在复制手动示例之前，请考虑以下进一步的资源：

- 完整的示例也可以从GitHub下载。<a href="https://github.com/jOOQ/jOOQ/tree/master/jOOQ-examples/jOOQ-spring-example" target="_blank">jOOQ-spring-example</a>
- 可以从GitHub下载使用Spring和Guice进行事务管理的另一个示例。<a href="https://github.com/jOOQ/jOOQ/tree/master/jOOQ-examples/jOOQ-spring-guice-example" target="_blank">jOOQ-spring-guice-example</a>
- Petri Kainulainen的另一篇优秀教程可以在这里找到。<a href="http://www.petrikainulainen.net/programming/jooq/using-jooq-with-spring-configuration/" target="_blank">点击下载</a>

添加所需的Maven依赖项

对于此示例，我们将创建以下Maven依赖项

```java
<!-- Use this or the latest Spring RELEASE version -->
<properties>
	<org.springframework.version>3.2.3.RELEASE</org.springframework.version>
</properties>
<dependencies>
	<!-- Database access -->
	<dependency>
		<!-- Use org.jooq for the Open Source Edition
		org.jooq.pro for commercial editions,
		org.jooq.pro-java-6 for commercial editions with Java 6 support,
		org.jooq.trial for the free trial edition

		Note: Only the Open Source Edition is hosted on Maven Central.
		Import the others manually from your distribution -->
		<groupId>org.jooq</groupId>
		<artifactId>jooq</artifactId>
		<version>3.11.11</version>
	</dependency>
	<dependency>
		<groupId>org.apache.commons</groupId>
		<artifactId>commons-dbcp2</artifactId>
		<version>2.0</version>
	</dependency>
	<dependency>
		<groupId>com.h2database</groupId>
		<artifactId>h2</artifactId>
		<version>1.3.168</version>
	</dependency>
	<!-- Logging -->
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>1.2.16</version>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-log4j12</artifactId>
		<version>1.7.5</version>
	</dependency>
	<!-- Spring (transitive dependencies are not listed explicitly) -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>${org.springframework.version}</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-jdbc</artifactId>
		<version>${org.springframework.version}</version>
	</dependency>
	<!-- Testing -->
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.11</version>
		<type>jar</type>
		<scope>test</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-test</artifactId>
		<version>${org.springframework.version}</version>
		<scope>test</scope>
	</dependency>
</dependencies>
```

请注意，Maven Central仅提供jOOQ开源版。
如果您使用的是jOOQ专业版或jOOQ企业版，则必须在本地Nexus或本地Maven缓存中手动安装jOOQ。

有关更多信息，请参阅许可页面。

创建一个最小的Spring配置文件

使用Spring Beans配置上述依赖项, 建议采用Spring-Boot来进行构建.

** 使用以上配置运行查询：**

使用上面的配置，您应该可以很快地运行查询。

例如，在集成测试中，您可以使用Spring来运行JUnit：

```java
@Autowired
DSLContext create;

@Test
public void testJoin() {
		// All of these tables were generated by jOOQ's Maven plugin
		Book b = BOOK.as("b");
		Author a = AUTHOR.as("a");
		BookStore s = BOOK_STORE.as("s");
		BookToBookStore t = BOOK_TO_BOOK_STORE.as("t");

		String sql = create.select(a.FIRST_NAME, a.LAST_NAME, DSL.countDistinct(s.NAME))
						.from(a)
						.join(b).on(b.AUTHOR_ID.eq(a.ID))
						.join(t).on(t.BOOK_ID.eq(b.ID))
						.join(s).on(t.BOOK_STORE_ID.eq(s.ID))
						.groupBy(a.FIRST_NAME, a.LAST_NAME)
						.orderBy(DSL.countDistinct(s.NAME).desc())
						.getSQL();

		System.out.println("查询SQL=" + sql);

		Result<Record3<String, String, Integer>> result = create.select(a.FIRST_NAME, a.LAST_NAME, DSL.countDistinct(s.NAME))
						.from(a)
						.join(b).on(b.AUTHOR_ID.eq(a.ID))
						.join(t).on(t.BOOK_ID.eq(b.ID))
						.join(s).on(t.BOOK_STORE_ID.eq(s.ID))
						.groupBy(a.FIRST_NAME, a.LAST_NAME)
						.orderBy(DSL.countDistinct(s.NAME).desc())
						.fetch();

		assertEquals(1, result.size());
		assertEquals("su", result.getValue(0, a.FIRST_NAME));
		assertEquals("lin", result.getValue(0, a.LAST_NAME));
		assertEquals(Integer.valueOf(1), result.getValue(0, DSL.countDistinct(s.NAME)));
}
```

** 在显式事务中运行查询：**

以下示例显示了如何使用Spring的TransactionManager显式处理事务：

```java
public void testExplicitTransactions() {
		boolean rollback = false;

		TransactionStatus tx = txMgr.getTransaction(new DefaultTransactionDefinition());
		try {
				for (int i = 0; i < 2; i++) {
						dsl.insertInto(BOOK)
										.set(BOOK.ID, 4+i)
										.set(BOOK.AUTHOR_ID, 1)
										.set(BOOK.TITLE, "深入浅出Python机器学习" + i + 1)
										.execute();
				}
				// 在不检查任何条件的情况下使断言失败.
//            Assert.fail();
		} catch (DataAccessException e) {
//            txMgr.rollback(tx);
				rollback = true;
		}
		txMgr.commit(tx);
//        assertEquals(4, dsl.fetchCount(BOOK));
//        assertTrue(rollback);
}
```

** Run queries using declarative transactions **

Spring-TX具有使用@Transactional注释以声明方式处理事务的非常强大的方法。

我们在之前的Spring配置中定义的BookService可以在这里看到：

```java
public interface BookService {
  /**
	 * Create a new book.
	 * <p>
	 * The implementation of this method has a bug, which causes this method to
	 * fail and roll back the transaction.
  **/
	@Transactional
 	void create(int id, int authorId, String title);
	}
```

以下是我们与之互动的方式：

```java
@Test
public void testDeclarativeTransactions() {
	boolean rollback = false;
	try {

		// The service has a "bug", resulting in a constraint violation exception
		books.create(5, 1, "Book 5");
		Assert.fail();
	}
	catch (DataAccessException ignore) {
		rollback = true;
	}
	assertEquals(4, dsl.fetchCount(BOOK));
	assertTrue(rollback);
}
```

使用jOOQ的事务API运行查询jOOQ有自己的编程事务API，可以通过实现jOOQ org.jooq与Spring事务一起使用。

TransactionProvider SPI并将其传递给您的jOOQ配置。

有关此事务API的更多详细信息，请参阅有关事务管理的手册部分。

你可以从GitHub下载它自己尝试上面的例子。<a href="https://github.com/jOOQ/jOOQ/tree/master/jOOQ-examples/jOOQ-spring-example" target="_blank">点击下载</a>

**3.4.6. A simple web application with jOOQ**

Feel free to contribute a tutorial!

#### 3.5. jOOQ and Java 8

Java 8引入了一系列增强功能，其中包括lambda表达式和新的java.util.stream.Stream. 这些新结构与jOOQ的流畅API非常吻合，如以下示例所示：

** jOOQ和lambda表达式 **

jOOQ的RecordMapper API完全支持Java-8，这基本上意味着它是一种SAM（单一抽象方法）类型，可以使用lambda表达式进行实例化。

考虑这个例子：

```java
try (Connection c = getConnection()) {
	String sql = "select schema_name, is_default " +
		"from information_schema.schemata " +
		"order by schema_name";
	DSL.using(c)
		.fetch(sql)
		// We can use lambda expressions to map jOOQ Records
		.map(rs -> new Schema(
					rs.getValue("SCHEMA_NAME", String.class),
					rs.getValue("IS_DEFAULT", boolean.class)
					))
		// ... and then profit from the new Collection methods
		.forEach(System.out::println);
}
```

上面的示例显示了jOOQ的Result.map()方法如何接收lambda表达式，该表达式实现RecordMapper以从jOOQ Records映射到自定义类型。

** jOOQ和the Streams API **

jOOQ的Result类型扩展了java.util.List，它打开了对Java 8中各种新Java特性的访问。以下示例显示了转换包含INFORMATION_SCHEMA元数据的jOOQ Result以生成DDL语句是多么容易：

```java
	DSL.using(c)
  .select(
		COLUMNS.TABLE_NAME,
		COLUMNS.COLUMN_NAME,
		COLUMNS.TYPE_NAME
		)
	.from(COLUMNS)
  .orderBy(
		COLUMNS.TABLE_CATALOG,
		COLUMNS.TABLE_SCHEMA,
		COLUMNS.TABLE_NAME,
		COLUMNS.ORDINAL_POSITION
		)
	.fetch() // jOOQ ends here
  .stream() // JDK 8 Streams start here
	.collect(groupingBy(
				r -> r.getValue(COLUMNS.TABLE_NAME),
				LinkedHashMap::new,
				mapping(
					r -> new Column(
						r.getValue(COLUMNS.COLUMN_NAME),
						r.getValue(COLUMNS.TYPE_NAME)
						),
					toList()
					)
				))
	.forEach(
			(table, columns) -> {
			// Just emit a CREATE TABLE statement
			System.out.println(
					"CREATE TABLE " + table + " (");
			// Map each "Column" type into a String
			// containing the column specification,
			// and join them using comma and
			// newline. Done!
			System.out.println(
					columns.stream()
					.map(col -> " " + col.name +
						" " + col.type)
					.collect(Collectors.joining(",\n"))
					);
			System.out.println(");");
			}
	);
```

上面的例子在这篇博客文章中有更深入的解释：http://blog.jooq.org/2014/04/11/java-8-friday-no-more-need-for-orms/

有关Java 8的更多信息，请考虑以下资源：
- 我们的Java 8 Friday博客系列, https://blog.jooq.org/tag/java-8/
- Baeldung.com的人们收集的Java 8资源, https://www.baeldung.com/java-streams

#### 3.6. jOOQ and JavaFX

待补充

#### 3.7. jOOQ and Nashorn

待补充

#### 3.8. jOOQ and Scala

待补充

#### 3.9. jOOQ and Groovy

待补充

#### 3.10. jOOQ and Kotlin

待补充

#### 3.11. jOOQ and NoSQL

待补充

#### 3.12. jOOQ and JPA

仅仅因为你正在使用jOOQ并不意味着你必须将它用于一切! 

#### 3.13. Dependencies

依赖性是现代软件中的一大麻烦。
许多库依赖于其他非JDK库部件，这些部件具有不同的，不兼容的版本，可能会在运行时环境中造成麻烦。
jOOQ对任何第三方库都没有外部依赖关系。
但是，上述规则有一些例外：

- 日志API被引用为“可选依赖项”。 jOOQ尝试在类路径上找到slf4j或log4j。 如果失败，它将使用java.util.logging.Logger

- 使用反射加载用于数组创建的Oracle ojdbc类型。 这同样适用于Postgres PG 类型。

- 具有兼容许可证的小型库已合并到jOOQ中。 这些包括jOOR，jOOU，OpenCSV的部分，简单的json，commons-lang的部分

- 如果激活相关的代码生成标志，则需要javax.persistence和javax.validation

#### 3.14. Build your own

待补充

#### 3.15. jOOQ和向后兼容性

待补充















