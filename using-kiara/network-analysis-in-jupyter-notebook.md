# Network analysis in jupyter notebook

## Install kiara&#x20;

Before running any notebook, you will need to install kiara on your computer using the command-line interface (CLI). We will guide you through the process step by step.

To begin, you must install conda or miniconda, which are tools for managing software environments and dependencies. We recommend installing miniconda, which is the lighter version of conda. Installation instructions for miniconda can be found [here](https://docs.anaconda.com/miniconda/).&#x20;

Be sure to download the right version for your operating system (Windows, macOS, or Linux).

## Set up a kiara environment

We suggest creating a separate environment for kiara. This makes it easier to manage and avoid conflicts with other software. You can do this by opening your CLI and typing:

```
conda create -n kiara_testing python jupyter
```

You can replace `kiara_testing` with any name you like for your environment.

Once the environment is created, activate it with:

```
conda activate kiara_testing
```

## Install packages

Now that your environment is set up, you can begin installing the necessary packages. kiara is not available directly through conda, so we’ll use `pip`, another common package manager:

```
pip install kiara
```

The first installation may take a few minutes. Once kiara is installed, we will also add some essential plugins by running:&#x20;

```
pip install kiara_plugin.core_types kiara_plugin.onboarding kiara_plugin.tabular
```

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

As we will see in the next section, depending on which notebooks you wish to run, you may also need to install the plugins for topic modelling and network analysis.

## Install kiara's plugins

Before we get started with network analysis, we need to check whether kiara and its associated plugins are installed. kiara's features are available through plugins.&#x20;

There are seven plugins:&#x20;

* [`kiara_plugin.core-types`](https://dharpa.org/kiara_plugin.core_types/latest/)
* [`kiara_plugin.onboarding`](https://dharpa.org/kiara_plugin.onboarding/latest/)
* [`kiara_plugin.tabular`](https://dharpa.org/kiara_plugin.tabular/latest/)
* [`kiara_plugin.network_analysis`](https://dharpa.org/kiara_plugin.network_analysis/latest/)
* [`kiara_plugin.language_processing`](https://dharpa.org/kiara_plugin.language_processing/latest/)
* [`kiara_plugin.html`](https://dharpa.org/kiara_plugin.html/latest/)
* [`kiara_plugin.streamlit`](https://dharpa.org/kiara_plugin.streamlit/latest/)

To install these, first launch Jupyter from the command line by running:

```
jupyter notebook
```

This will open the Jupyter notebook in your default web browser.&#x20;

In a notebook cell, run the following code:

```
try:
    from kiara_plugin.jupyter import ensure_kiara_plugins
except:
    import sys
    print("Installing 'kiara_plugin.jupyter'...")
    !{sys.executable} -m pip install -q kiara_plugin.jupyter
    from kiara_plugin.jupyter import ensure_kiara_plugins

ensure_kiara_plugins()
```

## Set up kiara

Now that the plugins are ready, let's set up kiara itself.

To start using kiara in Jupyter, you need to create an instance of the `kiaraAPI`. This API provides access to kiara's functions, enabling you to interact with and control your data workflows.

To set this up, run the following code in a notebook cell:

```
from kiara.api import KiaraAPI

kiara = KiaraAPI.instance()
```

## Create a project

In kiara, a context is your project space. It keeps track of your data, the tasks you run, and the steps you take. A default context is always available, but you can also create your own for specific projects.&#x20;

To create and use a new context called `hello_kiara` , run the following code:

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

Now, we can explore the tools kiara offers. To view a list of all available operations (based on the installed plugins), run:&#x20;

```
kiara.list_operation_ids()
```

This will return a list of operations, like:

```
['create.table.from.file', 'calculate.degree_score', 'export.table.as.csv_file', ...]
```

Each operation is a task you can perform in kiara, such as creating a table, calculating network metrics, or exporting files.&#x20;

## Download a file&#x20;

Now that kiara is set up, let's bring a file into our notebook using the `download.file` operation. &#x20;

To understand what this operation does and what information it needs, run:

```
kiara.retrieve_operation_info('download.file')
```

We’ll now download a sample CSV file using this operation. First, we define the input (the file URL and name) and then run the job:

```
inputs = {
    "url": "https://raw.githubusercontent.com/DHARPA-Project/kiara.examples/main/examples/data/network_analysis/journals/JournalNodes1902.csv",
    "file_name": "JournalNodes1902.csv"
}

outputs = kiara.run_job('download.file', inputs=inputs, comment="importing journal nodes")
```

This gives you a file object as output, including the downloaded file and some technical metadata. Let’s print it to confirm:

```
outputs
```

You will see a preview of the file's content. This shows the journal data was successfully downloaded.&#x20;

To keep using this file later (even if the notebook is closed), we will save it inside kiara using an alias. This works like giving a name that kiara remembers.&#x20;

```
downloaded_file = outputs['file']
kiara.store_value(value=downloaded_file.value_id, alias='Journal_Nodes')
```

Now, `Journal_Nodes` is saved in kiara's internal storage. You can refer to it later just by its alias, just like using a variable in Python.&#x20;

## Convert the file into a table

Now that we have downloaded the file, let's turn it into a table so we can work with the data.&#x20;

We can look through kiara’s available operations by filtering for those that start with `create`:

```
kiara.list_operation_ids('create')
```

This shows a list of operations. Since we’re working with a CSV file, the one we want is `create.table.from.file` .

This operation will read the file and turn it into a structured table.

To see what inputs and outputs this operation expects, run:

```
op_id = 'create.table.from.file'
kiara.retrieve_operation_info(op_id)
```

From this, we learn:

**Inputs**

* Required: a file.
* Optional:
  * `first_row_is_header` – indicates if the first row of a CSV file contains column headers.
  * `delimiter` – specifies the column separator (only for CSV), used if kiara cannot auto-detect it.

**Outputs**

* A `table` object, which can be used in the next steps.

Let’s turn the downloaded file (which we saved earlier under the alias `Journal_Nodes`) into a table:

```
inputs = {
    "file": kiara.get_value('Journal_Nodes'),
    "first_row_is_header": True
}

outputs = kiara.run_job(op_id, inputs=inputs, comment="")

outputs
```

This will process the CSV file and show the result as a table with columns and rows.

To make it easier to reuse the table later, we can save it in kiara under a new alias:

```
outputs_table = outputs['table']
kiara.store_value(value=outputs_table.value_id, alias="Journal_Nodes_table")
```

Now, your data is saved inside kiara and can be accessed at any time using the name `Journal_Nodes_table`.

## Query your data

Now that we have downloaded the file and converted it into a table, we can start exploring the data. One simple way to do that is by running SQL queries directly on the table using κiara.

To find relevant operations for querying data, search with the keyword `'query'`:

```
kiara.list_operation_ids('query')
```

This returns:

```
['query.database', 'query.table']
```

Since we are working with a table, we will use:

```
kiara.retrieve_operation_info('query.table')
```

This tells us that `query.table` allows us to write an SQL query to explore the data. The required inputs are:

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

Let's narrow this further and find all the journals that are about general medicine and published in Berlin.

We can re-use the `query.table` function and the table we've just made, stored in `outputs['query_result']`

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

Now that we’ve transformed and queried our data, let's review what κiara knows about the outputs we've created and how it tracks changes.

```
query_output = outputs['query_result']
query_output
```

Even though we have made changes along the way, we can still access a lot of information about our data.&#x20;

Specifically, the operation gave us:

* A unique value ID
* The data type (in this case, a `table`)
* When the value was created
* A record of the job that generated it
* Links to the inputs and outputs of previous steps

kiara automatically traces all of these changes, keeping track of inputs and outputs and assigning each a unique identifier, so you always know exactly what has happened to your data.&#x20;

To see a 'backstage' view of how your data was transformed, including the inputs for each function we have run and how they connect, run the following:

```
query_output.lineage
```

This shows a chain of operations:

* The SQL query you ran (`query.table`)
* The table that was queried (from `create.table`)
* The original file that was downloaded (`download.file`)

Each input is assigned a unique ID, allowing complete transparency and traceability.

## Network analysis with kiara

Now that we're comfortable with what kiara looks like and what it can do to help track your data and your research process, let's try out some of the digital analysis tools, starting with network analysis.

## Why network analysis?

Network analysis offers a computational and quantitative means to examine and explore relational objects, with proxies to measure structural roles and concepts such as power and influence. Doing so digitally - and at scale - also allows us to consider these kinds of questions with large amounts of material or documents that were not heretofore manageable with qualitative or manual approaches.

We will not get into any core network theories or their uses in the humanities here, as we're focused on the ways in which network analysis in kiara offers an interesting way to wrap the research process, and think about the decisions we're making and how to trace them. If you're interested in learning more about network analysis, or how to code using [NetworkX](https://networkx.org/), the library currently used in these kiara modules, check out our recommended reading at the bottom.

## Get started

Before we begin exploring **network analysis**, let's make sure everything is ready.

In this step, we will check that the necessary plugins are available and set up the Kiara API, which is the interface that allows us to run Kiara commands inside our Jupyter notebook.

This is the code to get started:

```
import networkx as nx
import os
from kiara.api import KiaraAPI
kiara = KiaraAPI.instance()
```

## Import data

Next, we will set up the filepaths for the data that we are going to use in this notebook. The data file is stored in the same directory as the two jupyter notebooks. To set the file path, you can either save the full path to the csv file in the variable below, or use the `os.path` modules in Python to shorten this, as below:&#x20;

```
notebook_path = os.path.abspath('')

csv_file_path = os.path.join(notebook_path,"data/CKCC.csv")
```

Great, we are all set up. We are now going to import some data again using the kiara function `import.local.file`. This function will allow you to bring in a local file, one stored on your computer. We're using sample data, but you can also use this function to import your own data.

The dataset we’re using is a sample from the **Circulation of Knowledge and Learned Practices in the 17th-century Dutch Republic (CKCC)** collection, compiled by the Huygens Institute in the Netherlands and made available on the LetterSampo portal, part of the Reassembling the Republic of Letters project. You can find more information about these projects [here](https://seco.cs.aalto.fi/projects/rrl/).

This collection includes about 20,000 letters written by and to 17th-century scholars in the Dutch Republic. By using network analysis, we can explore questions such as:

* Who was the most prolific writer?
* Which actor connected the most people?
* Who operated in closely knit writing groups?

While network analysis can be used to explore and map unknown datasets, in this case, we already know something about the data. The research questions and module parameters in this notebook have been shaped by that prior knowledge. That is important to keep in mind as we proceed.

Let’s now use the `import.local.file` module from Kiara to access our CSV file. We will specify the path to the CSV file in our inputs and save the outputs of the function as '**CKCC**'. Alternatively, we can use the `download.file` module used in the **Hello Kiara** notebook.

We will leave the comments blank here for you to fill in yourself, but the comment here might indicate why you have chosen this dataset, or a reminder of which version you are working with if you have multiple versions of the same dataset.

```
CKCC = kiara.run_job('import.local.file', inputs={'path': csv_file_path}, comment="importing bits")
```

## Create a network

Now that we’ve imported our data, it’s time to build a network from it.&#x20;

As with most network analysis tools, kiara requires the data to be in the form of an **edge** table first. An edge table shows the connections or relationships between different entities, in this case, between senders and recipients of letters. Later, we could also add a table with nodes (the individual entities), but that is optional, and we will skip it for now.

To transform our CSV file into an edge table, we will use the `create.table.from.file` function that we used in the first notebook. We will save this table in a variable called **CKCC**, which we will reuse later on.

Before running it, we should check the input requirements of the function, just to make sure we are using it correctly. You can do that with the following command:

```
kiara.retrieve_operation_info('create.table.from.file')
```

This will display useful information about the function, such as the inputs it needs and the outputs it produces.

Now, we can turn our CSV data into a kiara table by loading the data file we imported earlier (the one stored in `CKCC`) and telling Kiara that the first row of the file contains the column headers. You can do that with the following command:

```
inputs = {
    "file": CKCC['file'],
    "first_row_is_header": True
}

outputs = kiara.run_job('create.table.from.file', inputs=inputs, comment="")

edges = outputs['table']

outputs
```

## Preview the network structure

Now that we have our edges formatted as a kiara table, we are ready to make our network graph. But before we do that, it is helpful to preview the structure of the network using kiara’s `preview.network_info` function. All we need to do is select our edges table and the column names for our sources and targets by running:

```
inputs = {'edges': edges,
    'source_column': 'Source',
    'target_column': 'Target'}

network_info = kiara.run_job('preview.network_info', inputs=inputs, comment="")

network_info
```

This function gives us the total number of nodes, but it also helps us think about how different types of graphs  - **directed**, **undirected**, **multi-directed**, and **multi-undirected** - might affect the number of edges in the network.

We see that there are more edges in a directed graph than in an undirected graph. This suggests that there are reciprocal or directed edges between a pair of nodes, something typical in an epistolary network, where people are writing back and forth to each other.&#x20;

We also notice that there are even more edges in a multigraph than in either of our non-multigraphs, which means the dataset includes parallel edges (i.e., duplicates in our edge table). Again, this is common for an epistolary network, where someone writes more than one letter to their friend.

The preview shows no isolates (nodes without any edges) and a number of components. However, we see a large number of self-loops. This is unusual in epistolarly collections, as people are unlikely to write to themselves.&#x20;

So, in addition to helping us decide what graph type is most useful for our dataset, this module helps us to review our data by flagging up potential errors or inconsistencies in our dataset that we can go back to at some point.

Having access to this kind of information means we can make more informed decisions about the next steps of our research or digital analysis, especially those that are sometimes automated for us.

For our network, a **directed graph** makes the most sense.&#x20;

Let's now look at what we need to build one with our `assemble.network_graph` module using `kiara.retrieve_operation_info`.

```
kiara.retrieve_operation_info('assemble.network_graph')
```

## Graph decisions

This might seem like a chunky module, but it is doing a lot of important work up front. By making these decisions now, we will not have to make them later on when we move on to the more analytical parts.&#x20;

If we change our mind later about the kind of graph we want to use, we can always come back and rerun this step. This is why the `preview.network_info` function is very useful. It allows us to make an informed decision about our network early on.

We have already decided that we want to make a directed network, so we will select 'directed' for graph type. We created our edge table earlier and saved it as 'edges', so we can load that back in. We also need to specify our Source and Target columns again, and we can copy all this information from our preview module.&#x20;

We do not have a node table for this dataset, but if we did, this would be the place to include it.

Now, we can make some more decisions that we have not seen yet. One of the most important is deciding whether our network is **weighted** or **unweighted.** This can mean different things depending on your data, such as the number of letters between correspondents, the distance between them, or the number of years they have known each other. If all the relationships between nodes in your network are the same, we can set `is_weighted` to `False`.  But if not, we need to tell kiara where this weight information is coming from.

If weights already exist in the edges table, for example, if you have already assigned weights to the network before uploading the data into kiara, then you can just select the weight column. In case your graph contains parallel edges, which we would already know exist from the `preview.network_info` , and you prefer not to use a multigraph, you can specify how to handle these weighted parallel edges. You can choose to aggregate the weights of the parallel edges by summing them `sum` . calculating their average weight `mean` , finding the highest value `maximum` , or the lowest `minimum`. Each of these choices will assign a single value as the new weight for that edge in the network.&#x20;

If you want kiara to calculate the weights for you, you can choose `sum` , which will count the total number of times each edge appears and use that as the weight. Keep in mind that if you have not provided any weights in the data, kiara will automatically assign a weight of 1 to each edge. In that case, selecting `mean` , `minimum`, or `maximum` will simply return 1 for every edge, making the result he same as an **unweighted** network.

The inputs for this module encourage us to reflect on the decisions we are making as we go, and think about how our data fits into these kinds of measurements. By working through these steps in kiara, we are not only making choices but also documenting them. Kiara tracks these decisions, both through the way the process is stored and through the comments we can include along the way.

Since we are still working with our letter dataset, we will ask kiara to add all the edges together so that the weight will tell us how many letters each person wrote to each other.

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

Great, this has created a Kiara network graph object. The output includes both an edge table and a node table for the network.

Since we did not provide a separate node table, kiara has automatically extracted the node information from the edges. If you look at the edge table now, you will see that it includes weights, calculated according to the decision we just made.&#x20;

As we saw earlier in the `preview.network_info` module, the network consists of eight separate components. We might want to focus on just the main component, which includes the most nodes. To do this, we can extract the largest component and return it as its own network graph.&#x20;

```
CKCC_largest_component = kiara.run_job('extract.largest_component', inputs={'network_graph':CKCC['network_graph']}, comment="")

CKCC_largest_component
```

## Structural measures

