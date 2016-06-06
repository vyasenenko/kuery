# Kuery - easy SQL with Kotlin

The library provides a way to work with a subset of SQL in Kotlin programming language. SQLite is the only supported dialect right now.

**WARNING: the library is in developement stage. The APIs are unstable and might be changed in the future.**

## Data Definition Language

The first step is to define the tables by inheriting from the **Table** class.

```kotlin
object EmployeeTable : Table("employees") {
	val id = Column("id")
	val name = Column("name")
	val organizationId = Column("organization_id")
}

object OrganizationTable : Table("organizations") {
	val id = Column("id")
	val name = Column("name")
}
```

**Statement** class is a starting point for writing any code. Resulting object can be turned into SQL by calling **.toString()** method. 

```kotlin
Statement.on(EmployeeTable).drop().toString() // DROP TABLE "organizations"
```

### CREATE TABLE statement

```kotlin
// CREATE TABLE "employees" ...
Statement.on(EmployeeTable)
		.create { e ->
			e.id.integer().primaryKey(autoIncrement = true).notNull() +
			e.name.text().unique().notNull() +
			e.organizationId.integer()
		}

// CREATE TABLE "organizations" ...
Statement.on(OrganizationTable)
		.create { e ->
			e.id.integer().primaryKey(autoIncrement = true).notNull() +
			e.name.text().unique().notNull()
		}
```

### DROP TABLE statement

```kotlin
// DROP TABLE "employees"
Statement.on(EmployeeTable).drop()
```

## Data Manipulation Language

Data manipulation is the most powerfull and complex part of SQL. The library supports insert, select, update and delete statements.

### INSERT statement

```kotlin
// INSERT INTO "organizations"("name", "organization_id") VALUES('?', '?')
Statement.on(EmployeeTable)
		.insert { e -> e.name("?") + e.organizationId("?") } // invoke syntax is used to set a value for the column
```

### SELECT statement

The library provides the following operators to compose queries (infix functions):
* and
* or
* not
* eq (equals)
* ne (not equals)
* lt (less than)
* lte (less than or equal to)
* gt (greater than)
* gte (greater than or equal to)

```kotlin
// SELECT "id", "name" FROM "organizations" WHERE ...
Statement.on(EmployeeTable)
		.where { e -> (e.organizationId ne null) and (e.name eq "''") }
		.orderBy { e -> e.name + e.id.desc }
		.limit { 10 }
		.offset { 10 }
		.select { e -> e.id + e.name }
```

Joining tables is also supported

```kotlin
// SELECT ... FROM "organizations" JOIN "employees" ON ...
Statement.on(OrganizationTable)
		.join(EmployeeTable).on { o, e -> o.id eq e.organizationId }
		.select { o, e -> o.name + e.name }
```

### UPDATE statement

```kotlin
// UPDATE "organizations" SET ... WHERE "id" = 1
Statement.on(OrganizationTable)
		.where { o -> o.id eq 1 }
		.update { o -> o.name("?") }
```

### DELETE statement
```kotlin
// DELETE FROM "organizations" WHERE "id" = 0
Statement.on(OrganizationTable)
		.where { o -> o.id eq 0 }
		.delete()
```

## TODO
* GROUP BY and HAVING clauses
* JOIN on more than two tables
* ALTER TABLE statement
* Subqueries (research and prototyping)
* Views (research)
