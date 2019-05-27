---
layout: post
title:  "jooq insert 语句用法"
date:   2019-05-26 16:19:22 +0800
categories: jOOQ
tag: jOOQ
---

官方文档: <a href="https://www.jooq.org/doc/3.11/manual-single-page/#insert-statement" target="_blank">点击查看</a>

The INSERT statement is used to insert new records into a database table. 

INSERT语句用于将新记录插入数据库表。

The following sections describe the various operation modes of the jOOQ INSERT statement.

以下部分描述了jOOQ INSERT语句的各种操作模式。

### INSERT .. VALUES

**INSERT .. VALUES with a single row**

Records can either be supplied using a VALUES() constructor, or a SELECT statement. jOOQ supports both types of INSERT statements. An example of an INSERT statement using a VALUES() constructor is given here:

可以使用VALUES（）构造函数或SELECT语句提供记录。 jOOQ支持两种类型的INSERT语句。 这里给出了使用VALUES（）构造函数的INSERT语句的示例：

```SQL
INSERT INTO AUTHOR
       (ID, FIRST_NAME, LAST_NAME)
VALUES (100, 'Hermann', 'Hesse');
```

```Java
create.insertInto(AUTHOR,
        AUTHOR.ID, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
      .values(100, "Hermann", "Hesse")
      .execute();
```

请注意，对于最多22的显式度，VALUES（）构造函数提供了额外的类型安全性。 以下示例说明了这一点：

```Java
InsertValuesStep3<AuthorRecord, Integer, String, String> step =
  create.insertInto(AUTHOR, AUTHOR.ID, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME);
    step.values("A", "B", "C");
         // ^^^ Doesn't compile, the expected type is Integer
```

**INSERT .. VALUES with multiple rows**

The SQL standard specifies that multiple rows can be supplied to the VALUES() constructor in an INSERT statement. Here's an example of a multi-record INSERT

SQL标准指定可以在INSERT语句中向VALUES（）构造函数提供多行。 这是一个多记录INSERT的例子

```SQL
INSERT INTO AUTHOR
       (ID, FIRST_NAME, LAST_NAME)
VALUES (100, 'Hermann', 'Hesse'),
       (101, 'Alfred', 'Döblin');
```

```Java
create.insertInto(AUTHOR,
        AUTHOR.ID, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
      .values(100, "Hermann", "Hesse")
      .values(101, "Alfred", "Döblin")
      .execute()
```

jOOQ tries to stay close to actual SQL. In detail, however, Java's expressiveness is limited. That's why the values() clause is repeated for every record in multi-record inserts.

jOOQ试图保持接近实际的SQL。 然而，详细地说，Java的表现力是有限的。 这就是为多记录插入中的每个记录重复values（）子句的原因。

Some RDBMS do not support inserting several records in a single statement. In those cases, jOOQ emulates multi-record INSERTs using the following SQL:

某些RDBMS不支持在单个语句中插入多个记录。 在这些情况下，jOOQ使用以下SQL模拟多记录INSERT：

```SQL
INSERT INTO AUTHOR
    (ID, FIRST_NAME, LAST_NAME)
SELECT 100, 'Hermann', 'Hesse' FROM DUAL UNION ALL
SELECT 101, 'Alfred', 'Döblin' FROM DUAL;
```

```Java
create.insertInto(AUTHOR,
        AUTHOR.ID, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
      .values(100, "Hermann", "Hesse")
      .values(101, "Alfred", "Döblin")
      .execute();
```

**INSERT .. DEFAULT VALUES**

A lesser-known syntactic feature of SQL is the INSERT .. DEFAULT VALUES statement, where a single record is inserted, containing only DEFAULT values for every row. It is written as such:

SQL的一个鲜为人知的语法特性是INSERT .. DEFAULT VALUES语句，其中插入了一条记录，每行只包含DEFAULT值。 它是这样写的：

```SQL
INSERT INTO AUTHOR
DEFAULT VALUES;
```

```Java
create.insertInto(AUTHOR)
      .defaultValues()
      .execute();
```

This can make a lot of sense in situations where you want to "reserve" a row in the database for an subsequent UPDATE statement within the same transaction. Or if you just want to send an event containing trigger-generated default values, such as IDs or timestamps.

在您希望在同一事务中为后续UPDATE语句“保留”数据库中的行的情况下，这可能很有意义。 或者，如果您只想发送包含触发器生成的默认值的事件，例如ID或时间戳。

The DEFAULT VALUES clause is not supported in all databases, but jOOQ can emulate it using the equivalent statement:

并非所有数据库都支持DEFAULT VALUES子句，但jOOQ可以使用等效语句模拟它：

```SQL
INSERT INTO AUTHOR 
    (ID, FIRST_NAME, LAST_NAME, ...)
VALUES (
    DEFAULT, 
    DEFAULT, 
    DEFAULT, ...);
```

```Java
create.insertInto(
        AUTHOR, AUTHOR.ID, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME, ...)
      .values(
          defaultValue(AUTHOR.ID), 
          defaultValue(AUTHOR.FIRST_NAME), 
          defaultValue(AUTHOR.LAST_NAME), ...)
      .execute();
```

