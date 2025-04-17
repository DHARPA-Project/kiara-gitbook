# in CLI

### Import your data

Import the .csv file(s) you want to analyse:

{% code overflow="wrap" %}
```⏎
kiara run import.local.file path=/<full path to your .csv> -s file=<file alias> -c importing_data   
```
{% endcode %}

`-s` saves your imported file in the kiara environment, allowing you to call upon it later using the assigned alias.

Tracking your steps through comments is a fundamental aspect of using _kiara_, so don't forget to include an accurate description after `-c` , as done here with `importing_data` .

### Choose and run operations

To see the operations available, use:

```
kiara operation list
```

To run any operation, use:&#x20;

```
kiara run <operation_name>
```

> Tip: if you are ever unsure what the names for inputs and outputs are in kiara, run `kiara operation explain <module name>` .

### E.g. create a network for analysis

First you'll need to turn your imported .csv file into a table within kiara:

<pre data-overflow="wrap"><code><strong>kiara run create.table.from.file file=alias:&#x3C;file alias> -s table=&#x3C;table alias> -c creating_table
</strong></code></pre>

Now you can create the network using the module `assemble.network_graph`.

But first, use `kiara operation explain assemble.network_graph` to find out what input decisions are required, and what output you can expect. All of this information will be presented to you in a&#x20;

For example, if you wanted to create a directed weighted graph, where the parallel edges are added together to give the weight to the edge, you would use:

{% code overflow="wrap" %}
```
kiara run assemble.network_graph graph_type='directed' edges=alias:<table alias> source_column='Source' target_column='Target' is_weighted=True parallel_edge_strategy='sum' -s network_graph=<graph alias> -c creation_of_graph
```
{% endcode %}
