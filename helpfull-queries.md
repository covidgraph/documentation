[Back to CovidGraph Wiki](https://github.com/covidgraph/documentation/wiki)
# Helpful queries on the CovidGraph

## Papers

- List Fulltext papers with title

```cypher
MATCH (p:Paper)-[pr:PAPER_HAS_BODY_TEXT]->(c:Body_text:CollectionHub)-[r:BODY_TEXT_HAS_BODY_TEXT]->(t:Body_text)
WITH p.title as title,collect({txt:t.text, pos:r.position}) as text
UNWIND text as t
WITH title, t
order by t.pos
RETURN title, collect(t.txt) 
limit 4
```

- Get Papers and Authors

```cypher
MATCH (a:Author)<-[:AUTHOR_HAS_AUTHOR]-(:Author:CollectionHub)<-[:METADATA_HAS_AUTHOR]-(:Metadata)<-[:PAPER_HAS_METADATA]-(p:Paper)
RETURN a, p, apoc.create.vRelationship(a, 'AUTHORED',{}, p) as vrel 
limit 100
```

- Genes connected to papers

```cypher
MATCH (p:Paper)  MATCH (p)-[:MENTIONS]->(g:GeneSymbol)
RETURN p,g as result ORDER BY g.sid
limit 10
```

- Number of authors by location/region

```cypher
MATCH (loc:Location)<-[:AFFILIATION_HAS_LOCATION]-(aff:Affiliation)-[:AUTHOR_HAS_AFFILIATION]-(a:Author) 
WHERE loc.country IS NOT NULL 
RETURN loc.country as country, loc.region as region, count(distinct a.email) AS NbrOfAuthors 
ORDER BY count(distinct a.email) DESC
```

- Titles and dates of papers whose body text contains a user-specified keyword (e.g. Virus), ordered by date of publication. 

```cypher
MATCH (p:Paper)-[:PAPER_HAS_BODY_TEXT]->(b:Body_text)  
WHERE p.title IS NOT NULL AND p.title CONTAINS("Virus") 
RETURN p.title, p.publish_time 
ORDER BY p.publish_time DESC 
LIMIT 20
```

- Number of papers whose body text contains a user-specified keyword (e.g. Virus).

```cypher
MATCH (p:Paper)-[:PAPER_HAS_BODY_TEXT]->(b:Body_text)  
WHERE p.title IS NOT NULL AND p.title CONTAINS("Virus") 
RETURN count(p)
LIMIT 20
```

## Patents

- Find genes and proteins that are mentioned in patents

```cypher
match path=(e:Entity)<-[x:APPLICANT]-(p:Patent)-[y:HAS_CLAIM|:HAS_ABSTRACT|:HAS_TITLE]->(pa)-[z:HAS_FRAGMENT]->(ff:Fragment)-[m:MENTIONS]->(syn:GeneSymbol)-[:SYNONYM]->(gs:GeneSymbol)<-[:MAPS]-(g:Gene)-[:CODES]->(tc:Transcript)-[:CODES]->(pro:Protein)
where e.idLower starts with 'novar' 
return path limit 40
```

- Find gene names mentioned in patents

```cypher
match (p:Patent)-[x:HAS_TITLE|:HAS_CLAIM|:HAS_TITLE]-(pct)-[:HAS_FRAGMENT]->(f2:Fragment)-[:MENTIONS]->(gs2:GeneSymbol) return p,x,pct,gs2 limit 300
```

- Search patents with string against a textindex and get a hit score

```cypher
call db.index.fulltext.queryNodes("patents","Corona") 
yield node,score match (node)--(p:Patent)--(pt:PatentTitle)
return distinct(p.id) as id, collect(pt.text) as titles, labels(node)[0] as found_type, node.lang as found_in_lang ,score
order by score
desc limit 10
```

```cypher
call db.index.fulltext.queryNodes("fragments","corona and virus") 
yield node as f,score match (f)--(px)--(p:Patent) 
match (fp:Fragment)-[:NEXT]->(f),(f)-[:NEXT]->(fn:Fragment) 
return f.kind,[fp.text,f.text,fn.text],p.id,score 
order by score desc 
limit 10
``` 

- Find matching fragments in patent text

``` cypher
call db.index.fulltext.queryNodes("fragments","corona and virus") 
yield node as f,score match (f)--(px)--(p:Patent) 
return f.kind,f.text,p.id,score 
order by score desc 
limit 10
```

- This shows the previous and next fragment in the result 

```cypher
call db.index.fulltext.queryNodes("fragments","corona and virus") 
yield node as f,score match (f)--(px)--(p:Patent) 
match (fp:Fragment)-[:NEXT]->(f),(f)-[:NEXT]->(fn:Fragment) 
return f.kind,fp.text,f.text,fn.text,p.id,score 
order by score desc 
limit 10
```
## Authors

- Ranking authors. First create a projection on the graph, then call the PageRank algorithm:

```cypher
CALL gds.graph.create.cypher(
    'Authors_Influence',
    'MATCH (n:Author) RETURN id(n) AS id',
    'MATCH (a:Author)-[:AUTHOR_HAS_AUTHOR]->(b:Author) RETURN id(a) AS source, id(b) AS target'
)
YIELD graphName, nodeCount, relationshipCount, createMillis;
```
```cypher
CALL gds.pageRank.stream('Authors_Influence') YIELD nodeId, score RETURN gds.util.asNode(nodeId).first, gds.util.asNode(nodeId).last, score ORDER BY score DESC
```