The DEFAULT keyword (or DSL#defaultValue() method) can also be used for individual columns only, although that will have the same effect as leaving the column away entirely.

DEFAULT关键字（或DSL＃defaultValue（）方法）也可以仅用于单个列，但这与完全保留列的效果相同。

**INSERT .. SET**

MySQL (and some other RDBMS) allow for using a non-SQL-standard, UPDATE-like syntax for INSERT statements. This is also supported in jOOQ (and emulated for all databases), should you prefer that syntax. The above INSERT statement can also be expressed as follows:

MySQL（以及其他一些RDBMS）允许对INSERT语句使用非SQL标准，类似UPDATE的语法。 如果你喜欢这种语法，jOOQ也支持这种情况（并为所有数据库模拟）。 上面的INSERT语句也可以表示如下：

```java
create.insertInto(AUTHOR)
      .set(AUTHOR.ID, 100)
      .set(AUTHOR.FIRST_NAME, "Hermann")
      .set(AUTHOR.LAST_NAME, "Hesse")
      .newRecord()
      .set(AUTHOR.ID, 101)
      .set(AUTHOR.FIRST_NAME, "Alfred")
      .set(AUTHOR.LAST_NAME, "Döblin")
      .execute();
```

As you can see, this syntax is a bit more verbose, but also more readable, as every field can be matched with its value. Internally, the two syntaxes are strictly equivalent.

正如您所看到的，这种语法更冗长，但也更具可读性，因为每个字段都可以与其值匹配。 在内部，这两种语法完全相同。

**INSERT .. SELECT**

In some occasions, you may prefer the INSERT SELECT syntax, for instance, when you copy records from one table to another:

在某些情况下，您可能更喜欢INSERT SELECT语法，例如，当您将记录从一个表复制到另一个表时：

```java
create.insertInto(AUTHOR_ARCHIVE)
      .select(selectFrom(AUTHOR).where(AUTHOR.DECEASED.isTrue()))
      .execute();
```

**INSERT .. ON DUPLICATE KEY**

The synthetic ON DUPLICATE KEY UPDATE clause

合成的ON DUPLICATE KEY UPDATE子句

The MySQL database supports a very convenient way to INSERT or UPDATE a record. This is a non-standard extension to the SQL syntax, which is supported by jOOQ and emulated in other RDBMS, where this is possible (i.e. if they support the SQL standard MERGE statement). Here is an example how to use the ON DUPLICATE KEY UPDATE clause:

MySQL数据库支持一种非常方便的INSERT或UPDATE记录方式。 这是SQL语法的非标准扩展，由jOOQ支持并在其他RDBMS中模拟，这是可能的（即，如果它们支持SQL标准MERGE语句）。 以下是如何使用ON DUPLICATE KEY UPDATE子句的示例：

```Java
// Add a new author called "Koontz" with ID 3.
// If that ID is already present, update the author's name
create.insertInto(AUTHOR, AUTHOR.ID, AUTHOR.LAST_NAME)
      .values(3, "Koontz")
      .onDuplicateKeyUpdate()
      .set(AUTHOR.LAST_NAME, "Koontz")
      .execute();
```

The synthetic ON DUPLICATE KEY IGNORE clause

The MySQL database also supports an INSERT IGNORE INTO clause. This is supported by jOOQ using the more convenient SQL syntax variant of ON DUPLICATE KEY IGNORE:

```Java
// Add a new author called "Koontz" with ID 3.
// If that ID is already present, ignore the INSERT statement
create.insertInto(AUTHOR, AUTHOR.ID, AUTHOR.LAST_NAME)
      .values(3, "Koontz")
      .onDuplicateKeyIgnore()
      .execute();
```

If the underlying database doesn't have any way to "ignore" failing INSERT statements, (e.g. MySQL via INSERT IGNORE), jOOQ can emulate the statement using a MERGE statement, or using INSERT .. SELECT WHERE NOT EXISTS:

如果底层数据库没有任何方法可以“忽略”INSERT语句失败，（例如MySQL通过INSERT IGNORE），jOOQ可以使用MERGE语句模拟语句，或使用INSERT .. SELECT WHERE NOT EXISTS：

**INSERT .. RETURNING**

The Postgres database has native support for an INSERT .. RETURNING clause. This is a very powerful concept that is emulated for all other dialects using JDBC's getGeneratedKeys() method. Take this example:

Postgres数据库本身支持INSERT .. RETURNING子句。 这是一个非常强大的概念，使用JDBC的getGeneratedKeys（）方法为所有其他方言模拟。 举个例子：

```java
// Add another author, with a generated ID
Record<?> record =
create.insertInto(AUTHOR, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
      .values("Charlotte", "Roche")
      .returning(AUTHOR.ID)
      .fetchOne();

System.out.println(record.getValue(AUTHOR.ID));

// For some RDBMS, this also works when inserting several values
// The following should return a 2x2 table
Result<?> result =
create.insertInto(AUTHOR, AUTHOR.FIRST_NAME, AUTHOR.LAST_NAME)
      .values("Johann Wolfgang", "von Goethe")
      .values("Friedrich", "Schiller")
      // You can request any field. Also trigger-generated values
      .returning(AUTHOR.ID, AUTHOR.CREATION_DATE)
      .fetch();
```

Some databases have poor support for returning generated keys after INSERTs. In those cases, jOOQ might need to issue another SELECT statement in order to fetch an @@identity value. Be aware, that this can lead to race-conditions in those databases that cannot properly return generated ID values. For more information, please consider the jOOQ Javadoc for the returning() clause.

一些数据库在INSERT之后对返回生成的键的支持很差。 在这些情况下，jOOQ可能需要发出另一个SELECT语句才能获取@@ identity值。 请注意，这可能会导致那些无法正确返回生成的ID值的数据库中的竞争条件。 有关更多信息，请考虑return（）子句的jOOQ Javadoc。
