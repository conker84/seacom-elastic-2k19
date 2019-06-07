# Init

Lauch these two commands in order to create the sinks:

```bash
http post localhost:8083/connectors @connect.elastic.json
http post localhost:8083/connectors @connect.neo4j.json
```

```cypher
create constraint on (p:Person) assert p.id is unique;
create constraint on (m:Movie) assert m.id is unique;
```

Run the data generator: `java -jar kafka-connect-data-generator.jar`

Create the sink for the rank:

```bash
http post localhost:8083/connectors @connect.elastic.rank.json
```

Run the page-rank query in neo4j browser:

```cypher
CALL algo.pageRank.stream(
'MATCH (p:Person) RETURN p.id AS id',
'MATCH (p1:Person)-->()<--(p2:Person) RETURN distinct p1.id AS source, p2.id AS target',
{graph:'cypher'}) YIELD nodeId, score
MATCH (p:Person{id: nodeId})
CALL streams.publish('rank', {id: toString(p.id), entity: 'person', properties: {rank: score, name: p.name, born: p.born}})
RETURN nodeId, score
ORDER BY score DESC
```