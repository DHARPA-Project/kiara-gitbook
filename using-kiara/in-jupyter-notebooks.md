# in Jupyter notebooks

## Run kiara

To start using kiara in Jupyter notebooks, you need to create an instance of the `kiaraAPI`. This API provides access to kiara's functions, enabling you to interact with and control your data workflows.

To set this up, launch Jupyter from the command line by running:

```
jupyter notebook
```

This will open the Jupyter notebook in your default web browser.&#x20;

In a notebook cell, run the following:

```
from kiara.api import KiaraAPI

kiara = KiaraAPI.instance()
```

## Create a project

In kiara, a context is your project space. It keeps track of your data, the tasks you run, and the steps you take. A default context is always available, but you can also create your own for specific projects.&#x20;

To create and use a new context called `"Hello_kiara"`:

```
kiara.set_active_context(context_name='hello_kiara', create=True)
```

To see all your available contexts and confirm which one is currently active:

```
print("Available Contexts:", kiara.list_context_names())
print("Current Context:", kiara.get_current_context_name())
```

This will output something like:&#x20;

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

Now, `"Journal_Nodes"` is saved in kiara's internal storage. You can refer to it later just by its alias - just like using a variable in Python.&#x20;

## Convert the file into a table

Now that we have downloaded the file, let's turn it into a table so we can work with the data.&#x20;

We can look through kiara’s available operations by filtering for those that start with `"create"`:

```
kiara.list_operation_ids('create')
```

This shows a list of operations. Since we’re working with a CSV file, the one we want is:

```
create.table.from.file
```

This operation will read the file and turn it into a structured table.

To see what inputs and outputs this operation expects, run:

```
op_id = 'create.table.from.file'
kiara.retrieve_operation_info(op_id)
```

From this, we learn:

* It requires a file.
* It optionally accepts:
  * `first_row_is_header` – set this to `True` if the first row contains column names.
  * `delimiter` – if Kiara can’t automatically detect it.

The output will be a table, which we can use in later steps.

Let’s turn the downloaded file (which we saved earlier under the alias `'Journal_Nodes'`) into a table:

```
inputs = {
    "file": kiara.get_value('Journal_Nodes'),
    "first_row_is_header": True
}

outputs = kiara.run_job(op_id, inputs=inputs, comment="creating table from journal CSV")
```

This will process the CSV file and show the result as a table with columns and rows.

To make it easier to reuse the table later, we can save it in Kiara under a new alias:

```
outputs_table = outputs['table']
kiara.store_value(value=outputs_table.value_id, alias="Journal_Nodes_table")
```

Now, your data is saved inside Kiara and can be accessed at any time using the name `"Journal_Nodes_table"`.

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
* `query`: your SQL statement\
  (use `data` as the name of the table inside the query)

Let’s find all the rows where the City is Berlin:

```
inputs = {
    "table": kiara.get_value('Journal_Nodes_table'),
    "query": "SELECT * FROM data WHERE City LIKE 'Berlin'"
}

outputs = kiara.run_job('query.table', inputs=inputs, comment="")
```

The result (in `outputs['query_result']`) is a filtered table showing only journals published in Berlin.

We can now go one step further: from the previous result, let’s find only the journals about general medicine.

```
inputs = {
    "table": outputs['query_result'],
    "query": "SELECT * FROM data WHERE JournalType LIKE 'general medicine'"
}

outputs = kiara.run_job('query.table', inputs=inputs, comment="")

```

This returns a smaller table with only the Berlin-based general medicine journals.

