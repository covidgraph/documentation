# Helpful queries on the CovidGraph

## Papers

- List Fulltext papers with title

```cypher
MATCH (p:Paper)-[pr:PAPER_HAS_BODY_TEXT]->(c:Body_text:CollectionHub)-[r:BODY_TEXT_HAS_BODY_TEXT]->(t:Body_text)
WITH p.title as title,collect({txt:t.text, pos:r.position}) as text
UNWIND text as t
WITH title, t
order by t.pos
RETURN title, collect(t.txt) limit 4
```

- Get Papers and Authors

```cypher
MATCH (a:Author)<-[:AUTHOR_HAS_AUTHOR]-(:Author:CollectionHub)<-[:METADATA_HAS_AUTHOR]-(:Metadata)<-[:PAPER_HAS_METADATA]-(p:Paper)
RETURN a, p, apoc.create.vRelationship(a, 'AUTHORED',{}, p) as vrel limit 100
```

- Genes connected to papers

```cypher
MATCH (p:Paper)  MATCH (p)-[:MENTIONS]->(g:GeneSymbol)
RETURN p,g as result ORDER BY g.sid
limit 10
```

## Patents

- Search patents with string against a textindex and get a hit score

```cypher
call db.index.fulltext.queryNodes("patents","Corona") yield node,score match (node)--(p:Patent)--(pt:PatentTitle)
return distinct(p.id) as id, collect(pt.text) as titles, labels(node)[0] as found_type, node.lang as found_in_lang ,score
order by score
desc limit 10
```

```cypher
call db.index.fulltext.queryNodes("fragments","corona and virus") yield node as f,score match (f)--(px)--(p:Patent) match (fp:Fragment)-[:NEXT]->(f),(f)-[:NEXT]->(fn:Fragment) return f.kind,[fp.text,f.text,fn.text],p.id,score order by score desc limit 10
``` 

- Find matching fragments in patent text

``` cypher
call db.index.fulltext.queryNodes("fragments","corona and virus") yield node as f,score match (f)--(px)--(p:Patent) return f.kind,f.text,p.id,score order by score desc limit 10
```

- This shows the previous and next fragment in the result 

```cypher
call db.index.fulltext.queryNodes("fragments","corona and virus") yield node as f,score match (f)--(px)--(p:Patent) match (fp:Fragment)-[:NEXT]->(f),(f)-[:NEXT]->(fn:Fragment) return f.kind,fp.text,f.text,fn.text,p.id,score order by score desc limit 10
```


