# Network analysis in Jupyter notebook

##

## Network analysis with kiara

Now that you are comfortable with what kiara looks like and what it can do to help track your data and your research process, let's try out some of the digital analysis tools, starting with **network analysis**.

## Why network analysis?

Network analysis offers a computational and quantitative means to examine and explore **relational objects**, with proxies to **measure** structural roles and concepts such as power and influence.&#x20;

Doing this digitally, and **at scale**, enables you to ask questions of large datasets that would be impossible to answer qualitatively.

This tutorial is not a deep dive into network theory or its applications in the humanities. Rather, you will focus on how using kiara helps you structure your workflow, document your decisions, and trace the transformations your data undergoes during analysis.

## Get started

Before you begin, you will check that the necessary plugins are available and set up the `kiaraAPI`, the interface that allows us to run kiara commands inside your Jupyter notebook.

Here is the code to get started:

```
import networkx as nx
import os
from kiara.api import KiaraAPI
kiara = KiaraAPI.instance()
```

## Set up your data path

Next, you need to set the **file path** to the data you will use in this notebook.&#x20;

In this example, the CSV file is stored in the same directory as the notebook, inside a subfolder named `data`.&#x20;

You can either write out the full file path, or use Python's `os.path` module to construct it, as below:&#x20;

```
notebook_path = os.path.abspath('')

csv_file_path = os.path.join(notebook_path,"data/CKCC.csv")
```

## About the dataset

The dataset you are working with comes from the _Circulation of Knowledge and Learned Practices in the 17th-century Dutch Republic (CKCC)_ collection.

This corpus of around 20,000 letters exchanged between scholars in the Dutch Republic during the 17th century. It was compiled by the Huygens Institute in the Netherlands and is available through the LetterSampo portal as part of the _Reassembling the Republic of Letters_ project.&#x20;

By applying network analysis, you can explore questions such as:

* Who was the most prolific writer?
* Which actor connected the most people?
* Who operated in closely knit writing groups?

Although network analysis can be used to explore unfamiliar datasets, in this case, you already have some prior knowledge of the material. The research questions and parameter choices in this notebook reflect that prior knowledge — something to keep in mind as you proceed.

## Import data

Now you will import the data using kiara's `import.local.file` operation.&#x20;

This module allows you to import any local file - in this case, your CSV file. You are using a sample dataset here, but you can also use this same module to import your own data.

You will pass the file path you defined earlier as an input. You will also save the output of this operation under the alias 'CKCC'. Alternatively, you can use the `download.file` module mentioned above.

You can leave the `comment` field empty for now — or you might use it to note why you chose this dataset, or which version you are working with (if you have multiple versions of the same data).

Here is the code:

```
CKCC = kiara.run_job('import.local.file', inputs={'path': csv_file_path}, comment="importing bits")
```

## Create a network

Now that you have imported your data, it’s time to build a **network** from it.&#x20;

To create a network, kiara expects your data to be in the form of an **edge table**. An edge table lists relationships between entities — in this case, between senders and recipients of letters. Later, you could also add a **node table** (a table listing all the individual entities), but this is optional — you will skip it for now.

## Convert your CSV into an edge table

To turn your CSV file into an edge table, you will use the `create.table.from.file` function that you used earlier.&#x20;

You will first check what inputs this function requires:

```
kiara.retrieve_operation_info('create.table.from.file')
```

This will display helpful information about what inputs the function expects and what it returns.

Now, you can load your CSV data (which you imported earlier and stored in `CKCC`) and tell kiara that the first row contains column headers:

```
inputs = {
    "file": CKCC['file'],
    "first_row_is_header": True
}

outputs = kiara.run_job('create.table.from.file', inputs=inputs, comment="")

edges = outputs['table']

outputs
```

At this point, you have created your **edges table**, which will serve as the basis for your network graph.

## Preview the network structure&#x20;

Before you assemble your network graph, it is helpful to preview its structure.

You can do this using kiara’s `preview.network_info` function.

This function will show you:

* How many nodes and edges the network contains
* How the graph structure might change depending on whether you choose **directed**, **undirected**, **multi-directed**, or **multi-undirected** representations

To run the preview, specify your edges table, along with the source and target columns:

```
inputs = {'edges': edges,
    'source_column': 'Source',
    'target_column': 'Target'}

network_info = kiara.run_job('preview.network_info', inputs=inputs, comment="")

network_info
```

This preview will give you useful insights:

