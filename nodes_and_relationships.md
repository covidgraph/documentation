# Nodes and Relationships in CovidGraph
## Format of this document
- **Nodes** are declared as `:Nodelabel` (starting with ":" colon followed by a capital letter)
- **Realtationships** are declared as `:RELATIONSHIP_NAME` (all capital letters)
- **Properties** are declared as `propertyName` (starting with small letter)
- Graph patterns are presented in Cypher query language style
-- Example 1: node1 connected to node2 via relationship,
**(node1:Nodelabel)-[relationship:RELATIONSHIP_TYPE]->(node2:Nodelabel)**
-- Example 2: node connected to itself
**(node1)-[r:RELATIONSHIP]->(node1)**


## Node labels in CovidGraph

### Literature information
- `:PubMedArticle` represents a scientific article in the PubMed database.
	- `ArticleTitle`
  - `PMID`
  - `PublicationType`
- `:ArticleId` of a PubMedArticle
	- `ArticleId.ID` 
- `:Abstract` of a PubMedArticle
- `:AbstractText` stores text of an `:Abstract` as String
	- `text` 
- `:Author` of one or many PubMedArticle
	- `ForeName`
  - `Initials`
  - `LastName`
- `:Affiliation` `Affiliation` of an authors
	- `Affiliation`
- `:Contribution` is the link between combination of author, PubMedArticle and the author's affiliations
- `:Journal` that published one or many PubMedArticle
	- `Title`
  - `ISOAbbreviation`
- `:DZDPubMedArticle` Second label on `PubMedArticle` which for DZD article
	- `ArticleTitle`
  - `PMID`
  - `PublicationType`
- `:DZDAuthor` Second label on `Author` for DZD scientists
	- `ForeName`
  - `Initials`
  - `LastName`
- `:DZDAuthorNotationsHub` standardized names for DZDAuthor connecting all 
author spellings
	- `ForeName`
  - `Initials`
  - `LastName`
- `:DZDAcademy` academy of the DZD, self explanary
	- `AcademyName`
- `:DZDInstitute` institutes of the DZD, self explanary
	- `InstituteName`
- `:DZDAffiliation` affiliations of the DZD, self explanary
	- `Affiliation`
- `:DZDContribution` Second label on `:Contribution`
- `:MeshQualifier` stores MeSH-Term
	- `text`
- `:MeshDescriptor` stores MeSH-Term
	- `text`
- `:MeshHeading` connects MeshQualifier and MeshDescriptor
- `:MeshHeadingList` list of MeSH headings
- `:Keyword` stores keyword of a PubMedArticle
	- `Keyword`
- `:KeywordList`
- `:Date` of a PubMedArticle
	- `Day`
  - `Month`
  - `Year`
- `:MedlineJournalInfo` stores the iso abbreviation of a Journal
	- `MedlineTA`
- `:Reference` of a PubMedArticle
	- `Citation`
- `:ChemicalList` list of chemicals in a PubMedArticle
- `:ReferenceList` list of references of a PubMedArticle
- `:ISSN` of a PubMedArticle
- `:JournalIssue` of a scientific journal
	- `Issue`
  - `Volume`
- `:Identifier` from `source`: ORCID, ISNI, GRID and RINGGOLD
	- `ID`
  - `Source`
- `:GrantList` list of grants of a PubMedArticle
- `:Grant` of a PubMedArticle
- `:PersonalNameSubjectList` self explanary
- `:PersonalNameSubject` PubMedArticles that are Autobiographies
- `:Investigator` PI of a PubMedArticle
- `:GeneralNote` not yet investigated
- `:Chemical` used in a PubMedArticle
### Molecular entities information
- `:Gene` from Ensembl holds information of a gene in `sid` and `name`
	- `source`
  - `sid`
  - `name`
  - `taxid`
- `:Transcript` are RNAs from coding genes, from `source`=RefSeq
	- `source`
  - `sid`
  - `taxid`
- `:Protein` from `source`=Uniprot, RefSeq, Swissprot stores protein information
	- `source`
  - `sid`
  - `taxid`
- `:Lipid` from `source`=SwissLipid
- `:Metabolite` from `source`=HNGC
	- `name`
  - `definition`
  - `sid`
  - `taxid`
  - `source`
- `:Pathway` molecular pathway from `source`=Reactome
	- `name`
  - `sid`
  - `taxid`
  - `source`
  - `org` Organism
- `:SNP` SingleNucleotidePolymorphism/gene varition from `source`=GWAS which is associated with a `:Trait` and a `:Gene`
	- `snp_id`
  - `taxid`
- `:Study` associated with a `:Trait` and a `:SNP`
	- `title`
  - `pubmedId`
  - `reported_genes`
- `:Association` between `:Trait`, a `:SNP` and `:Study`
- `:SNP_Interaction` of two or more `:SNP`s
	- `snp_id`
  - `taxid`
- `:Trait` `name`is a feature or phenotype or disease which is connected to a gene variation `:SNP`
	- `name`
- `:Phenotype` which is connected to a `:Gene`
	- `name`
- `:GtexSample` specific sample of a `:GtexTissue` with raw data
- `:GtexTissue` human tissue 
	- `name`
- `:GtexDetailedTissue` sub tissue of `:GtexTissue` and connected `:Gene`
	- `name`
- `:MgiDescription` mouse gene description from `source`=MGI
- `:GeneSymbolList` self explanatory


### Ontology information
- `:Term`	with `name` holds a term and is connected to its `:Ontology`
	- `name`
  - `definition`
  - `sid`
- `:Disease`	currently empty
- `:Ontology`	with `name`, connected to `:Term` and `source`=Obo FoundryCurrently from *Disease Ontology*, *GeneOntology*, *Mouse Phenotype Ontology*
	- `sid`
- `:OntologySubset` self explanatory
- `:SynonymTerm` 

### Technical information
- `:_FulltextLookUpDone_Label_property` Second label for node label. Combination of label and property was looked up on the fulltext index in abstracts
- `:_FulltextLookUpDone_Protein_name`
- `:_FulltextLookUpDone_Protein_sid`
- `:OmitInMatch` second label for genes, and proteins that are omitted for text search
- `:OmitSpecialChar` second label for genes and proteins because of special characters in name
- `:OmitLengthOne` second label for genes and proteins because the name is only one character
- `:OmitWord` second label for genes and proteins, because the name is an English word
- `:_PipelineLogNode` technical information to one pipeline module
- `:_PubMedXmlLoadingLog` technical information for data integration of one XML file from PubMed 
- `:_PipelineLogRun` technical information of the central pipeline run
- `:ProteinSearch` intermediate label for proteins that need preprocessing steps before search
- `:Word` list of words that are omitted in text search

