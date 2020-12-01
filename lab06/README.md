## Tarefa de Cypher e o FDA Adverse Event Reporting System (FAERS)

### Exercício 1

~~~cypher
CREATE (:Drug {drugbank: "DB04817", name: "Metamizole"})
~~~

### Exercício 2

~~~cypher
MATCH (d1:Drug {name:"Dipyrone"})
MATCH (d2:Drug {name:"Metamizole"})
CREATE (d1)-[:SameAs]->(d2)
~~~

### Exercício 3

~~~cypher
MATCH (d1:Drug {name:"Dipyrone"})
MATCH (d2:Drug {name:"Metamizole"})
MATCH (d1)-[r:SameAs]->(d2)
DELETE r
~~~

### Exercício 4

~~~cypher
MATCH (p1:Pathology)<-[a]-(d:Drug)-[b]->(p2:Pathology)
MERGE (p1)<-[r:Relates]->(p2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1
~~~

### Exercício 5

Obter a tabela drug-use, usando o MERGE para evitar repetições.

~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MERGE (p:Person {code: line.idperson})
MERGE (d:Drug {code: line.codedrug})
MERGE (p)-[:Takes]->(d)
~~~

Relacionar, em seguida, com os efeitos colaterais.

~~~cypher
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
MERGE (p:Person {code: line.idPerson})
MERGE (s:SideEffect {code: line.codePathology})
MERGE (s)-[:Affects]->(p)
~~~

Agora, falta apenas realizar a relação entre os medicamentos e seus efeitos colaterais.

~~~cypher
MATCH (s)-[:Affects]->(p)-[:Takes]->(d)
MERGE (d)-[c:Causes]->(s)
ON CREATE SET c.weight=1
ON MATCH SET c.weight=c.weight+1
~~~

### Exercício 6

Podemos selecionar a porção do grafo de remédios que estão mais relacionados a efeitos colaterais (peso > 15, por exemplo).

~~~cypher
MATCH (d)-[c:Causes]->(s)
WHERE c.weight>15
RETURN d,s
~~~

A partir disso, podemos fazer uma projeção com os medicamentos:

~~~cypher
MATCH (s1:SideEffect)<-[a]-(d:Drug)-[b]->(s2:SideEffect)
MERGE (s1)<-[r:Relates]->(s2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1
~~~