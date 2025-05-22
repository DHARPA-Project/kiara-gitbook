# Network analysis in Jupyter Notebook

## Installing kiara and its plugins

Before we get started, we need to check whether kiara and its associated plugins are installed. kiara's features are available through plugins.&#x20;

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

## Setting up kiara

Now that the plugins are ready, let's set up kiara itself.

To start using kiara in Jupyter, you need to create an instance of the `kiaraAPI`. This API provides access to kiara's functions, enabling you to interact with and control your data workflows.

To set this up, run the following code in a notebook cell:

```
from kiara.api import KiaraAPI

kiara = KiaraAPI.instance()
```

## Creating a project

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

## Downloading a file&#x20;

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

## Converting the file into a table

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

Inputs

* Required: a file.
* Optional:
  * `first_row_is_header` – indicates if the first row of a CSV file contains column headers.
  * `delimiter` – specifies the column separator (only for CSV), used if kiara cannot auto-detect it.

Outputs

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

## Querying your data

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

## Recording and tracing your data

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