* If there are more edges in a **directed** graph than in an **undirected** one, it suggests that some nodes are reciprocally connected, as is common in letter networks.
* If a **multigraph** has even more edges, this means that **parallel edges** exist between the same pairs of nodes (for example, one person sending several letters to another).
* If there are **no isolates** (nodes without any edges) and several **components**.
* If you see a large number of **self-loops** (nodes connected to themselves), this is unusual in epistolarly collections and could indicate an issue with the data, for example, missing or misformatted recipient information.&#x20;

This module helps you decide what type of graph is appropriate for your dataset, and also alerts you to any potential data quality issues you may want to address.

In this case, a **directed graph** makes sense, since letters are sent from one person to another.

## Review the assemble network graph module

Next, you will use the `assemble.network_graph` module to actually build your graph.

Before you run it, it’s helpful to check its inputs:

```
kiara.retrieve_operation_info('assemble.network_graph')
```

This is a flexible module that allows you to make several important decisions:

* The **type of graph** you want to create (directed or undirected)
* Whether the graph should be **weighted** or not
* How to handle **parallel edges**
* If you have a **node table**, you can include it — but this is optional

If you later decide to change your mind about how to structure your graph, you can simply re-run this step.&#x20;

That’s why previewing the network first is so helpful — it gives you the information you need to make an informed decision.

## Assemble the network graph

Now that you have reviewed the structure of your data, you are ready to assemble the graph.

For this dataset, you will create a **directed graph**.

You will also choose to create a **weighted graph**, where the weight of each edge reflects the **number of letters** sent between two people.

If the dataset contains parallel edges (which the preview revealed), you can choose how to aggregate them. In this case, you will choose `'sum'` — which will count how many times each edge occurs.

Keep in mind that if you have not provided any weights in the data, kiara will automatically assign a weight of 1 to each edge. In that case, selecting `mean` , `minimum`, or `maximum` will simply return 1 for every edge, making the result the same as an **unweighted** network.

Here is the full set of inputs:

```
inputs = {
    'graph_type': 'directed',
    'edges': edges,
    'source_column': 'Source',
    'target_column': 'Target',
    'is_weighted': True,
    'parallel_edge_strategy':'sum'
}

CKCC = kiara.run_job('assemble.network_graph', inputs=inputs, comment="")
CKCC
```

Now you have created a kiara **network graph object**.&#x20;

The output includes both an **edge table** and a **node table**. Since you did not provide a separate node table, kiara automatically extracted node information from the edges.

If you look at the edge table now, you will see that it includes **weights**, calculated based on the number of letters exchanged.

## Extract the largest component

The preview you ran earlier showed that the network has eight **components** (disconnected subgraphs).&#x20;

You may want to focus your analysis on the **largest component** — the one with the most nodes — since it often contains the most interesting or well-connected part of the network.

You can extract the largest component like this:

```
CKCC_largest_component = kiara.run_job('extract.largest_component', inputs={'network_graph':CKCC['network_graph']}, comment="")

CKCC_largest_component
```

Now you have a new graph containing just the **largest component** of your network, ready for further analysis.

## Structural measures

Now that you have extracted the largest component of your network, you can calculate some basic **structural measures**: the **diameter** and the **average path length**.

These calculations require the graph to be **connected**, which is why it’s important that you are working with the largest component.

## Calculate the diameter

The **diameter** tells you the length of the longest shortest path between any two nodes in the network, in this case, 7 steps.

Here is how you calculate it:

```

diameter = kiara.run_job('calculate.diameter', inputs={'network_component':CKCC_largest_component['largest_component']}, comment='c')
diameter
```

## Calculate the average path length

The **average path length** gives you the average number of steps it takes to get from one node to another, about 2.7 in this case.

Here is how you calculate it:

```
avg_path = kiara.run_job('calculate.average_path', inputs={'network_component':CKCC_largest_component['largest_component']}, comment='c')
avg_path
```

These two metrics give us an idea of the overall structure of the network.

## Statistical measures

Now that you have your network assembled, you can start exploring **centrality measures** — ways of assessing how important or influential each node is in the network.

## Degree centrality

You will start by calculating **degree centrality** using kiara’s `calculate.degree_score` module.

Before running it, check what inputs the module requires:

```
kiara.retrieve_operation_info('calculate.degree_score')
```

Now you can run the calculation:

```
output = kiara.run_job('calculate.degree_score', inputs={'network_graph':CKCC['network_graph']}, comment="")
output
```

This will return a table of **degree scores**:

* **Undirected degree**: number of correspondents
* **Weighted degree**: number of letters sent and received

kiara also adds these degree scores as **node attributes**, so you can use them in later analyses.

## Betweenness centrality

Next, you will calculate **betweenness centrality**.

Again, first check what inputs the module requires:

