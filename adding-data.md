## Adding your own data to the graph
Thank you for contributing to the Covidgraph! This file provides some guidelines on adding data to the project. Feel free to use the CovidGraph Data Sources channel for any further questions.
### 1. Check which data is already in the knowledge graph
- add your dataset to the list here: https://ethercalc.org/3d1whloznieq
- If needed, add if your loading script has a dependency on other data. If you are linking your newly added data to existing nodes, check the sourceguid properties on these nodes to determine the dependencies.
 
### 2. Follow the suggested naming conventions for the nodes and relationships you create.
- See https://neo4j.com/docs/cypher-manual/current/syntax/naming/
- Node labels in camel case with an uppercase first letter (GeneSymbol)
- Relationships in all caps with underscores (HAS_BODY)
- properties in camel case (numberOfInfections)

### 3. Ensure data traceability
- Make sure that each node/relationship you import has these two properties:
- Add a unique sourceguid property to the nodes/relationships you create, so we can trace back where data comes from.
- If possible, add a sourceroutine property that refers to the script/query that imported the data. 

### 4. Think about data modeling
- Check the existing model for the graph. 
- Re-use existing node labels and relationship types if possible. If not possible, document the usecase for your node label and relationship.
- Create indexes/constraints if necessary. This will make the data faster to search, and also ensure quality (e.g. a given property must exist)
- New to graph data modeling? There’s some great material out there on which pitfalls to avoid: https://neo4j.com/blog/data-modeling-pitfalls/
