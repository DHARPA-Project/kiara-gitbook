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

To create and use a new context called `'hello_kiara'`:

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

Now we can explore the tools kiara offers. To view a list of all available operations (based on the installed plugins), run:&#x20;

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

