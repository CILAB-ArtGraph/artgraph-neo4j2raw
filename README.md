# Artgraph to raw
No assumption are made about the artgraph schema (this is schema agnostic). Therefore, you need to provide your queries for mapping and relations. Optionally you can add other queries for some statistics about the database content. 

For node mapping you have to name the column returned as "name". (e.g. `MATCH (f:Field) RETURN f.name as name`). <br />
For relation mapping you have to name the column returned as "rel_label" (e.g. `MATCH (n)-[r]->(n2) RETURN distinct toLower(type(r)) as rel_label`). <br />
For relation you have to name the source node column as "source_name" and the destination node column as "dest_name".

The creation of the raw data is distinct from saving on the disk. First, the raw data are created and saved in memory, on a dataframe (get_* methods). Then, all the data are saved on the disk (write_* methods).

You can generate the raw files with split (use the class ArtGraphWithSplit) or without split (use the class ArtGraphWithoutSplit). Note that you can provide a custom split specifying in a csv file the artworks you want in the train, test and validation split (each set has its file). 
See the notebooks to know how to use them.


**Remember to write the .env file with the database credentials**
e.g.
```
uri=bolt://localhost:7687
username=neo4j
password=admin
database=artgraph
```

# Main requirements
- neo4j (https://neo4j.com/docs/api/python-driver/current/)
- pandas

# Usage
See the `test` (with split) and `test-nosplit` notebooks.

# ArtGraph raw folder structure
What is ArtGraph? Check this: https://arxiv.org/abs/2105.15028

The generation of the raw files has followed the convention used by the Open Graph Benchmark.

## Mapping folder
It contains files used for mapping the node id to the node name. The following naming convention is used:
<ul>
    <li>
        <strong>&lt;node_name&gt;__entidx2name.csv</strong>, a 2 columns csv files, the first column for the node id and the second column for the correspondig name. For example, the artwork_entidx2name.csv contains rows like those:
        <br/><br/>
        <code>0, jackson-pollock_composition-white-black-blue-and-red-on-white-1948.jpg</code><br/>
        <code>1, jackson-pollock_portrait-and-a-dream-1953.jpg</code><br/>
        <code>...</code>
    </li>
    <li>
        <strong>labelidx2&lt;label&gt;name.csv</strong>, a 2 columns csv files, the first column for the label id and the second column for the correspondig name. Here &lt;label&gt; is the label node name for the classification task. For ArtGraph we have three labels: artist, style and genre.
    </li>
    <li>
        <strong>relidx2relname</strong>, a 2 columns csv files, the first column for the relation id and the second column for the correspondig name. The first rows are:
        <br/><br/>
        <code>0, influenced</code><br/>
        <code>1, subject</code><br/>
        <code>...</code>
    </li>
</ul>

### raw folder
It contains folders and files about graph-level information, relation connectivities, node labels and node features.
It contains the following files:
<ul>
    <li>
        <strong>triplet-type-list.csv</strong>, a 3 columns csv file, the first for the source node name, the second for the relation name and the third for the destination node name. The first rows are:
        <br/><br/>
        <code>artist, influenced, artist</code><br/>
        <code>artist, subject, subject</code><br/>
        <code>...</code>
    </li>
    <li>
        <strong>num-node-dict.csv</strong>, a csv file that contains the number of nodes for each node name. The first columns are:
        <br/><br/>
        <code>artist, gallery, city, country, ...</code><br/>
        <code>300, 1090, 665, 64, ...</code><br/>
    </li>
    <li>
        <strong>nodetype-has-feat.csv</strong>, a csv file that contains for each node name "True" if that node has an associated feature vector or "False" if not.
        <br/><br/>
        <code>artwork, gallery, city, country, artwork, ...</code><br/>
        <code>False, False, False,  False, True, ...</code><br/>
    </li>
    <li>
        <strong>nodetype-has-label.csv</strong>, a csv file that contains for each node name "True" if that node has an associated label (for classification task) or "False" if not. 
        <br/><br/>
        <code>artwork, gallery, city, country, artwork, ...</code><br/>
        <code>False, False, False,  False, True, ...</code><br/>
    </li>
</ul>

### relations folder
For each relation there is a folder named  &lt;source_node_name&gt;___&lt;relation__name&gt;___&lt;destination_node_name&gt;. Each folder contains three csv files:
<ul>
    <li>
        <strong>edge.csv</strong>: contains two columns: the id of the source node and the id of the destination node.
    </li>
    <li>
        <strong>edge_reltype.csv</strong>: contains the id of the relation type repeated for each relation entry. If edge.csv contains 100 rows (hence 100 relations) and we are in the folder artist__author__artwork that has label 3, there will be a csv file with 100 rows full of 3.
    </li>
    <li>
        <strong>num-edge-list.csv</strong>: a single cell that contains the number of relations of that type.
    </li>
</ul>

### node-label folder
It contains folders about the nodes for which we want to do the classification, therefore just the 'artwork' folder. Inside the folder there is a csv file for each label. Each file consists of only one column containing the the label for each artwork node. The id of the artwork node is determined by the row number. For example: row 1 idenfifies the artwork node with id 0, etc.

### node-feat folder
It contains folders about nodes that have features vectors. In our case only the 'artwork' node. Inside the 'artwork' folder there is the file <strong>node-feat.csv</strong>, a file with only X columns containing the features vector for each artwork node. For example if the size of the vector is 128 there will be 128 columns for each artwork node. The id of the artwork node will determined using the row. For example: row 1 idenfifies the node with id 0, etc.

### split folder (only when you use ArtGraphWithSplit)
Contains the files for train, test, validation splitting for the node 'artwork'. There is a file for each splitting, that is <strong>train.csv</strong>, <strong>test.csv</strong> and <strong>validation.csv</strong>. Each file consists of only one column containing the id of the artwork node. The splitting for the training and test/validation is the same used by previous work on ArtGraph.
