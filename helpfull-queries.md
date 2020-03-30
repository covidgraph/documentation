# Helpfull queries on the CovidGraph

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

## Patents

- Search patents with string against a textindex

```cypher
call db.index.fulltext.queryNodes("patents","Corona") yield node,score match (node)--(p:Patent)--(pt:PatentTitle)
return distinct(p.id) as id, collect(pt.text) as titles, labels(node)[0] as found_type, node.lang as found_in_lang ,score
order by score
desc limit 10
```
