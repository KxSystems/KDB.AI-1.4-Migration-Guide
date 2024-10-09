# KDB.AI 1.4 Migration Guide

We have introduced powerful changes in KDB.AI 1.4 that improve indexing, table management, hybrid search, and integration with KDB+. These updates bring significant benefits, including improved performance, scalability, and flexibility for managing AI workloads, but they also necessitate changes to your existing code. This guide will help you navigate those changes to ensure a smooth migration from previous versions to version 1.4.

## What Has Changed

The KDB.AI 1.4 update introduces several key changes:

1. **Multi-Index Support**: You can now create and use multiple indexes on a single table, allowing for enhanced search capabilities, including assigning weights to indexes.
2. **Database Layer Above Tables**: A new database layer improves organization and management by allowing a database to contain multiple tables.
3. **Enhanced Index Management**: New APIs for creating, deleting, and updating indexes provide greater control.
4. **KDB+ Integration Improvements**: Seamless reference and indexing of KDB+ tables directly from KDB.AI, including running Time Series Similarity (TSS) searches.
5. **New q API and Enhanced REST API**: Expanded developer experience, including a public q API and improved RESTful API design.
6. **Hybrid Search**: A new feature allowing combined sparse and dense search, improving result relevancy for a variety of use cases.

## Important Migration Notice

KDB.AI version 1.4 will be released on **October 17th**. This update introduces breaking changes that will affect all current applications, especially those in production. Applications will need to undergo **downtime** in order to upgrade.

- **Migration Timing**: You will not be able to migrate until version 1.4 is released in the cloud on October 17th. Once released, it is recommended that you migrate **immediately**, as previous versions will no longer function correctly.
- **Server Versions**: If you are using server version 1.2 or 1.3, migration is not mandatory, and there will be no breaking changes. However, it is recommended that you upgrade your server to version 1.4 to ensure future compatibility. Always ensure that you are using the same version of the KDB.AI client library as your server instance.

While these changes include significant benefits, they require immediate action for those relying on previous versions in production environments. We believe this will create a more robust product in the long term, making the experience better for all users.

## Migration Steps

Follow these steps to migrate your code from previous versions to version 1.4.

### 1. Table Creation Syntax Changes

In version 1.4, table creation requires the explicit separation of schema and index definitions, allowing multiple indexes to be defined on the same table. Schema definition syntax has also been simplified. We now recommend using 'float64s' as the type for vectorIndexes. 

#### Old Syntax:

```python
schema = {
    "columns": [
        {"name": "id", "pytype": "str"},
        {"name": "embeddings", "vectorIndex": {"dims": 384, "metric": "CS", "type": "flat"}},
    ]
}

table = database.create_table("my_table", schema=schema)
```

#### New Syntax:

```python
schema = [
    {"name": "id", "type": "str"},
    {"name": "vectors", "type": "float64s"}
]

indexes = [
    {
        "name": "vectorIndex", "type": "flat",
        "params": {"dims": 384, "metric": "CS"},
        "column": "vectors"
    }
]

table = database.create_table("my_table", schema=schema, indexes=indexes)
```

### 2. Adding the Database Layer

The new version introduces a database layer to manage tables, making data organization more scalable. You must now create or connect to a database before creating tables. The 'default' database always exists and will be used when there is only one database. It cannot be deleted, but the tables within it can be managed or deleted.

#### Old Syntax:

```python
session = kdbai.Session(api_key=API_KEY, endpoint=ENDPOINT)
table = session.table("my_table")
```

#### New Syntax:

```python
session = kdbai.Session(api_key=API_KEY, endpoint=ENDPOINT)
database = session.database('default')
table = database.table("my_table")
```

### 3. Search Method Updates

The search method has been updated to reference specific indexes, allowing for simultaneous multi-index searches with weight assignment. You must now explicitly include the index you want to search.

#### Old Syntax:

```python
results = table.search(vectors=[query_vector], n=3)
```

#### New Syntax:

```python
results = table.search(vectors={'vectorIndex': [query_vector]}, n=3)
```

### 4. Hybrid Search Example

Hybrid search in KDB.AI 1.4 allows users to combine sparse and dense vector searches to improve relevancy.

#### Old Syntax for Hybrid Search:

```python
table.hybrid_search(dense_vectors=dense_query, sparse_vectors=sparse_query, n=5, sparse_index_options={'b': 0.1, 'k': 3})
```

#### New Syntax for Hybrid Search:

```python
results = table.search(
    vectors={"sparse_index": sparse_query, "dense_index": dense_query},
    index_params={"sparse_index": {'weight': 0.1, 'b': 0.1, 'k': 3}, "dense_index": {'weight': 0.9}},
    n=5
)
```

## Common Pitfalls to Watch Out For

- Ensure all index definitions are updated to match the new format.
- Always explicitly reference a database before interacting with tables. The database 'default' will be used automatically when a new instance is created.
- Ensure all search queries include the correct index names.
- Some default behaviors may have changed, so make sure to test thoroughly.

## Support

We understand this migration might be challenging, and our support team is ready to help. Please reach out via our community Slack at kx.com/slack if you have any questions.

Your efforts to migrate to version 1.4 will contribute to a more stable and efficient experience for everyone. Let's make this transition together!

Best regards,\
Michael Ryaboy\
Developer Advocate, KDB.AI Team
