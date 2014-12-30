# uncommons-dbutils

A modernised fork of [Apache Commons DBUtils](http://commons.apache.org/proper/commons-dbutils/). The aim is to make a more flexible API and fix previous design mistakes, while keeping the original's small size and tight focus. 

## Requirements

Java 8

## Installation

Add the following dependency:

```xml
<dependency>
  <groupId>com.moandjiezana.dbutils</groupId>
  <artifactId>uncommons-dbutils</artifactId>
  <version>1.0.0-SNAPSHOT</version>
</dependency>
```

## Examples

```java
import static com.moandjiezana.uncommons.dbutils.ResultSetHandler.list;
import static com.moandjiezana.uncommons.dbutils.ResultSetHandler.map;
import static com.moandjiezana.uncommons.dbutils.ResultSetHandler.single;
import static com.moandjiezana.uncommons.dbutils.RowProcessor.firstColumn;

try (Connection connection = // obtain connection) {
  QueryRunner queryRunner = QueryRunner.create(connection);
  
  // Insert a record and retrieve its primary key
  Long newId = queryRunner.insert("INSERT INTO persons(name) VALUES(?)", single(firstColumn()), "John");
  
  // Update a record
  int affectedRows = queryRunner.execute("UPDATE persons SET name = ? WHERE id = ?", "Thabiso", newId);
  
  // Get a single column from a query that returns a single row
  String name = queryRunner.select("SELECT name FROM persons WHERE id = ?", single(firstColumn()), 1L);
  
  // Get a single column from a query that returns multiple rows
  List<String> names = queryRunner.select("SELECT name FROM persons", list(firstColumn()), 1L);
  
  // Create a Map containing all the single-row query's columns, i.e. { "id": 1, "name": "John" }
  Map<String, Object> person = queryRunner.select("SELECT * FROM persons WHERE id = ?", single(new MapRowProcessor(), 1L);
  
  // Create a custom Person object for each row returned by the query
  // Note that the ObjectRowProcessor, like most uncommons-dbutils classes, can be reused across queries, QueryRunners and threads
  ObjectRowProcessor<Person> personRowProcessor = new ObjectRowProcessor<Person>(Person.class, ObjectRowProcessor.matching());
  List<Person> persons = queryRunner.select("SELECT * FROM persons", list(personRowProcessor));
  
  // Create a Map of primary keys to Person objects, i.e. { 1: Person(...), 2: Person(...) }
  Map<Long, Person personsMap = queryRunner.select("SELECT * FROM persons", ResultSetHandler.map("id", Long.class, personRowProcessor));
}
```

## QueryRunner

A QueryRunner manages the database connection and executes queries. It can use either a `DataSource` or a `Connection`. When a `Connection` is used, it is up to you to close it.

## ResultSetHandler

A functional interface that manages the `ResultSet` and determines what it will be converted to. The built-in `ResultSetHander`s are:

* ResultSetHandler.single: Returns an object created by the passed in `RowProcessor`, or null if the `ResultSet` is empty
* ResultSetHandler.list: Returns a list populated by the `RowProcessor`, or an empty `List` if the `ResultSet` is empty
* ResultSetHandler.map: A shorthand way of creating a `MapResultSetHandler` that uses a single column as the entries' key
* ResultSetHandler.optional: Delegates processing to another `ResultSetHandler`, then wraps the returned value in an `Optional`
* ResultSetHandler.VOID: discards the `ResultSet`

## RowProcessor

Converts a `ResultSet` row into something else.
