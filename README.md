# data2services-insert

## Clone

```shell
git clone --recursive https://github.com/MaastrichtU-IDS/data2services-insert
```

## Build

Uses [rdf4j-sparql-operations](https://github.com/vemonet/rdf4j-sparql-operations).

```shell
docker build -t rdf4j-sparql-operations ./data2services-insert/rdf4j-sparql-operations
```

## Run

Transform datasets from generic RDF generated by [Data2Service](https://github.com/MaastrichtU-IDS/data2services-pipeline) to [BioLink](https://github.com/biolink/biolink-model) (e.g.: Drugbank or HGNC)

```shell
# Command to load in local GraphDB
docker run -it --rm --link graphdb:graphdb \
  -v "$PWD/data2services-insert/insert-biolink":/data \
  rdf4j-sparql-operations -f "/data" \
  -ep "http://graphdb:7200/repositories/test/statements" \
  -un MYUSERNAME -pw MYPASSWORD \
  -var serviceUrl:http://localhost:7200/repositories/test inputGraph:https://w3id.org/data2services/graph/input outputGraph:https://w3id.org/data2services/graph/output
```



## Execute on specific datasets

```shell
# DrugBank
docker run -it --rm -v "$PWD/insert-biolink/drugbank":/data rdf4j-sparql-operations \
	-f "/data" -un USERNAME -pw PASSWORD \
	-ep "http://graphdb.dumontierlab.com/repositories/ncats-red-kg/statements" \
    -var serviceUrl:http://localhost:7200/repositories/test inputGraph:https://w3id.org/data2services/graph/xml2rdf outputGraph:https://w3id.org/data2services/biolink/drugbank

# HGNC
docker run -it --rm -v "$PWD/insert-biolink/hgnc":/data rdf4j-sparql-operations \
	-f "/data" -un USERNAME -pw PASSWORD \
	-ep "http://graphdb.dumontierlab.com/repositories/ncats-red-kg/statements" \
	-var serviceUrl:http://localhost:7200/repositories/test inputGraph:https://w3id.org/data2services/graph/autor2rml outputGraph:https://w3id.org/data2services/biolink/hgnc
```





## RDF validation

### Simple validation with rapper

Quick validation of turtle RDF file using rapper on Unix 

```shell
sudo apt install raptor2-utils
rapper -i turtle 181107-biolink-drugbank-reified-associations.ttl
```

### PyShEx

https://github.com/hsolbrig/PyShEx

```shell
pip3 install PyShEx
shexeval -h

shexeval /home/vemonet/Desktop/181107_biolink-drugbank.jsonld /home/vemonet/sandbox/biolink-model/shex/biolink-model.shex

# Turtle
shexeval -f turtle /home/vemonet/Desktop/181121-biolink-drugbank.ttl /home/vemonet/sandbox/biolink-model/shex/biolink-model.shex
# Json-ld
shexeval -f json-ld /home/vemonet/Desktop/181107_biolink-drugbank.jsonld /home/vemonet/sandbox/biolink-model/shex/biolink-model.shex
```

### ShEx Java

http://shexjava.lille.inria.fr/

```shell
# Start the UI
java -jar shexjapp-0.0.1.jar

# Commandline
mvn exec:java -Dexec.classpathScope=test -Dexec.mainClass="fr.inria.lille.shexjava.commandLine.Validate" -Dexec.args="-s ../../shexTest/schemas/1dotSemi.shex -d file:///home/jdusart/Documents/Shex/workspace/shexTest/validation/Is1_Ip1_Io1.ttl -l http://a.example/S1 -f http://a.example/s1 -a recursive"
```

### Shex js

NOT WORKING. 40 tests fail

```shell
# Install and test
npm install --save shex
npm explore shex 'npm install'

# Tests (failing)
npm explore shex 'npm test'
```

## KGX validate

```shell
cd ~/sandbox/kgx
source env/bin/activate
kgx validate /home/vemonet/sandbox/data-constructor/resources/biogrid_test.ttl
```



## Valid BioLink

- Valid prefix for CURIE

https://github.com/monarch-initiative/dipper/blob/master/dipper/curie_map.yaml

- Spec

https://docs.google.com/document/d/1TrvqJPe_HwmRJ5HCwZ7fsi9_mwGcwLOZ53Fnjo8Sh4E/edit

Required for nodes: id, name, category



# Use SPARQL for conversion

Generating 2 types of generic SPARQL:

* From tabular (csv, tsv, RDBMS)
* From complex structure (XML, JSON, YAML)

### How we do

* For each attribute/node extracted we look for corresponding classes in BioLink Ontology

https://biolink.github.io/biolink-model/

https://raw.githubusercontent.com/biolink/biolink-model/master/ontology/biolink.ttl

https://bioportal.bioontology.org/ontologies/BLM

* TSV
  * Opening TSV file with calc and looking for concept matching columns in BioLink ontology
  * Opening  the Mapping file to get the exact URI of the column
* XML: using the schema printed by xml2rdf and the original XML file to look for matching concepts in BioLink ontology



### Limitations

* Needs to know really well the BioLink model
  * Can be hard to find some matching concepts in BioLink
  * What to do if no matching concept in BioLink?

* We can't build one big construct query. It needs to be divided in little construct queries with a focused goal
  * E.g.: getting targets infos
  * We need to avoid having too much query getting different arrays (result is a cartesian product)
* Problem: we first get all the hasChild and then filter on the type
  * **Why not putting the XPath directly in the predicate?**
* We can't split one statement value into multiple ("apple, orange" to "apple" and "orange")

### Enhancement

Also generating a generic construct query when generating the generic RDF (with AutoR2RML and xml2rdf) 

Then we "just" have to match it with the right bioentity class

* Automatically generate SPARQL query

The programs that do the generic RDF transformation should generate a file formally describing the data structure that is then used to generate the SPARQL construct query 

* Recommend class from an existing datamodel
  * Based on attribute name in the original file
  * Based on data 





# Use RML for conversion

Not used because of scalability issues. Everything is loaded in memory (at the root, they load all models in memory) https://github.com/RMLio/rmlmapper-java

- Build

```shell
mvn clean package -DskipTests
```

- Run

```shell
# HGNC
java -jar /home/vemonet/sandbox/rmlmapper-java/target/rmlmapper-4.1.0-r55-jar-with-dependencies.jar -c /data/drugbank/drugbank_config.properties

# DrugBank. Java heap space outOfMemory error (for 900M)
java -jar /home/vemonet/sandbox/rmlmapper-java/target/rmlmapper-4.0.0-r50-jar-with-dependencies.jar -c /home/vemonet/sandbox/rml_vs_sparql/rml/drugbank_config.properties
```



































