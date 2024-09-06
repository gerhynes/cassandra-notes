# Cassandra Notes
Sources:
- [Apache Cassandra Database - Full Course for Beginners](https://youtu.be/J-cSy5MeMOA)
- [Introduction to Cassandra for Developers](https://youtu.be/jgqu1BcSKUI)

Cassandra is a NoSQL distributed database.

Each instance of Cassandra is called a node and this contains a full Cassandra database. Each node can handle ~2-4TB of data and several thousand operations per second per core.

Cassandra is a leaderless perr-to-peer system, meaning any node can do what any other node can. Nodes communicate through a protocol called GOSSIP. Nodes are organized into groups called data centres or rings.

Cassandra is a petabyte level database. It's designed for high availability even in cases of node failure. It does this through its distributed nature, automatic replication and leaderless topology. Datacentres can be globally distributed. 

Cassandra is known for its performance at scale. Writes usually happen at the speed of wire, while reads are usually within milliseconds. This is independent of the number of nodes. Cassandra scales linearly and can scale indefinitely.

Cassandra is also vendor agnostic and can be installed anywhere.

## Cassandra vs Relational Databases
Cassandra databases follow the Cassandra modelling methodology, while relational databases folow the sample relational model methodology.

With Cassandra, you think about the application needs when building the data model. Queries are king and must be conceptualized before the data model. Unlike relational databases, where entities are central.

In relational databases primary keys ensure the uniqueness of records. In Cassandra, they take on additional significance, particularly in determining performance on a large scale.

To provide performance and reliability on a large scale, Cassandra uses a distributed architecture.

#### ACID
By default, Cassandra doesn't support ACID transactions or joins, but uses denormalization instead. Cassandra also doesn't enforce referential integrity across tables.

In relational data modelling, queries aren't considered until a late stage in the process. With Cassandra, the application workflow and queries are considered earlier in the process.

In the relational modelling approach: you first think about the type of data, create a data model based on the data, and think about the application last. In the approach followed by Cassandra, the process is reversed. The application of the data must be decided up front, that dictates how the logical model takes shape, once your queries and data modela re complete, you can load the data into your tables.

Relational databases are ACID compliant by default.
- Atomicity - refers to the integrity of the database transacxtion, either all the statements in a transaction succeed or none of them
- Consistency - the data in a database should comply with certain rules. Transactions cannot leave the database in an inconsistent state.
- Isolation - multiple transactions should be able to be processed at the same time without interference.
- Durability - a completed transaction should persisten even after a subsequent failure. 

ACID causes a significant performance penalty, and is difficult to achieve in a distributed database.

However, Cassandra does demonstrate atomicity, isolation and durability on read and write commands on a single row. It also provides tunable consistency, which can be adjusted depending on the number of nodes that have to agree to execute a particular read or write command.

#### Consistency and the CAP Theorem
The CAP theorem (Consistency, Availability and Partition Tolerance) talks about the trade offs in implementing a distributed database.

All distributed databases are primarily tolerant to partitions, so the choice is between consistency and availability. Cassandra, by default, is an AP system. It chooses availability and partition tolerance while sacrificing consistency. It does, however, provide the option to make the system more consistent by sacrificing availability. 

Tunable consistency means that the consistency of the database can be tuned by adjusting the number of nodes that have to return a read or write request. Cassandra primarily operates under the AP model because all data eventually becomes consistent. 

#### Joins
Join statements combine two different tables to return a query result. The base requirement for a join command to work is that the results from two combined tables should be stored at one location. However, in distributed databases, the different partitions of data are stored on different nodes, and so joins would have a negative impact on latency.

Cassandra doesn't support joins but uses denormalization instead. Denormalization ensures that all the required information is confined to the appropriate table. This can duplicate data in multiple tables, but this is cost efficient and allows for faster retrieval of this data.

While using Cassandra, the developer can decide up front what tables they want their data to be in, for example `comments_by_video` and `comments_by_user`.

In a `comments_by_video` table, `video_title` is used as a partition key. `comment_id` helps in defining a unique row in the database for each comment.

In a `comments_by_user` table, `user_id` is used as the partition key, while `comment_id` helps define a unique row.

The conventional way of naming Cassandra tables is `ENTITY_by_QUERY`.

#### Referential Integrity
Joins rely on referential integrity constraints to combine data.

With relational databases, referential integrity means a value in one table must exist in another table. For example, a user must exist in both the `users` and `emails` tables, or none of them.

Cassandra doesn't enforce referential integrity because it would require a read before a write. This avoids certain performance issues. Referential integrioty can be enforced in an application design but that would require more work by the developer.

## Terminology
A **Data Model** is an abstract model for organizing elements of the data. They will vary depending on the type, capability and purpose of the database. For Cassandra, the data model is based on the queries you might want to perform.

A **Keyspace** is the outermost logical container of tables. It stores tables and replication data.

A **Table** is a combination of rows and columns contained within a keyspace.

As a columnal database, Cassandra stores data based on **Partitions**. A Partition is a row, or rows, of data stored on a node in the data table based on the partitioning strategy.

Each row in the partition consists of Key and Value pairs. Cassandra stores and retrieves data based on partitions. A Partition can make or break the data model.

A **Primary Key** is the most important part of the data model. It guarantees the uniqueness of the data, and it defines the placement of the record in the cluster. This allows for easy access to data in the model. 

A **Partition Key** is the first part of the Primary Key. It determines where in the cluster your data will be stored, on which node.

Inside each Partition there are Rows and Columns. Columns are stored locally and are referred to as cells in the table. A Row is contained inside a Partition. Rows are stored together on a Partition.

**Clustering Columns** define the order of data within a given partition, sorted in ascending order by default.

## CQL
Cassandra Query Language (CQL) is the primary language for communicating with Apache Cassandra databases. The most basic way to interact with Cassandra is via the CQL shell, cqlsh.

A **Keyspace** is a top-level container in Cassandra to organize a related set of tables. It's similar to a relational database's schema. Keyspaces define replication settings, which describe how many copies of a given piece of data are stored in a cluster.

```CQL
CREATE KEYSPACE kpopvideos WITHREPLICATION={'class':'SimpleStrategy', 'replication_factor': 1}; 
```

You can use `USE KEYSPACE_NAME;` to switch between keyspaces. If you don't use `USE`, you'll have to reference the keyspace name whenever you reference the name of a table. This helps Cassandra interpret your requests.

Keyspaces contain tables and tables contain data. Every Cassandra table has a Primary Key.

```CQL
CREATE TABLE users (
	user_id UUID,
	first_name TEXT,
	last_name TEXT,
	PRIMARY KEY (user_id)
)
```

The Primary Key clause as a whole uniquely identifies rows of data in a table.

The `SELECT` command is used to pull data from a table. Cassandra suopports a paging mechanism to read specific columns. Cassandra also supports a couple of built-in aggregation functions, such as `COUNT`.

```CQL
SELECT * FROM table1;

SELECT column1, column2 FROM table1;

SELECT * FROM table1 LIMIT 10;

SELECT COUNT(*) FROM table1;
```

The `TRUNCATE` command removes all rows from a table while leaving the schema in place. Once removed, the data can only be restored from backups. `TRUNCATE` sends a JMX command to all nodes. The command will fail if any nodes are down.

The `ALTER TABLE` command lets you change the datatype of a column, add columns, drop columns, rename columns, and change table properties. But you cannot change the columns in a Primary Key clause.

```CQL
ALTER TABLE table1 ADD new_column text;
ALTER TABLE table1 DROP new_column;
```

The `SOURCE` command lets you execute a set of CQL statements from a file. The filename must be enclosed in singlequotes.

```CQL
SOURCE './myscript.cql';
```

Cqlsh will output the results of each command sequentially as it executes.

## Partitions
Partitions give you an indication of where your data is in your data model as well as in the cluster.

The Partition Key is always the first value in a Primary Key.

For example a table with three columns, `country`, `city`, `population` could eb partitioned on the country value. Sicne these have the same partition key they will be stored together within a node and data will automatically be distributed around the nodes in the cluster. 

Rows with the same partition key will be stored in the same physical partition on diask.

You choose the partition key for a table and Cassandra will handle the data distribution.

If you lose the node containing a particular partition the data is still secure due to replication.

## Replication
Each node has a Partition Token assigned to it. When you add a Partition Key to a table, that value is automatically hashed out to a token value. Each node "owns" a particular token range. This is how Cassandra knows where to store and retrieve your data. The Partition Key is essentially the address of your data.

A Replication Factor of 1 would mean a single ring, meaning one copy of a partition on one node.

If you increase the Replication Factor to 2, you now have two rings and two nodes containing a partition.

A Replication Factor of 3 (which is standard) means three copies of a partition are stored on three different nodes. This gives a good balance between availability, performance and consistency.

As a request comes in, any node is chosen to handle the request. This becomes the **Coordinator Node**. Any node can be a coordinator. The Coordinator forwards the write to the three nodes with the appropriate token range and each node stores a copy.

If a node is down during this process, Cassandra stores a hint. Once the node comes back up, the write is automtaically played back to heal the node.

### Consistency
Just as you can set a Replication Factor, you can set a Consistency Level, such as `QUORUM`. With a replication factor of 3 and using quorum, Cassandra will wait for an acknowledgement from 2 of the 3 nodes before confimring that a write is ok and sending a result back to the client.

If consistency level cannot be reached for an operation, that operation will fail.

For a read operation, the client makes and request and the coordinatorn reads from 2 of the 3 nodes, and the result is sent back to the client. If one node is stale, Cassandra will automatically repair the data and then send the result to the client.

The combination of write at quorum and read at quorum is called **Immediate Consistency**.

## Denormalization

### Data Types
Data types, such as Collections, Counters and User Defined Types simplify table design, optimize table functionality, store data more efficiently and might change table designs completely.

**Collections** group and store data together in a single column. Collection columns are multi-valued columns, designed to store a small amount of data which will be retrieved in its entirety.

They cannot be part of a Primary Key, Partition Key, or Clustering Column. You cannot nest a collection inside another collection unless you use the `FROZEN` keyword. 

There are three collection types: Set, List and Map.

A **Set** is a list that stores a typed collection of unique values. They are stored unordered but returned in sorted order. 

```CQL
CREATE TABLE users (
	id text PRIMARY KEY,
	fname text,
	lname text,
	email: set<text>
);

INSERT INTO users (id, fname, lname, email)
VALUES (
	'cass123',
	'Cassandra',
	'Dev',
	{'cass@dev.com', 'cassd@gmail.com'}
);
```

A **List** groups and stores values in a single cell. There can be duplicate values. They are stored in a particular order and retrieved according to an index.  

```CQL
ALTER TABLE users ADD freq_dest list<text>;

UPDATE users SET freq_dest = ('Berlin', 'London', 'Paris') WHERE id = 'cass123';
```

A **Map** lets you enter values related to each other. They are ordered by unique key. You use the `SET` command to enter the values.

```CQL
ALTER TABLE users ADD todos = map<timestamp, text>;

UPDATE users SET todos = {
	'2018-01-01':'create database',
	'2018-01-02':'load data and test',
	'2018-02-01':'deploy to production',
} WHERE id = 'cass123';
```

`FROZEN` lets you nest datatypes and serialize multiple components into a single value. Values in a `FROZEN` collection are treated like blobs, while non-frozen types allow updates to individual fields.

**User Defined Types** (UDTs) group together related fields of information. They can include any supported data types, including other collections and other UDTs. You can attach multiple data fields, each named and typed to a single column. They allow for embedding more complex data within a single column, adding flexibility to your data model.

```CQL
CREATE TYPE address (
	street text,
	city text,
	zipcode text,
	phones set<text>
);

CREATE TYPE full_name (
	first_name text,
	last_name text
);

CREATE TABLE users (
	id uuid,
	name frozen <full_name>,
	direct_reports set<frozen <full_name>>,
	addresses map<frozen <text, address>>
	PRIMARY KEY (id)
);
```

A **Counter** is a data type with columns that can store a 64-bit signed integer. It can be incremented/decremented and values are changed using `UPDATE`. Counters need specially dedicated tables, can only have a Primary Key and Counter columns. They can lead to duplicate data in multiple tables.

```CQL
CREATE TABLE moo_counts (
	cow_name text,
	moo_count counter,
	PRIMARY KEY (cow_name)
);

UPDATE moo_counts 
SET moo_count = mnoo_count + 8
WHERE cow_name = 'Betsy';
```

## Data Modelling
### Conceptual Data Model
The **Conceptual Data Model** is responsible for determining the attributes of all the objects in your domain and analyzing how they are related. A Conceptual model defines what a model should contain, for example users, comments, videos, ratings. Its purpose is to understand your data and identify essential objects and constraints.

It's critical that your database solutions directly serve the business' needs. This will require collaboration between technical and non-technical staff.

The Conceptual Data Model offers an abstract view of your domain. It's generally independent of technology and not specific to any database system.

Cardinality refers to the relationship between two entities in a data model. It specifies how many times an entity can or must participate in a relationship.

Attribute types are the fields that store properties about an entity or relationship. 

Key attributes uniquely identify an entity.

Multi-valued attributes store multiple values per attribute.

Relationship keys can be One to One, One to Many, or Many to Many. 

Weak entity types are entities that cannot exist without a corresponding strong entity on the other side.

### Workflow and Access Pattern
Application wokrflow describes how users will navigate through the application. It also helps determine which queries you will perform against a Cassandra database.

Every application has a workflow. Access patterns help us to determine how the data is accessed, and so what queries to run first.

Access patterns might be:
1. Find a user with a specified email
2. Find most recently uploaded videos
3. Find a user with a specified id
4. Find videos uploaded by a user with a known id (order by most recently uploaded)
5. Find a video with a specified video id

### Mapping and Conceptual Model
To make a Logical Data Model, you need to map your Conceptual Data Model to a logical one.

The mapping rules ensure that a logical data model is correct, each query has a corresponding table, tables are designed to allow queries to execute properly, and the tables return data in the correct order.

The mapping rules are:
1. Identify entities and their relationships
2. Identify the equality search attributes
3. Identify the inequality search attributes
4. Identify the ordering attributes
5. Identify the key attributes

You will create a table schema from the conceptual data model for each query and apply the mapping rules in order.

To get a Logical Data Model you combine the results of the Conceptual Data Model and the Access Patterns, and apply mapping rules and patterns.

Logical Data Models are described using a Chebotko Diagram. Chebotko Diagrams are graphical representations of Cassandra database schema designs. They document the logical and physical data models respectively.

On a logical level, the Chebotko Diagram shows the column names and properties. On a physical level, it shows the column data types.

|table_name||||
|----|---|:---:|---|
|column_1|CQL Type|K| Partition Key Column|
|column_2|CQL Type|C↑| Clustering Key Column (ASC) |
|column_3|CQL Type|C↓| Clustering Key Column (DESC)|
|column_4|CQL Type|S|Static Column|
|column_5|CQL Type|IDX|Secondary Index Column|
|column_6|CQL Type|↑↑|Counter Column|
|[column_7]|CQL Type||Counter Column (list)|
|{column_8}|CQL Type||Counter Column (set)|
|<column_9>|CQL Type||Counter Column (map)|
|\*column_10\*|CQL Type||UDT Column|
|(column_11)|CQL Type||Tuple Column|
|column_12|CQL Type||Regular Column|

UDT is a datatype and can be nested within one another.

The four main principles of Cassandra data modelling are:
- Know your data
- Know your queries
- Nest data
- Duplicate data

Understanding the data is key to successfully implementing a database. In cassandra, space is sacrificed to reduce time and so organizing data is of utmost importance. 

Entity and relationship keys affect the table's primary keys. A Primary Key uniquely identifies a row/entity/relationship.

For example, a `videos` table with a primary key of `video_id` doesn't organize data to satisfy queries. By contrast, a `videos_by_user` table with a primary key of `user_id`, and clustering columns of `uploaded_timestamp` and `video_id` makes it easier to query and return results in a specified order.

Data cardinality helps in selecting an efficient partition key while the relationship cardinality helps in satisfying queries.

Queries are captured by the application workflow models and each entity is created with queries in mind. In Cassandra, it's always better to reduce the number of partitions that have to be read for a query result. Table schemas will change if queries change.

A single partition per query is the most efficient access pattern. Entities should be designed so that queries can generate results with the minimum number of reads, ideally from a single partition search.

If you want to query for a particular partition, you can provide the partition key and thus don't need to filter or search the entire cluster.

There are, however, cases where multiple partitions need to be accessed to retrieve the results. This less-efficient approach is acceptable as long as these kind of queries are limited in number.

The anti-pattern that is to be avoided at all costs is asking the database to perform a linear search through all partitions and tables. This would defeat the whole purpose of having a horizontally-scaled database.

### Logical Model
The working rule with Cassandra is to group as much data as possible on disk. This is called data nesting and is the main data modelling technique. 

Nesting helps to organize multiple entities into a single partition. It also helps support partition-per-query during data access.

Schema design should be driven by application queries.

The three main ways to nest data in cassandra are:
- Clustering Columns
- Collection Columns
- UDT Columns

Clustering columns are the primary data nesting mechanism. The partition key identifies an entity that other entities will nest into. Values in a clustering column identify the nested entities. When a table has multiple clustering columns, the data is stored in a nested sorted order (multi-level nesting).

UDTs are a secondary data nesting mechanism. They usually represent a one-to-one relationship but they cna be used in conjunction with collections. Working with UDTs is easier than working with multiple collection columns. For example, the `videos_by_user` table nests all videos as a collection in the `videos` column with the `video_type` type.

Partition per query and data nesting may result in data duplication. Cassandra effectively performs joins on writes instead of reads to save time. For example, video data may be duplicated in the `videos_by_actor`, `videos_by_genre` and `videos_by_tag` tables.

### Physical Model
To create a Physical Model, you add data types to each column of the Logical Model. Physical Models also optimize efficiency and performance, ensuring that parition sizes don't grow too large and will return queries quickly.

|comments_by_user|||
|--|--|---|
|user_id|K| UUID|
|posted_timestamp|C|TIMESTAMP|
|video_id|C|TIMEUUID|
|comment||TEXT|
|title||TEXT|
|type||TEXT|
|{tags}||SET\<TEXT\>|
|\<preview_thumbnails\>||MAP\<INT.BLOB\>|

```CQL
CREATE TABLE comments_by_user(
	user_id UUID,
	posted_timestamp TIMESTAMP,
	video_id TIMEUUID,
	comment TEXT,
	title TEXT,
	type TEXT,
	tags SET<TEXT>,
	preview_thumbnails MAP<INT,BLOB>,
	PRIMARY KEY ((user_id), posted_timestamp, video_id)
) WITH CLUSTERING ORDER BY (posted_timestamp, DESC, video_id ASC);
```

If you're migrating from an existing database, there are a few ways to get data into a table.

The `COPY` command can insert c.2 million values into a database.

The SSTable Loader is used when migrating from one cluster to another.

Spark for Data Loading has several options (and doesn't require a working knowledge of Spark).

`COPY TO` exports data from a table to a CSV file.

`COPY FROM` imports data from a CSV file into a table.

The process verifies the primary key and updates the existing records.

If `HEADER` is false, it specifies that the fields are imported in a deterministic order. 

When column names are specified, fields are imported in that order. If any field is missing or empty, it's set to null.

The source cannot have more fields than the target table, but it can have fewer.

```CQL
COPY table1(column1, column2, column3) FROM 'tabledata.csv' WITH HEADER=true;
```

## Use Cases
Cassandra supports Geographic Distribution. You can put data near your users (with the lowest response time) and use Cassandra's inherent replication to move data where it needs to go.

Cassandra works with all cloud providers, as well as on-premise installations. It can be operated in hybrid-cloud and multi-cloud environments. 

Cassandra is good for scalability, handling both high throughput and high volume, availability for mission-critical systems, global distribution for global presence and workload mobility.




Thanks. That would be really great. I'll bring it up in scrum tomorrow. Having a couple of tickets to work on over the next few weeks would be really helpful, 


