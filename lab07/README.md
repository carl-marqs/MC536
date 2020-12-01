## Análise de Redes

### Exercício 1

~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/network/pagerank/pagerank-wikipedia.csv' AS line
MERGE (p1:WikiPage {name:line.source})
MERGE (p2:WikiPage {name:line.target})
CREATE (p1)-[:WikiLinks]->(p2)

CALL gds.graph.create(
  'WikiGraph',
  'WikiPage',
  'WikiLinks'
)

CALL gds.pageRank.stream('WikiGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY score DESC, name ASC

CALL gds.pageRank.stream('WikiGraph')
YIELD nodeId, score
MATCH (p:WikiPage {name: gds.util.asNode(nodeId).name})
SET p.pagerank = score
~~~


### Exercício 2

Carregando o lab anterior:

~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug.csv' AS line
CREATE (:Drug {code: line.code, name: line.name});
CREATE INDEX ON :Drug(code);

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/pathology.csv' AS line
CREATE (:Pathology { code: line.code, name: line.name});
CREATE INDEX ON :Pathology(code);

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MATCH (d:Drug {code: line.codedrug})
MATCH (p:Pathology {code: line.codepathology})
MERGE (d)-[t:Treats]->(p)
ON CREATE SET t.weight=1
ON MATCH SET t.weight=t.weight+1;

MATCH (d1:Drug)-[a]->(p:Pathology)<-[b]-(d2:Drug)
WHERE a.weight > 20 AND b.weight > 20
MERGE (d1)<-[r:Relates]->(d2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1;

MATCH (d1:Drug)<-[:Relates]->(d2:Drug)
RETURN d1, d2;
~~~

Criando o grafo:

~~~cypher
CALL gds.graph.create(
  'drugRelationGraph',
  'Drug',
  {
    Relates: {
      orientation: 'UNDIRECTED'
    }
  }
)

CALL gds.louvain.stream('drugRelationGraph')
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).name AS name, communityId
ORDER BY communityId ASC

CALL gds.louvain.stream('drugRelationGraph')
YIELD nodeId, communityId
MATCH (d:Drug {name: gds.util.asNode(nodeId).name})
SET d.community = communityId
~~~