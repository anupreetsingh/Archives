A data model defines how your data is Structured, Stored and Related

![Data Modeling](<Media/Data Modeling/Data Modeling.png>)

An example of how the DB design is supposed to look like in the system design diagram:

![Example Data](<Media/Data Modeling/Example Data Model.png>)

## Database Model Options

### Relational Database

This is the **default** option for using in System design
Example: PostgreSQL, MySQL

An example data model for a social media app using a relational DB where each table is one of the core entities mentioned in the second stage of the system design.

![Relation DB](<Media/Data Modeling/Relational DB.png>)

Choosing the correct relation makes queries down the line much simpler.

### Document Database

Would use this when you want to support **schema flexibility**.
Example: MongoDB, Firestore.

But since you normally would have agreed on the schema in the core entities section and so there isn't much chance of its evolving in the context of an interview you would stick with the Relational option.

Also a social media data model using a document store:
![Document DB](<Media/Data Modeling/Document DB.png>)

The reason we split based on John, Jane, Bob instead of Users, Posts and Like is that non relational DB don't usually have the same level of support for `JOINs` as relational database.

### Key-Value Store

Key-value stores are fast as expected because you would search by using the key.

But they are also limited and you cannot do things like get all posts from last week or posts from users that I follow. Which is why we quite often **denormalize** heavily and duplicate data across multiple keys.

Example: Redis, DynamoDB.
![Key Value](<Media/Data Modeling/Key value DB.png>)

### Wide Column Database

They differ from traditional relational database because each row can have completely different number of columns. It's meant to support **massive write** volumes because you are always appending coolums instead of updating entries. Suitable for situations like time series database, event logging, etc where you need to store but not query them often.
Example: Apache Cassandra, Google Bigtable, Apache HBase

![Wide Column Database](<Media/Data Modeling/Wide Column DB.png>)

### Graph Database

Not a recommended option for a system design interview.

![Wide Column Database](<System Design/Media/Data Modeling/Graph DB.png>)

## Schema Design

Three key factors that affect the decision of your schema design. These factors would have been decided either during the requirements or API discussion and are:

1. Data Volume - Where can data live? (Single DB vs distributed)

2. Access Patterns - How is data queried? (Drives indexes & structure)

3. Consistency Requirements - How strict? (ACID vs eventual consistency)

![Relationships](<Media/Data Modeling/Relationships.png>)

Foreign Keys in a table enforce **referential integrity** meaning you can only enter its value if it exists in the other table.

There are also some **constraints** associated like Emails being unique or name not being null.

Different kinds of relationships are:

- 1-to-Many(1:N): One row in one table can be related to many rows in another table. For example, one customer can have many orders. In this case, the `Order` table contains `customer_id` as a foreign key referencing the Customer table.

- Many-to-1(N:1): Many rows in one table can be related to one row in another table. For example, many orders can belong to one customer. This is the same relationship as 1-to-many, just viewed from the opposite direction. The `Order` table still contains `customer_id` as the foreign key.

- Many-to-Many(M:N): Many rows in one table can be related to many rows in another table. For example, a student can take many courses, and a course can have many students. This relationship is usually represented using a junction table, such as `StudentCourse`, which contains `student_id` and `course_id` as foreign keys referencing the `Student` and `Course` tables.

## Indexing

**Indexing** is the process within a database where you create datastructures that help the database find records more quickly than having to scan every row.

It is the first step you take to make data access faster.

Example: If you want to query all posts for a given user. You would create an index for user_id(fk) on the post table
Example: If you want to be able to sort using createdAt then we add an index at createdAt.

![Indexing](<Media/Data Modeling/Indexing.png>)

In this example we use a B-Tree, which makes access logarithmic times faster than a linear scan.

### B-Tree

A **B-Tree** is a balanced search tree that stores sorted keys and allows each node to have multiple children(Unlike a BST). Its high branching factor keeps the tree shallow, making it well suited to database indexes because fewer nodes—and therefore fewer disk pages—must be read.

Its main properties are:

- Keys within each node are sorted.
- All leaf nodes remain at the same depth, so the tree stays balanced.
- Each non-root node has a defined minimum and maximum number of keys, determined by the tree's order. The root is allowed to contain fewer keys.
- An internal node containing $k$ keys has $k + 1$ children; leaf nodes have no children.
- Search, insertion, and deletion take $O(\log n)$ time.
- Nodes split or merge as data changes to preserve the tree's balance.

## Normalization vs Denormalization

![Normalization and Denormalization](<Media/Data Modeling/Normalization vs Denormalization.png>)

**Normalization** means you are storing each piece of information in exactly one location. There is no redundancy.

So in the example on the left, User data lives only on the User table and its not duplicated across any other tables. This prevents anomalies where you might update a users Email in one place but forget to update it in one place.

This does not mean that foreign keys, such as user_id, cannot appear in other tables. In the Post table, user_id represents information specific to that post—namely, which user created it—while also establishing a relationship with the corresponding record in the User table. What normalization avoids is duplicating the user's actual data, such as storing the same email address in both the User and Post tables.

**Denormalization** means deliberately duplicating data across tables. This is almost always for performance reasons. Like Instead of having to query two separate tables in separate places(using JOINs or calling two different shards) you have information in one table to get quick read access.

This does add the challenge of having to manage stale data and anomalies caused by inconsistent updates.

Often best to put denormalized data in the cache for improved performance.  
