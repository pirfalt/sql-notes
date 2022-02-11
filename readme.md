# SQL what I like about it, but mostly, the horror 😱

Some reflections about the SQL language and "ecosystem".

## What I like 👍

- Set logic
- Relations
- Joins
- Input validation on write
- Transactions
- Support for exploratory queries
- Indexes
- Legacy, the good kind
- "Works everywhere"

## Dialects 🤔

### Classics 🗿

- [SQLite](https://www.sqlite.org/index.html) [(select)](https://www.sqlite.org/lang_select.html)
  > SQLite is a C-language library that implements a small, fast, self-contained, high-reliability, full-featured, SQL database engine. SQLite is the most used database engine in the world. SQLite is built into all mobile phones and most computers and comes bundled inside countless other applications that people use every day. More Information...
- [MySQL](https://dev.mysql.com/) [(select)](https://dev.mysql.com/doc/refman/8.0/en/select.html)\
  The "M" in "LAMP".
- [Postgres](https://www.postgresql.org/) [(select)](https://www.postgresql.org/docs/11/sql-select.html)
  > The World's Most Advanced Open Source Relational Database

### Hosted 😙

- [Aurora](https://aws.amazon.com/rds/aurora/)\
  👆 + AWS secret sauce.
  > Amazon Aurora is a MySQL and PostgreSQL-compatible relational database built for the cloud that combines the performance and availability of traditional enterprise databases with the simplicity and cost-effectiveness of open source databases.

### Scaled 🤯

- [Vitess](https://vitess.io/)
  > A database clustering system for horizontal scaling of MySQL
  - [PlanetScale](https://planetscale.com/)\
    Hosted Vitess.
    > The MySQL-compatible serverless database platform.
- [Spanner](https://cloud.google.com/spanner) [(select)](https://cloud.google.com/spanner/docs/reference/standard-sql/query-syntax)\
  Google secret sauce + GPS clocks.
  > Fully managed relational database with unlimited scale, strong consistency, and up to 99.999% availability.
- [Snowflake](https://docs.snowflake.com/en/sql-reference/constructs.html#query-syntax) [(select)](https://docs.snowflake.com/en/sql-reference/sql/select.html)
- [CockroachDB](https://www.cockroachlabs.com/) [(select)](https://www.cockroachlabs.com/docs/v21.2/select-clause)
  > A distributed SQL database designed for speed, scale, and survival. Trusted by thousands of innovators around the globe.

### Weird 🤪

- [Cassandra](https://cassandra.apache.org/doc/latest/cassandra/cql/dml.html#select-statement)
  > Open Source NoSQL Database
  > Manage massive amounts of data, fast, without losing sleep
  > `CQL` instead of `SQL`.
  > CQL does not execute joins or sub-queries and a select statement only apply to a single table.
- [Athena/S3 Select?](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-glacier-select-sql-reference-select.html)
  > Amazon S3 Select and S3 Glacier Select queries currently do not support subqueries or joins.
- [BigQuery](https://cloud.google.com/bigquery) [(select)](https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax)
  > Serverless, highly scalable, and cost-effective multicloud data warehouse designed for business agility.
- [Fauna](https://fauna.com/) [(read)](https://fauna.com/blog/getting-started-with-fql-faunadbs-native-query-language-part-1#read)\
  Relational database without SQL support.
  > Fauna is a flexible, developer-friendly, transactional database delivered as a secure and scalable cloud API with native GraphQL

## What I dislike 👎

### "Specification".

- [Wikipedia](https://en.wikipedia.org/wiki/SQL)
  - https://en.wikipedia.org/wiki/SQL#Overview
  - https://en.wikipedia.org/wiki/SQL#Reasons_for_incompatibility
- [ANSI](https://www.iso.org/standard/63555.html)

- "Works" _everywhere_.
  - But not the same.

Specification of user interface, not an interoperable protocol.

- Queries as strings (not data structure).
  - Which (every?) implementation changes.
  - List inputs/outputs.
- Java and Go have standardized APIs which helps. But an adapter is always required.

### Language

Queries as strings (not data structure). The format of the strings.

#### Read order

https://en.wikipedia.org/wiki/SQL_syntax#Queries

```
SELECT <columns>              5.
FROM <table>                  1.
WHERE <predicate on rows>     2.
GROUP BY <columns>            3.
HAVING <predicate on groups>  4.
ORDER BY <columns>            6.
OFFSET                        7.
FETCH FIRST                   8.
```

- Two different `<predicate>` syntaxes.

#### Non similar queries

```sql
SELECT column1, column2
FROM example
WHERE column2 = 'N';
```

```sql
INSERT INTO example
  (column1, column2, column3)
VALUES
  ('test', 'N', NULL);
```

```sql
UPDATE example
  SET column1 = 'updated value'
WHERE column2 = 'N';
```

```sql
DELETE FROM example
WHERE column2 = 'N';
```

#### Name bindings

Only somewhat hacky name bindings (variables are separate).

```sql
SELECT
  (
    SELECT COUNT(*)
    FROM example
    WHERE column2 = ?
  ) AS match_count,
  (
    SELECT COUNT(*)
    FROM example
    WHERE column2 <> ?
  ) AS non_match_count
;
```

```sql
WITH
  bindings(col2_val) AS (VALUES (?))
SELECT
  (
    SELECT COUNT(*)
    FROM example
    WHERE column2 = bindings.col2_val
  ) AS match_count,
  (
    SELECT COUNT(*)
    FROM example
    WHERE column2 <> bindings.col2_val
  ) AS non_match_count
;
```

### Multi network call transactions

- Can't submit full query at once -> `N+1` queries.
- See name binding 👆

```sql
BEGIN TRANSACTION;

  INSERT INTO talks
    (title, description)
  VALUES
    ('Sql notes', 'Ramblings');

  UPDATE user
    SET most_recent_talk_id = '???'
  WHERE user_name = 'Emil';

COMMIT TRANSACTION;
```

```js
await sqlClient.query(`begin transaction;`);

const createdTalk = await sqlClient.query(
  `
  INSERT INTO talks
    (title, description)
  VALUES
    ('Sql notes', 'Ramblings')
  RETURNING
    talks_id;
  `
);

const updatedUser = await sqlClient.query(
  `
  UPDATE user
    SET most_recent_talk_id = '?'
  WHERE user_name = 'Emil';
  `,
  [createdTalk]
);

await sqlClient.query(`commit transaction;`);
```

### Misc

- ORMs.
- Similar queries can have wildly different performance.
  - Completely unknown at query time.
- Upsert (Merge).
  - https://www.postgresql.org/message-id/CAM3SWZRP0c3g6+aJ=YYDGYAcTZg0xA8-1_FCVo5Xm7hrEL34kw@mail.gmail.com
  - https://www.postgresql.org/message-id/attachment/55920/sql-merge.html
  - https://www.postgresql.org/docs/10/unsupported-features-sql-standard.html (F312)
- Change detection, all have it but its hidden.

Also, "[coolest SQL hack I learned this week?](https://www.linkedin.com/posts/jtraubenheimer_sql-hack-sqltips-activity-6887328581970743297-nuoV/)"

## Final note

If you want to know more, I _highly_ recommend reading/listening to https://dataintensive.net/.
