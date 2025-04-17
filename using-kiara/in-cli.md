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

### Choose and run modules

To see the modules (a.k.a operations) available, along with short descriptions, use:

```
kiara operation list
```

To find out what inputs are needed to run any of these modules , use:

```
kiara operation explain <module ID>
```

This will provide you with&#x20;

To run any module, use:&#x20;

{% code overflow="wrap" %}
```
kiara run <module ID> <field name for input>=<the string, boolean or table required for that input> <any further inputs in same format> -s <kiara name for output>=<output alias> -c <comment>
```
{% endcode %}

You don't have to save your output each time ( `-s`), but should always leave a comment (`-c`).

### E.g. create a network for analysis

First you'll need to turn your imported .csv file into a table within kiara:

<pre data-overflow="wrap"><code><strong>kiara run create.table.from.file file=alias:&#x3C;file alias> -s table=&#x3C;table alias> -c creating_table
</strong></code></pre>

Now you can create the network using the module `assemble.network_graph`.

But first, use `kiara operation explain assemble.network_graph` to find out what input decisions are required. All of the relevant information will be presented to you in a table, explaining what the module does, what it needs (inputs), and what it creates (outputs). For example, based on the information provided on `assemble.network_graph`, you would write the following command if you wanted to create a directed weighted graph where the parallel edges are added together to give the weight to the edge:

{% code overflow="wrap" %}
```
kiara run assemble.network_graph graph_type='directed' edges=alias:<table alias> source_column='Source' target_column='Target' is_weighted=True parallel_edge_strategy='sum' -s network_graph=<graph alias> -c creation_of_graph
```
{% endcode %}

Now you have your desired graph (under 'Result'), and can analyse it using one or more analysis modules. For example:

```
kiara operation explain tropy.calculate.degree_score
```

