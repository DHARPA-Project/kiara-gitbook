# Network analysis in jupyter notebook

Before you run any notebook, you will need to install kiara on your computer.&#x20;

This process is quite manageable, and I will walk you through it step by step.

You will need:

* a tool called **conda** or **miniconda** for managing Python and dependencies
* a special **environment** where kiara will live
* the **kiara** software itself, plus a few plugins

## Install conda (or miniconda)

**Conda** is a popular tool that helps you manage different versions of Python and software libraries. If you already have Conda (or Miniconda) installed, you can skip this step.

If not:

* Go to the Miniconda [download page](https://www.anaconda.com/docs/getting-started/miniconda/main).
* Download the version that matches your computer (Windows, macOS, or Linux).
* Follow the instructions to install it.

\*Double-check that you choose the correct version for your operating system and architecture (most modern computers use the 64-bit version).

## Create a kiara environment

Once Conda is installed, open your **Terminal** (on Mac or Linux) or **Command Line Interface (CLI)** (on Windows).

You are now going to create a special **environment** just for Kiara. This is like giving Kiara its own room, so that it will not interfere with other tools or projects on your computer.

Type this command:

```
conda create -n kiara_testing python jupyter
```

This creates an environment called `kiara_testing` and installs a recent version of Python and **Jupyter Notebook** (the tool you will use to run the notebooks).

You can replace `kiara_testing` with any name you like for your environment.

## Activate the envrionment

Next, tell Conda that you want to activate this new environment:

```
conda activate kiara_testing
```

## Install kiara

Now that your environment is set up, let's install **kiara** iteself.&#x20;

kiara is installed using a package-management system for Python called `pip`. Just type:

```
pip install kiara
```

The first installation may take a few minutes. That is normal.

## Install kiara plugins

To make kiara more useful, you will also install some basic **plugins** by running:&#x20;

```
pip install kiara_plugin.core_types kiara_plugin.onboarding kiara_plugin.tabular pip install observable_jupyter
```

These plugins provide support for:

* core data types
* helpful onboarding tools
* tabular data (spreadsheets, CSVs, etc.)

To see which versions of kiara and its plugins are installed, you can run:

```
pip list | grep kiara
```

At the time of writing, the versions installed are:

```
kiara                     0.5.13
kiara_plugin.core_types   0.5.2
kiara_plugin.onboarding   0.5.2
kiara_plugin.tabular      0.5.6
```

## Import kiara and create an API

Now that kiara and its plugins are installed, let's set up `kiaraAPI`.

To start using kiara in Jupyter Notebook, you first need to create a `kiaraAPI` **instance**. This instance allows us to control kiara and see what operations are available.&#x20;

To set this up, run the following code in a notebook cell:

```
from kiara.api import KiaraAPI

kiara = KiaraAPI.instance()
```

## Create a project context

In kiara, a **context** is your project space. It stores:

* the data you import or create
* the operations you run
* your notes and decisions

A default context is always available, but you can also create your own for specific projects.&#x20;

To create and use a **new context** called `hello_kiara` , run the following code:

```
kiara.set_active_context(context_name='hello_kiara', create=True)

print('Available Contexts:', kiara.list_context_names())
print('Current Context:', kiara.get_current_context_name())
```

This operation will also show all your available contexts and confirm which one is currently active.

The output will be something like:&#x20;

```
Available Contexts: ['default', 'hello_kiara']
Current Context: hello_kiara
```

This confirms that your new context is set up and ready to use.&#x20;

## Explore kiara operations

Now, you can explore the tools kiara offers. To view a list of all available **operations** (based on the installed plugins), run:&#x20;

```
kiara.list_operation_ids()
```

This will return a list of operations, like:

```
['assemble.network_graph',
 'assemble.tables',
 'calculate.betweenness_score',
 'calculate.closeness_score',
 'calculate.degree_score',
 'calculate.eigenvector_score',
 'compute.modularity_group',
 'create.cut_point_list',
 'create.database.from.file',
 'create.database.from.file_bundle',
 'create.database.from.table',
 'create.database.from.tables',
 'create.job_log',
 'create.network_graph.from.file',
 'create.table.from.file'
 ...]
```

Each **operation** is a task you can perform in kiara, such as creating a table, calculating network metrics, or exporting files.&#x20;

## Download a file&#x20;

Now that kiara is set up, let's bring a file into our notebook using the `download.file` operation. &#x20;

To understand what this operation does and what information it needs, run:

```
kiara.retrieve_operation_info('download.file')
```

You will now download a sample CSV file using this operation. First, you define the **input** (the file URL and name) and then run the job:

```
inputs = {
        "url": "https://raw.githubusercontent.com/DHARPA-Project/kiara.examples/main/examples/data/network_analysis/journals/JournalNodes1902.csv",
        "file_name": "JournalNodes1902.csv"
}

outputs = kiara.run_job('download.file', inputs=inputs, comment="importing journal nodes")
```

This gives you a **file object** as **output**, including the downloaded file and some technical metadata. Let’s print it to confirm:

```
outputs
```

You will see a preview of the file's content. This shows the journal data was successfully downloaded.&#x20;

## Save the downloaded file

To keep using this file later (even if the notebook is closed), you will save it inside kiara using an **alias**. This works like giving a name that kiara remembers.&#x20;

```
downloaded_file = outputs['file']
kiara.store_value(value=downloaded_file.value_id, alias='Journal_Nodes')
```

Now, `Journal_Nodes` is saved in kiara's internal storage. You can refer to it later just by its alias.

## Convert the file into a table

Now that you have downloaded the file, let's turn it into a **table** so you can work with the data.&#x20;

You can look through kiara’s available operations by filtering for those that start with `create`:

```
kiara.list_operation_ids('create')
```

This shows a list of operations. Since you are working with a CSV file, the one you want is `create.table.from.file` .

This operation will read the file and turn it into a **structured table**.

To see what inputs and outputs this operation expects, run:

```
op_id = 'create.table.from.file'

kiara.retrieve_operation_info(op_id)
```

From this, you learn the inputs and outputs:

**Inputs**

* Required: a file.
* Optional:
  * `first_row_is_header` – indicates if the first row of a CSV file contains column headers.
  * `delimiter` – specifies the column separator (only for CSV), used if kiara cannot auto-detect it.

**Outputs**

* A `table` object, which can be used in the next steps.

Let’s turn the downloaded file (which you saved earlier under the alias `Journal_Nodes`) into a **table**:

```
inputs = {
    "file": kiara.get_value('Journal_Nodes'),
    "first_row_is_header": True
}

outputs = kiara.run_job(op_id, inputs=inputs, comment="")

outputs
```

This will process the CSV file and show the result as a **table** with **columns** and **rows**.

## Save the table

To make it easier to reuse the table later, you can save it in kiara under a **new alias**:

```
outputs_table = outputs['table']

kiara.store_value(value=outputs_table.value_id, alias="Journal_Nodes_table")
```

Now, your data is saved inside kiara and can be accessed at any time using the name `Journal_Nodes_table`.

## Query the data

Now that you have downloaded the file and converted it into a table, you can start exploring the data. One simple way to do that is by running **SQL queries** directly on the table using κiara.

To find relevant operations for querying data, search with the keyword `'query'`:

```
kiara.list_operation_ids('query')
```

This returns:

```
['query.database', 'query.table']
```

Since you are working with a table, you will use:

```
kiara.retrieve_operation_info('query.table')
```

This tells us that `query.table` allows us to write an **SQL query** to explore the data.&#x20;

The required **inputs** are:

* `table`: the data you want to query
* `query`: your SQL statement

Let’s find out how many of these journals were published in Berlin:

```
inputs = {
    "table": kiara.get_value('Journal_Nodes_table'),
    "query": "SELECT * from data where City like 'Berlin'"
}

outputs = kiara.run_job('query.table', inputs=inputs, comment="")

outputs
```

The result (in `outputs['query_result']`) is a filtered table showing only journals published in Berlin.

## Refine the query

Let's narrow this further and find all the journals that are about general medicine and published in Berlin.

You can re-use the `query.table` function and the table you have just made, stored in `outputs['query_result']`

```
inputs = {
    "table" : outputs['query_result'],
    "query" : "SELECT * from data where JournalType like 'general medicine'"
}

outputs = kiara.run_job('query.table', inputs=inputs, comment="")
outputs
```

This returns a smaller table with only the Berlin-based general medicine journals.

## Record and trace your data

Now that you have transformed and queried our data, let's review what κiara knows about the outputs you have created and how it tracks changes. **Data lineage** is one of kiara’s most powerful features.&#x20;

Let’s check the lineage of our query output:

```
query_output = outputs['query_result']
query_output
```

Even though you have made changes along the way, you can still access a lot of information about our data.&#x20;

kiara automatically traces all of these changes, **keeping track of** **inputs** and **outputs** and assigning each a unique identifier, so you always know exactly what has happened to your data.&#x20;

To have a 'backstage' view of how your data was transformed, including the inputs for each function you have run and how they connect, run the following:

```
query_output.lineage
```

Each input is assigned a unique ID, allowing complete transparency and traceability.

You can also **visualize** the lineage by running:

```
lineage = kiara.retrieve_augmented_value_lineage(query_output)
from observable_jupyter import embed
embed('@dharpa-project/kiara-data-lineage', cells=['displayViz', 'style'], inputs={'dataset':lineage})
```

## Review and export all jobs

kiara keeps a **record** of all operations you’ve run in this context.

You can print out this history:

```
kiara.print_all_jobs_info_data(show_inputs=True, show_outputs=True, max_char=100)
```

Finally, you can **export** your job log to a CSV file to keep a full record of what you’ve done:

```
import pandas as pd
job_table = pd.DataFrame(kiara.get_all_jobs_info_data(add_inputs_preview=True, add_outputs_preview=True))
job_table.to_csv('job_log.csv', index=False)
```

## Network analysis with kiara

Now that you are comfortable with what kiara looks like and what it can do to help track your data and your research process, let's try out some of the digital analysis tools, starting with **network analysis**.

## Why network analysis?

Network analysis offers a **computational** and **quantitative** means to examine and explore **relational objects**, with proxies to **measure** **structural roles** and **concepts** such as power and influence. Doing so digitally - and **at scale** - also allows us to consider these kinds of questions with large amounts of material or documents that were not heretofore manageable with qualitative or manual approaches.

You will not get into any core network theories or their uses in the humanities here, as you are focused on the ways in which network analysis in kiara offers an interesting way to wrap the research process, and think about the decisions you are making and how to trace them. If you're interested in learning more about network analysis, or how to code using [NetworkX](https://networkx.org/), the library currently used in these kiara modules, check out our recommended reading at the bottom.

## Get started

Before you begin exploring **network analysis**, let's make sure everything is ready.

In this step, you will check that the necessary **plugins** are available and set up the `kiaraAPI`, which is the interface that allows us to run Kiara commands inside our Jupyter notebook.

This is the code to get started:

```
import networkx as nx
import os
from kiara.api import KiaraAPI
kiara = KiaraAPI.instance()
```

## Import data

Next, you will set up the **filepaths** for the data that you are going to use in this notebook. The data file is stored in the same directory as the two Jupyter notebooks. To set the file path, you can either save the full path to the CSV file in the variable below, or use the `os.path` modules in Python to shorten this, as below:&#x20;

```
notebook_path = os.path.abspath('')

csv_file_path = os.path.join(notebook_path,"data/CKCC.csv")
```

Great, you are all set up. You are now going to import some data again using the kiara function `import.local.file`. This function will allow you to bring in a local file, one stored on your computer. You are using sample data, but you can also use this function to import your own data.

The dataset you are using is a sample from the **Circulation of Knowledge and Learned Practices in the 17th-century Dutch Republic (CKCC)** collection, compiled by the Huygens Institute in the Netherlands and made available on the LetterSampo portal, part of the Reassembling the Republic of Letters project. You can find more information about these projects [here](https://seco.cs.aalto.fi/projects/rrl/).

This collection includes about 20,000 letters written by and to 17th-century scholars in the Dutch Republic. By using network analysis, you can explore questions such as:

* Who was the most prolific writer?
* Which actor connected the most people?
* Who operated in closely knit writing groups?

While network analysis can be used to explore and map unknown datasets, in this case, you already know something about the data. The research questions and module parameters in this notebook have been shaped by that prior knowledge. That is important to keep in mind as you proceed.

Let’s now use the `import.local.file` module from kiara to access our **CSV file**. You will specify the path to the CSV file in our inputs and save the outputs of the function as '**CKCC**'. Alternatively, you can use the `download.file` module used in the **Hello Kiara** notebook.

You will leave the comments blank here for you to fill in yourself, but the comment here might indicate why you have chosen this dataset, or a reminder of which version you are working with if you have multiple versions of the same dataset.

```
CKCC = kiara.run_job('import.local.file', inputs={'path': csv_file_path}, comment="importing bits")
```

## Create a network

Now that you have imported our data, it’s time to build a **network** from it.&#x20;

As with most network analysis tools, kiara requires the data to be in the form of an **edge** **table** first. An edge table shows the connections or relationships between different entities, in this case, between senders and recipients of letters. Later, you could also add a **node table** (the individual entities), but that is optional, and you will skip it for now.

To transform our CSV file into an edge table, you will use the `create.table.from.file` function that you used in the first notebook. You will save this table in a variable called **CKCC**, which you will reuse later on.

Before running it, you should check the input requirements of the function, just to make sure you are using it correctly. You can do that with the following command:

```
kiara.retrieve_operation_info('create.table.from.file')
```

This will display useful information about the function, such as the **inputs** it needs and the **outputs** it produces.

Now, you can turn our CSV data into a kiara table by loading the data file you imported earlier (the one stored in `CKCC`) and telling Kiara that the first row of the file contains the column headers. You can do that with the following command:

```
inputs = {
    "file": CKCC['file'],
    "first_row_is_header": True
}

outputs = kiara.run_job('create.table.from.file', inputs=inputs, comment="")

edges = outputs['table']

outputs
```

## Preview the network structure <mark style="color:red;">(This doesn't work. There is no</mark> `preview.network_info`<mark style="color:red;">)</mark>

Now that you have our edges formatted as a kiara table, you are ready to make our **network graph**. But before you do that, it is helpful to preview the structure of the network using kiara’s `preview.network_info` function. All you need to do is select our **edges table** and the column names for our **sources** and **targets** by running:

```
inputs = {'edges': edges,
    'source_column': 'Source',
    'target_column': 'Target'}

network_info = kiara.run_job('preview.network_info', inputs=inputs, comment="")

network_info
```

This function gives us the total number of **nodes**, but it also helps us think about how different types of graphs  - **directed**, **undirected**, **multi-directed**, and **multi-undirected** - might affect the number of **edges** in the network.

You see that there are more edges in a **directed** graph than in an **undirected** graph. This suggests that there are reciprocal or directed edges between a pair of nodes, something typical in an **epistolary network**, where people are writing back and forth to each other.&#x20;

You also notice that there are even more edges in a **multigraph** than in either of our non-multigraphs, which means the dataset includes **parallel edges** (i.e., duplicates in our edge table). Again, this is common for an epistolary network, where someone writes more than one letter to their friend.

The preview shows **no isolates** (nodes without any edges) and several **components**. However, you see a large number of **self-loops**. This is unusual in epistolarly collections, as people are unlikely to write to themselves.&#x20;

So, in addition to helping us decide what graph type is most useful for our dataset, this module helps us **review our data** by flagging potential errors or inconsistencies in our dataset that you may want to revisit later.

Having access to this overview means you can make more informed decisions about the next steps of our research or digital analysis, especially those that are sometimes automated for us.

For our network, a **directed** graph makes the most sense.&#x20;

Let's now look at what you need to build one with our `assemble.network_graph` module using `kiara.retrieve_operation_info`.

```
kiara.retrieve_operation_info('assemble.network_graph')
```

## Graph decisions

This might seem like a chunky module, but it is doing a lot of important work up front. By making these decisions now, you will not have to make them later on when you move on to the more analytical parts.&#x20;

If you change our mind later about the kind of graph you want to use, you can always come back and rerun this step. This is why the `preview.network_info` function is very useful. It allows us to make an informed decision about our network early on.

You have already decided that you want to make a **directed** network, so you will select 'directed' for graph type. You created our edge table earlier and saved it as 'edges', so you can load that back in. You also need to specify our Source and Target columns again, and you can copy all this information from our preview module.&#x20;

You do not have a node table for this dataset, but if you did, this would be the place to include it.

Now, you can make some more decisions. One of the most important is deciding whether our network is **weighted** or **unweighted.** This can mean different things depending on your data, such as the number of letters between correspondents, the distance between them, or the duration of their relationship. If all the relationships between nodes in your network are the same, you can set `is_weighted` to `False`.  But if not, you need to tell kiara where this weight information is coming from.

If weights already exist in the edges table, for example, if you have already assigned weights to the network before uploading the data into kiara, then you can just select the weight column. In case your graph contains parallel edges, which you would already know exist from the `preview.network_info` , and you prefer not to use a multigraph, you can specify how to handle these weighted parallel edges. You can choose to aggregate the weights of the parallel edges by summing them `sum` . calculating their average weight `mean` , finding the highest value `maximum` , or the lowest `minimum`. Each of these choices will assign a single value as the new weight for that edge in the network.&#x20;

If you want kiara to calculate the weights for you, you can choose `sum` , which will count the total number of times each edge appears and use that as the weight. Keep in mind that if you have not provided any weights in the data, kiara will automatically assign a weight of 1 to each edge. In that case, selecting `mean` , `minimum`, or `maximum` will simply return 1 for every edge, making the result the same as an **unweighted** network.

The inputs for this module encourage us to reflect on the decisions you are making as you go, and think about how our data fits into these kinds of measurements. By working through these steps in kiara, you are not only making choices but also **documenting** them. Kiara tracks these decisions, both through the way the process is stored and through the comments you can include along the way.

Since you are still working with our letter dataset, you will ask kiara to add all the edges together so that the weight will tell us how many letters each person wrote to each other.

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

Great, this has created a kiara **network graph object**. The output includes both an **edge table** and a **node table** for the network.

Since you did not provide a separate node table, kiara has automatically extracted the node information from the edges. If you look at the edge table now, you will see that it includes **weights**, calculated according to the decision you just made.&#x20;

As you saw earlier in the `preview.network_info` module, the network consists of eight separate components. You might want to focus on just the **main component**, which includes the most nodes. To do this, you can extract the largest component and return it as its own network graph.&#x20;

```
CKCC_largest_component = kiara.run_job('extract.largest_component', inputs={'network_graph':CKCC['network_graph']}, comment="")

CKCC_largest_component
```

## Structural measures

Now that you have extracted the largest component of the network, you can calculate some basic structural measures: the **average path length** and the **diameter** of the network.

These kinds of calculations only work on connected graphs (i.e., networks with only one component) or if you have extracted the largest component already.

Let’s take a look at the **diameter**:

```

diameter = kiara.run_job('calculate.diameter', inputs={'network_component':CKCC_largest_component['largest_component']}, comment='c')
diameter
```

And now for the **average path length**:

```
avg_path = kiara.run_job('calculate.average_path', inputs={'network_component':CKCC_largest_component['largest_component']}, comment='c')
avg_path
```

These two metrics give us an idea of the overall sturcture of the network:

* The **diameter** tells us the longest shortest path between any two nodes in the network—in this case, 7 steps.
* The **average path length** tells us, on average, how many steps it takes to get from one node to another—about 2.7 in this case.

## Statistical measures

Now that you have created our graph, you can start doing some of the more analytical work. Let’s begin by exploring some common measurements that assess the value or importance of individual nodes within the network.

## Degree

You will begin with degree, using kiara's [`create.degree_rank_list`](#user-content-fn-1)[^1] module. This module allows us to calculate degree as both **undirected** and **weighted**. In this epistolary network, **undirected** **degree** counts the number of individual correspondents each person has, whereas **weighted** **degree** counts the total number of incoming and outgoing letters for each actor in the network.

Let's use our `retrieve_operation_info` function to check what you need to calculate these degrees:

```
kiara.retrieve_operation_info('calculate.degree_score')
```

Nice and simple. You just need to provide the network graph you created in `assemble.network_graph`.

Let’s give it a go then:

```
output = kiara.run_job('calculate.degree_score', inputs={'network_graph':CKCC['network_graph']}, comment="")
output
```

This function creates a table showing the undirected degree of each node. Since it automatically detects that the network is weighted (based on how it was created), it also calculates the weighted degree for each node. It has also assigned the two degree scores as node attributes in our network, which means you can keep these for further centrality measurements, allowing us to accumulate different scores rather than overwriting them each time.

## Betweeness

Let's look at a different centrality measure now. You will use `retrieve_operation_info` again to see what you need to calculate **betweenness** for the nodes in our network:

```
kiara.retrieve_operation_info('calculate.betweenness_score')
```

This module asks us to define how you want our weights to be interpreted – is the weight `'positive'`, indicating **strong relationships**, or is it `'negative'`, acting as a **distance** or **time needed** for these edges? Whilst this is often automated in network measures, kiara prompts us to think more carefully about our data and our network. This again allows us to trace the decisions you are making about your analysis.\


As you are dealing with **epistolary data**, you will eave this input as `'True'`, as the weight indicates **strength**. At this stage, the module is also set to calculate both **unweighted** and **weighted betweenness** using the network as a **directed graph**. Though this is another **pre-made** decision for this notebook and the dataset in use, it's important to acknowledge this and be as transparent about these kinds of choices as the ones actively documented by user input.

Let’s give it a go then. You want to use the network you just created using the degree ranking module, so let's save that and use it in our inputs:

```
network_graph = output['centrality_network']

output = kiara.run_job('calculate.betweenness_score', inputs={'network_graph':network_graph}, comment="")

output
```

Just like the degree module, it has returned a table with the two **betweenness scores**, ranked by unweighted, and also assigned these as **node attributes** that you can carry forward into more measurements.

Let’s look at one more centrality here in this notebook.

## Eigenvector

kiara also includes a module to measure **eigenvector centrality**, so let's look at what that needs:

```
kiara.retrieve_operation_info('calculate.eigenvector_score')
```

This module is set up similarly to the betweenness measure, and again you can define how to interpret the weights. If you have a larger dataset, you can also adjust the number of iterations for the calculation. For now, you will leave the parameters as they are and use our updated network graph, which already has **degree** and **betweenness** scores attached.

```
network_graph = output['centrality_network']

output = kiara.run_job('calculate.eigenvector_score', inputs={'network_graph':network_graph}, comment="")

output
```

As before, you now have a **score table** and updated **node attributes** in the network graph — great!

There is one final centrality measure available in the network analysis plugin: **closeness**. See if you can figure out how to check the information for this and run it on the network here, or feel free to move on to exploring other measures.

## Modularity Group

This next module determines the **modularity groups** in the network, again assigning each group as a **node attribute**. Let’s have a look at the parameters for it:

```
kiara.retrieve_operation_info('compute.modularity_group')
```

Here, you can either **set the number of communities** you want the module to divide the network into, or you can allow the code to find this automatically.

Let’s give it a go with our network once more:

```
network_graph = output['centrality_network']

output = kiara.run_job('compute.modularity_group', inputs={'network_graph':network_graph, 'number_of_communities':10}, comment="")

output
```

Great — this once again gives us our **updated network**, and also tells us how many **modularity groups** the measure has found in the network.

Let's look at one last measure.

## Cut points

This last function finds all the **cut-points** in the network — these are nodes which, if removed, would split the component into two or more separate pieces. The function returns a **list of cut-points**, and also assigns each node a **'Yes'** or **'No'** as a node attribute to indicate whether it is a cut-point.

Let’s have a look one last time:

```
kiara.retrieve_operation_info('create.cut_point_list')
```

Nice and simple — no extra parameters needed: it just requires our network.

It’s worth pointing out that the **cut-point** function in **NetworkX** does not work on **directed** or **multidirected** graphs. If you are using one of these graph types, this `create.cut_point_list` function will first convert the graph into an **undirected** version to perform the calculation, and then it will return the results in your original directed graph. This does not affect the metrics, but it’s good to know!

Let’s run it on our network:

```
network_graph = output['modularity_network']

output = kiara.run_job('create.cut_point_list', inputs={'network_graph':network_graph}, comment="")

output
```

As with the previous modules, this gives us an updated **node table** with a new **Cut Point** column (`Yes` or `No`), and a **list** **of nodes** that are identified as cut-points.

Having started simply with an imported CSV of letter edges, you have now got a lot of information. This is great — but what next?

## Export the network

kiara has stored all of the information you have just created, and because it’s **interoperable**, it also allows us to **export** the network again.

You can export this network data as a set of **CSVs**, or as one of several **network file formats** (such as **graphml** or **gexf**) using built-in kiara modules like this:

```
kiara.retrieve_operation_info('export.network_graph')
```

From this, you can work with our kiara network object in other software — for example, to do **further analysis** or to create **visualisations**!

Give it a go yourself!

Finally, you can check out the **lineage** for our final `cut_network` output. This shows how **all of our decisions** were stored — and how they shaped the creation of our network — all the way from the original import step.

```
lineage = kiara.retrieve_augmented_value_lineage(output['cut_network'])
from observable_jupyter import embed
embed('@dharpa-project/kiara-data-lineage', cells=['displayViz', 'style'], inputs={'dataset':lineage})
```

## Onboarding data: an alternative

So far, you have created a network object in kiara by importing a **CSV** from a local path.

But what about other formats? Let’s pause quickly and look at how to import a **GML file** instead.

Here you will use a different sample dataset, a [co-appearance network](http://www-personal.umich.edu/~mejn/netdata/) of characters in Victor Hugo's novel _Les Miserables_, which is already in GML format.

Let’s take a look at the function `import.network_graph.from.file` and how it works:

```
kiara.retrieve_operation_info('import.network_graph.from.file')
```

For this, you only need the **path** to the file you want to use — there is no need to import it into kiara first, as this module takes care of everything in one step.

If the **node labels** in your GML file are named something other than `'id'`, you can specify that using the `label` input. If the **weight column** in your file has a different name (for example, in the _Les Misérables_ graph it is `'value'`), you can specify that too — kiara will then rename it to `'weight'` so that further metrics can use the weighted edges.

Let’s try it:

```
lesmis_path = os.path.join(notebook_path,"data/lesmis.gml")

lesmis = kiara.run_job('import.network_graph.from.file', inputs={'path': lesmis_path, 'file_type':'gml', 'weight_column':'value'}, comment="")
lesmis
```

As you can see, this module **not only imports the GML file** into kiara, but also **automatically converts it** into a kiara network object for us — great!

Notice that the **edge table** now uses the column name `'weight'`, which was automatically updated from `'value'` to match kiara’s expected format for edge weights.

This network can now be used in degree calculations, just like you did before:

```
output = kiara.run_job('calculate.degree_score', inputs={'network_graph':lesmis['network_graph']}, comment="")
output
```

You will leave this _Les Misérables_ network here — but it’s useful to see this **alternative option** for importing data for networks. If you want to experiment with this dataset more, feel free to give it a try!

[^1]: This should be replaced with calculate.degree\_score?
