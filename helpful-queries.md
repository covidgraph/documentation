# helpfull-queries

[Back to CovidGraph Wiki](https://github.com/covidgraph/documentation/wiki)

## Helpful queries on the CovidGraph


⛔ This document needs to be updated to newest datamodel

### Papers

* List Fulltext papers with title

```text
MATCH (p:Paper)-[pr:PAPER_HAS_BODYTEXTCOLLECTION]->(c:BodyTextCollection)-[r:BODYTEXTCOLLECTION_HAS_BODYTEXT]->(t:BodyText)
WITH p.title as title,collect({txt:t.text, pos:r.position}) as text
UNWIND text as t
WITH title, t
order by t.pos
RETURN title, collect(t.txt) 
limit 4
```

* Get Papers and Authors

```text
MATCH (a:Author)<-[:AUTHORCOLLECTION_HAS_AUTHOR]-(:AuthorCollection)<-[:PAPER_HAS_AUTHORCOLLECTION]-(p:Paper)
RETURN a, p, apoc.create.vRelationship(a, 'AUTHORED',{}, p) as vrel 
limit 100
```

* Genes connected to papers

⛔ Not working atm

```text
MATCH (p:Paper)
MATCH (p)-[:PAPER_HAS_BODYTEXTCOLLECTION]->(:BodyTextCollection)-[:BODYTEXTCOLLECTION_HAS_BODYTEXT]->(:BodyText)-[:HAS_FRAGMENT]->(f:Fragment)-[:MENTIONS]->(g:GeneSymbol) with collect(distinct g.sid) as gsid
RETURN gsid as result order by gsid
limit 10
```

* Number of authors by location/region




```text
MATCH (loc:Location)<-[:AFFILIATION_HAS_LOCATION]-(aff:Affiliation)-[:AUTHOR_HAS_AFFILIATION]-(a:Author) 
WHERE loc.country IS NOT NULL 
RETURN loc.country as country, loc.region as region, count(distinct a.email) AS NbrOfAuthors 
ORDER BY count(distinct a.email) DESC
```

* Titles and dates of papers whose body text contains a user-specified keyword \(e.g. Virus\), ordered by date of publication. 

⛔ Not working atm

```text
MATCH (p:Paper)-[:PAPER_HAS_BODY_TEXT]->(b:Body_text)  
WHERE p.title IS NOT NULL AND p.title CONTAINS("Virus") 
RETURN p.title, p.publish_time 
ORDER BY p.publish_time DESC 
LIMIT 20
```

* Number of papers whose body text contains a user-specified keyword \(e.g. Virus\).


⛔ Not working atm

```text
MATCH (p:Paper)-[:PAPER_HAS_BODY_TEXT]->(b:Body_text)  
WHERE p.title IS NOT NULL AND p.title CONTAINS("Virus") 
RETURN count(p)
LIMIT 20
```

### Patents

* Find genes and proteins that are mentioned in patents


⛔ Not working atm

```text
match path=(e:Entity)<-[x:APPLICANT]-(p:Patent)-[y:HAS_CLAIM|:HAS_ABSTRACT|:HAS_TITLE]->(pa)-[z:HAS_FRAGMENT]->(ff:Fragment)-[m:MENTIONS]->(syn:GeneSymbol)-[:SYNONYM]->(gs:GeneSymbol)<-[:MAPS]-(g:Gene)-[:CODES]->(tc:Transcript)-[:CODES]->(pro:Protein)
where e.idLower starts with $company and exists(pro.name)
return path limit 100
```

* Does company xyz work on protein xxx?


⛔ Not working atm

```text
match path=(e:Entity)<-[x:APPLICANT]-(p:Patent)-[y:HAS_CLAIM|:HAS_ABSTRACT|:HAS_TITLE]->(pa)-[z:HAS_FRAGMENT]->(ff:Fragment)-[m:MENTIONS]->(syn:GeneSymbol)-[:SYNONYM]->(gs:GeneSymbol)<-[:MAPS]-(g:Gene)-[:CODES]->(tc:Transcript)-[:CODES]->(pro:Protein)
where e.idLower starts with $company and pro.name contains $protein
return path limit 40
```

* Find gene names mentioned in patents

```text
match (p:Patent)-[x:HAS_TITLE|:HAS_CLAIM|:HAS_TITLE]-(pct)-[:HAS_FRAGMENT]->(f2:Fragment)-[:MENTIONS]->(gs2:GeneSymbol) return p,x,pct,gs2 limit 300
```

* Search patents with string against a textindex and get a hit score

```text
call db.index.fulltext.queryNodes("patents","Corona") 
yield node,score match (node)--(p:Patent)--(pt:PatentTitle)
return distinct(p.id) as id, collect(pt.text) as titles, labels(node)[0] as found_type, node.lang as found_in_lang ,score
order by score
desc limit 10
```

```text
call db.index.fulltext.queryNodes("fragments","corona and virus") 
yield node as f,score match (f)--(px)--(p:Patent) 
match (fp:Fragment)-[:NEXT]->(f),(f)-[:NEXT]->(fn:Fragment) 
return f.kind,[fp.text,f.text,fn.text],p.id,score 
order by score desc 
limit 10
```

* Find matching fragments in patent text

```text
call db.index.fulltext.queryNodes("fragments","corona and virus") 
yield node as f,score match (f)--(px)--(p:Patent) 
return f.kind,f.text,p.id,score 
order by score desc 
limit 10
```

* This shows the previous and next fragment in the result 

```text
call db.index.fulltext.queryNodes("fragments","corona and virus") 
yield node as f,score match (f)--(px)--(p:Patent) 
match (fp:Fragment)-[:NEXT]->(f),(f)-[:NEXT]->(fn:Fragment) 
return f.kind,fp.text,f.text,fn.text,p.id,score 
order by score desc 
limit 10
```

### Authors

* Ranking authors. First create a projection on the graph, then call the PageRank algorithm:

```text
CALL gds.graph.create.cypher(
    'Authors_Influence',
    'MATCH (n:Author) RETURN id(n) AS id',
    'MATCH (a:Author)-[:AUTHOR_HAS_AUTHOR]->(b:Author) RETURN id(a) AS source, id(b) AS target'
)
YIELD graphName, nodeCount, relationshipCount, createMillis;
```

```text
CALL gds.pageRank.stream('Authors_Influence') YIELD nodeId, score RETURN gds.util.asNode(nodeId).first, gds.util.asNode(nodeId).last, score ORDER BY score DESC
```

