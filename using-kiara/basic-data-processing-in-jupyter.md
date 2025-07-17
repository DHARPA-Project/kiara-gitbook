# Basic data processing in Jupyter

## Activate your kiara environment

As you have seen in the install instructions, you need to create a special [environment](https://docs.anaconda.com/working-with-conda/environments/) for kiara to run in. Use the following command to activate your previously created kiara environment, replacing `kiara_explore` with whatever name you assigned it:

```
conda activate kiara_explore
```

> Tip: to check what environments you have created in the past, you can use `conda env list`

## Dependencies

You already installed some basic plugins when setting up your kiara environment. Now you can use conda to also install the necessary packages for using kiara in Jupyter notebooks.

You'll be using Jupyter notebook and Observable within that notebook, so enter:

```
conda install jupyter observable_jupyter
```

## Start Jupyter notebook

To open the Jupyter interface, run:

```
jupyter notebook
```

## Import kiara and create an API

To start using kiara in Jupyter Notebook, you first need to create a `kiaraAPI` **instance**. This instance allows us to control kiara and see what operations are available.&#x20;

To set this up, run the following code in a notebook cell:

```
from kiara.api import KiaraAPI

kiara = KiaraAPI.instance()
```

## Create a project context

In kiara, a [**context**](../before-you-begin/key-concepts.md#context) is your project space. To create and use a **new context** called `project1_DP`, run the following code:

```
kiara.set_active_context(context_name='project1_DP', create=True)

print('Available Contexts:', kiara.list_context_names())
print('Current Context:', kiara.get_current_context_name())
```

This operation will also show all your available contexts and confirm which one is currently active. The output will be something like:&#x20;

```
Available Contexts: ['default', 'project1_DP']
Current Context: project1_DP
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

Now that kiara is set up, let's bring a file into your notebook using the `download.file` operation. &#x20;

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

Now that you have transformed and queried your data, let's review what κiara knows about the outputs you have created and how it tracks changes. **Data lineage** is one of kiara’s most powerful features.&#x20;

Let’s check the lineage of your query output:

```
query_output = outputs['query_result']
query_output
```

Even though you have made changes along the way, you can still access a lot of information about your data.&#x20;

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
