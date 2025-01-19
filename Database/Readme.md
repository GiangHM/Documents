# Relational database (SQL Server)
## Normalizing form
### 1NF
- Each column must have a unique name.
- The order of the rows and columns doesn’t matter.
- Each column must have a single data type.
- No two rows can contain identical values. --> a table should have primary key.
- Each column must contain a single value. --> break multi values apart, into new table.
- Columns cannot contain repeating groups.

### 2NF
- It is in 1NF.
- All of the non-key fields depend on all of the key fields. ==>
- Violate when: table hold two kind of information/ or more, it means: non key field just depend on one of key field.
- To fix this problem: break the table into two/more table.
### 3NF
- It is in 2NF.
- It contains no transitive dependencies. A transitive dependency is when one non-key field’s value depends on another non-key field’s value.
- To fix this problem: find the fields and move them to new table.
### BCNF
### 4NF
### 5NF
=>  should stop at 3NF.

## Common design pitfalls
- Poor naming
- Thinking too small
- Not planning for change in the future: except/sometime/ usually -> keyword to think about future
- Too much normalization: 3NF is enough
- Increase database load

## Common design patterns
- Different kind of association between objects
 ++ many-to-many: ex: 1 student can take (n) course, and 1 course can have (n) students.
  +++ to build: add a association table
 ++ multiple many-to-many: want to know course that student register belongs to which semester, and also know more semester information
  +++ to build: add more columns in association table, that refer to new table "semester"
 ++ multiple object associations: 
   - Ex: 
		  

  +++ To build: divide complex relationship to many simpler relationship.

 ++ Repeated attribute associations: Ex: person may have many phone, email address
  +++ To build: build new table to contain the repeated data. use original table's PK to link new record back to original table
		- 
		- also you can use unique key... to perform some business rules like: unique purpose for one phone number...

	- Reflexive associations
		- One-to-One reflexive: ex: person is married to person
			- To build: create new table Marriage
		- One-to-many reflexive: ex: 1 employee can manage (n) employee
			- add more column 
			  

	- Hierarchical data
			- a instance One-to-Many reflexive association
			- Think about this pattern: example:Tom Referential: geographical data -> performance effect when do query

	- Network data: remember Tom context: 
		- We need table to store node information
		- Another table to store Link between node: FromNode, ToNode, LinkType

	- Temporal data: apply for online store/ banking (rate of credit/ rate deposit/ rate)
		- use Effective/Valid Date

		- delete object: consider using effective date, Or using another column to stock original data
		- also consider which data should Temporalize

	- Logging and locking
		- Logging: 
			- audit columns: createdby, modifiedby, createddate, modifieddate -> apply to Tom
			- audit table: create separate table to log changes of data -> apply to TOM with order history functionality 
		- Locking: when user modify data of a record, the record should be lock to make data consistency .
			- apply TOM:  with lock user edit order by block (General/ Routing....)
			- add more column: LockedBy; 
			- If we have process flow like: create order -> approve order -> ... 
				- we can use AssignedTo -> to notify, transfer order to right user to process it on next step

# Design table to support application
- Convert domain into table: It mean convert value range (like master data) into table
- Keep table focused: by using three kinds of tables
	- 1st: store information about object of a particular type
	- 2nd: to represent link between two or more objects.
	- 3rd: lookup table (or master data table)
- Should have conventions for your database
	- don't use special characters
	- foreign key should have same name
	- meaningful name
	- consistent with BE code/ FE code (model or entity)
- Use some redundancy and denormalized tables to improve performance

# Translate user need into data model
There are some data model:
- User interface model: can be defined when we look into form/ requirement..
- Semantic object:
- ERD: Entity relationship diagram => It's important to translate customer's requirement into ERD; After that we should optimize ERD before designing database
- Relation database model

# Extract business rule
- Extract to some table:  
	- validation list, it like master data -> convert to foreign key constraint.
	- calculation with parameters: -> new table to stock these params, load them
- Extract to stored procedure:
	- complicated calculation
- Extract to business layer : heavy business rule