```
kiara.retrieve_operation_info('calculate.betweenness_score')
```

This module asks you to define whether **weights** should be interpreted as **positive** (indicating strength) or **negative** (indicating distance or cost).

For the epistolary dataset, you will leave this input as `'True'` , as the weight indicates strength (number of letters).

Before running the calculation, save the current network (with degree scores attached):

```
network_graph = output['centrality_network']
```

Now calculate betweenness:

<pre><code><strong>output = kiara.run_job('calculate.betweenness_score', inputs={'network_graph':network_graph}, comment="")
</strong>
output
</code></pre>

As before, kiara will return a table of **betweenness scores**, and also add these scores to the **node attributes**.

## Eigenvector centrality

You can now calculate **eigenvector centrality**.

First, check the module:

```
kiara.retrieve_operation_info('calculate.eigenvector_score')
```

You will use your updated network again:

```
network_graph = output['centrality_network']

output = kiara.run_job('calculate.eigenvector_score', inputs={'network_graph':network_graph}, comment="")

output
```

Once again, kiara will return a table of scores and update the node attributes.

## Closeness centrality

kiara also includes a module to calculate **closeness centrality**.

You can try this on your own:

1. Use `retrieve_operation_info` to check the module.
2. Run it on your current network.

## Modularity Groups

Next, you will calculate **modularity groups** — clusters of nodes that are more tightly connected to each other.

First, check the module:

```
kiara.retrieve_operation_info('compute.modularity_group')
```

You can either set the number of communities manually or let kiara detect them.

Now run the module:

```
network_graph = output['centrality_network']

output = kiara.run_job('compute.modularity_group', inputs={'network_graph':network_graph, 'number_of_communities':10}, comment="")

output
```

kiara will update the node attributes with community group numbers.

## Cut points

Finally, you can identify **cut points** — nodes whose removal would break the network into separate parts.

First, check the module:

```
kiara.retrieve_operation_info('create.cut_point_list')
```

Now run it:

```
network_graph = output['modularity_network']

output = kiara.run_job('create.cut_point_list', inputs={'network_graph':network_graph}, comment="")

output
```

kiara will return:

* A list of cut points
* An updated node table with a `'Cut Point'` attribute (`Yes` or `No`)

**Note:** The `cut_point_list` function in **NetworkX** (which kiara uses internally) does not support **directed** or **multi-directed** graphs. If you are working with one of these graph types, kiara will automatically convert your graph to an **undirected version** just for this calculation. The results are then returned in your original directed graph. This does not affect the correctness of the results — but it’s useful to be aware of this behind-the-scenes step.

## Export the network

Now that your network is fully analyzed, you can export it for use in other tools (for visualization or further analysis).

Check the export module:&#x20;

```
kiara.retrieve_operation_info('export.network_graph')
```

You can export your network as:

* CSV
* GraphML
* GEXF
* and other formats

## Check the lineage

Finally, you can check the **lineage** of your entire network workflow — to see how every step was documented:

```
lineage = kiara.retrieve_augmented_value_lineage(output['cut_network'])
from observable_jupyter import embed
embed('@dharpa-project/kiara-data-lineage', cells=['displayViz', 'style'], inputs={'dataset':lineage})
```

## Importing other data formats

So far, you have created a network from a **CSV**.&#x20;

But you can also import networks from formats like **GML**.

You will now import a [co-appearance network](http://www-personal.umich.edu/~mejn/netdata/) of characters from Victor Hugo's novel, _Les Misérables_ (in GML format).

First, check the module:

```
kiara.retrieve_operation_info('import.network_graph.from.file')
```

For this module, you only need to provide:

* the path to the file
* the file type (`'gml'`)

If the **node labels** in your GML file are named something other than `'id'`, you can specify that using the `label` input.&#x20;

If the **weight column** in your file has a different name (for example, `'value'`), you can tell kiara to rename it to `'weight'`.

Now import the GML file:

```
lesmis_path = os.path.join(notebook_path,"data/lesmis.gml")

lesmis = kiara.run_job('import.network_graph.from.file', inputs={'path': lesmis_path, 'file_type':'gml', 'weight_column':'value'}, comment="")
lesmis
```

kiara imports the file and automatically converts it into a **network graph object**.&#x20;

**Note**: The **edge table** now uses the column name `'weight'`, which was automatically updated from `'value'` to match kiara’s expected format for edge weights.

You can now run analyses just as you did before. For example:

```
output = kiara.run_job('calculate.degree_score', inputs={'network_graph':lesmis['network_graph']}, comment="")
output
```

This is a great way to work with published networks or networks from other tools — and you can use all the same kiara operations on them.
