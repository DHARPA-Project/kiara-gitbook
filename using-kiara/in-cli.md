# in CLI

For those users comfortable working from the CLI instead of Jupyter notebooks, here is some general-level guidance to get you started using kiara:

### Import your data

Import the .csv file(s) you want to analyse:

{% code overflow="wrap" %}
```⏎
kiara run import.local.file path=/<full path to your .csv> -s file=<file alias> -c importing_data   
```
{% endcode %}

`-s` saves your imported file in the kiara environment, allowing you to call upon it later using the assigned alias.

Tracking your steps through comments is a fundamental aspect of using kiara, so don't forget to include an accurate description after `-c` , as done here with `importing_data` .

### Choose and run modules

To see the modules (a.k.a operations) available, along with their IDs in kiara and short descriptions, use:

```
kiara operation list
```

Each operation shown in the list is a task you can perform in kiara, such as creating a table, calculating network metrics, or exporting files. To find out more about any of these modules, use:

```
kiara operation explain <module ID>
```

This will provide you with documentation on that operation (i.e. what it does), the inputs it requires or allows, and the outputs it creates. The field names provided here – for inputs and outputs – are vital knowledge for running modules, given that:

To run any module, use:

{% code overflow="wrap" %}
```
kiara run <module ID> <field name(s) for input(s)>=<required input(s)> -s <field name for output>=<output alias> -c <comment>
```
{% endcode %}

You don't have to save your output each time ( `-s`), but should always leave a comment (`-c`).

### E.g. create a network for analysis

First you'll need to turn your imported .csv file into a table within kiara:

<pre data-overflow="wrap"><code><strong>kiara run create.table.from.file file=alias:&#x3C;file alias> -s table=&#x3C;table alias> -c creating_table
</strong></code></pre>

Now you can create the network using the module `assemble.network_graph`.

But first, use `kiara operation explain assemble.network_graph` to find out what input decisions are required and what the field name for the output is.

For example, based on the information provided for `assemble.network_graph`, you would write the following command if you wanted to create a directed weighted graph where the parallel edges are added together to give the weight to the edge:

{% code overflow="wrap" %}
```
kiara run assemble.network_graph graph_type='directed' edges=alias:<table alias> source_column='Source' target_column='Target' is_weighted=True parallel_edge_strategy='sum' -s network_graph=<graph alias> -c creation_of_graph
```
{% endcode %}

This will produce your desired graph in tabular form, under 'Result'. Now you can analyse it using one or more analysis modules. As before, start with `kiara operation explain <module ID>` to find out what is needed.
