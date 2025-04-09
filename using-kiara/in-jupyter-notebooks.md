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

In Kiara, a context is your project space. A context keeps track of your data, the tasks you run, and the steps you take. To create and use a new context called `'hello_kiara'`, run:

```
kiara.set_active_context(context_name='hello_kiara', create=True)
```

