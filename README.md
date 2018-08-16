# Overview
* How to import relational data between entities into local Neo4j using cypher

# Instructions:
* For setting up new local data run the following:
    * In database settings set:
    * `dbms.directories.import=/<home>/Documents/neo4j/import`
    * `dbms.security.allow_csv_import_from_file_urls=true`
* Install APOC plugin through application
* Each chunk needs to be ran to establish all relationships from the CSV files
* Once a file has been imported, increment a new file version so we don't cause duplicate relations

1. `CREATE CONSTRAINT ON (person:Person) ASSERT person.name IS UNIQUE`

2. Person - Person

```
with ['LOAN','LOBBIED','SALE','SUPPLIER','SHAREHOLDER','LICENSES','AFFILIATED','TIES','NEGOTIATION','INVOLVED','PARTNER','WORKED_AT','CONTRACTOR','FRIEND'] AS terms
LOAD CSV WITH HEADERS FROM "file:///<filename>.csv" AS row
WITH terms, row WHERE row.entity_a_type = 'Person' AND row.entity_b_type = 'Person'
WITH apoc.text.regreplace(toUpper(row.connection),'\\W+','_') AS type, row, terms
WITH coalesce(head(filter(term IN terms WHERE type CONTAINS term)), type) AS type, row
MERGE (o1:Person {name:toUpper(row.entity_a)})
MERGE (o2:Person {name:toUpper(row.entity_b)})
WITH o1,o2,type,row
CALL apoc.create.relationship(o1,type, {source:row.source, connection:toUpper(row.connection), startdate: row.startdate, enddate: row.enddate},o2) YIELD rel
RETURN type(rel), count(*)
ORDER BY count(*) DESC
```

3. Person - Organization

```
with ['LOAN','LOBBIED','SALE','SUPPLIER','SHAREHOLDER','LICENSES','AFFILIATED','TIES','NEGOTIATION','INVOLVED','PARTNER','WORKED_AT','CONTRACTOR','FRIEND'] AS terms
LOAD CSV WITH HEADERS FROM "file:///<filename>.csv" AS row
WITH terms, row WHERE row.entity_a_type = 'Person' AND row.entity_b_type = 'Organization'
WITH apoc.text.regreplace(toUpper(row.connection),'\\W+','_') AS type, row, terms
WITH coalesce(head(filter(term IN terms WHERE type CONTAINS term)), type) AS type, row
MERGE (o1:Person {name:toUpper(row.entity_a)})
MERGE (o2:Organization {name:toUpper(row.entity_b)})
WITH o1,o2,type,row
CALL apoc.create.relationship(o1,type, {source:row.source, connection:toUpper(row.connection), startdate: row.startdate, enddate: row.enddate},o2) YIELD rel
RETURN type(rel), count(*)
ORDER BY count(*) DESC
```

4. Organization - Organization

```
with ['LOAN','LOBBIED','SALE','SUPPLIER','SHAREHOLDER','LICENSES','AFFILIATED','TIES','NEGOTIATION','INVOLVED','PARTNER','WORKED_AT','CONTRACTOR','FRIEND'] AS terms
LOAD CSV WITH HEADERS FROM "file:///<filename>.csv" AS row
WITH terms, row WHERE row.entity_a_type = 'Organization' AND row.entity_b_type = 'Organization'
WITH apoc.text.regreplace(toUpper(row.connection),'\\W+','_') AS type, row, terms
WITH coalesce(head(filter(term IN terms WHERE type CONTAINS term)), type) AS type, row
MERGE (o1:Organization {name:toUpper(row.entity_a)})
MERGE (o2:Organization {name:toUpper(row.entity_b)})
WITH o1,o2,type,row
CALL apoc.create.relationship(o1,type, {source:row.source, connection:toUpper(row.connection), startdate: row.startdate, enddate: row.enddate},o2) YIELD rel
RETURN type(rel), count(*)
ORDER BY count(*) DESC
```

## Helpful Queries

match(n)-[r]->(m) return *
match(n) return n

## TODO
* how to remove duplicate relationships between orgs and persons, maybe do this on query side

