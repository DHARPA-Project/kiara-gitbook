---
description: How to assemble multiple kiara modules into a pipeline.
---

# Assembling a kiara pipeline

## Preparation

## Creating a pipeline

A kiara pipeline is a dict-like data structure that includes one or several processing steps (implemented by kiara operations), connected (or not) in a specific way so that 
some steps outputs feed into other steps' inputs.

### A single-step pipeline

The simplest pipeline contains a single operation, and is not useful in any way, since it's easier to just `kiara run`
the operation directly. Nonetheless, below is how that would look like, we'll be using the module we created in the [writing your own kiara module](../../creating_modules/the_basics) guide:

```yaml
steps:
  - module_type: filter.table
    step_id: filter_table_step
```

A pipeline step is a dictionary with 2 required keys (and some optional ones, which we'll cover later):

- **`module_type`**: the name of the module or operation that should be used.
- **`step_id`**: the name of the step, ideally a short, descriptive name outlining what the step does. It can't contain special characters except '_', 
- and it must be unique within the pipeline.

An assembled pipeline has the same characteristics as a kiara module, and in fact is a perfectly valid operation, like any other one, and can be called the same way.

Create a new file `my_first_pipeline.yaml`, and copy and paste the above code into it. Then, run the `operation explain` command against the file:

```
uv run kiara operation explain my_first_pipeline.yaml
```

As you can see, kiara turned this (single-step) pipeline into an operation, and auto-generated some input- and output-fields, by assembling the step-id and step input-/output-field(s). 
Those long field names are a bit unwieldy, and we'll remedy that later, for now let's just ignore that.

### Adding a second step

In the previous tutorial we pre-seeded the kiara data store with a csv file/tabular dataset, to help us with developing our table filter module. 
In this tutorial, we'll remove the requirement to do that, by adding a step to our pipeline that lets the user specify a path to a csv file, 
and import and convert that into a table value.

Previously, we've used the `import.table.from.local_file_path` operation to import the csv file, and we can do the same now. 
Edit the pipeline file you created so it looks like the following:

```yaml
steps:
  - module_type: import.table.from.local_file_path
    step_id: import_table_step
  - module_type: filter.table
    step_id: filter_table_step
    input_links:
      table_input: import_table_step.table
```

What we did here:

- add a new step with the id `import_table_step`, which will execute the `import.table.from.local_file_path` operation
- leave our filter step in place, but connect the `table_input` input of this steps operation to the `table` output field of the `import_table_step` operation

!!! note:
    To find out the input/output field names of each step, use `kiara operation explain <MODULE_TYPE>`.

We can ask kiara again about what it thinks of this new pipeline/operation:

```
uv run kiara operation explain my_first_pipeline.yaml
```

As you can see, the previously existing input with the field name `filter_table_step__table_input` (type: `table`) is gone now, replaced by a new one, 
with the field name `import_table_step__path` (type: `string`). The other two inputs remain the same (since we did not connect a step output to them).

### Side-note: visualizing the pipeline

We can let kiara visualize our pipeline at each step in the development process. This is quite useful, as it can serve as a visual aid to debug and assemble pipelines and their steps.

Currently, the following commands exist to print a pipeline as graph on the command-line:

- `kiara pipeline execution-graph <pipeline_file>`: display the pipeline steps in the order they will be executed.
- `kiara pipeline data-flow-graph <pipeline_file>`: display the the connections of inputs/outputs as well as processing steps.

As an example, let's look at the execution graph of our current pipeline:

```
uv run kiara pipeline execution-graph my_first_pipeline.yaml
```

### Adjusting the input-/output-field names

We could run our pipeline as is, but let's adjust its input- and output field names first. Mainly to make it more intuitive to use. To do that we can add one or both of the 
following keys to our pipeline description:

- `input_aliases`: a mapping of pipeline inputs to more user-friendly names
- `output_aliases`: a mapping of pipeline outputs to more user-friendly names

Lets start with our inputs. Add the following to your pipeline file:

```yaml
input_aliases:
    import_table_step.path: csv_file_path
    filter_table_step.column_name: column_name
    filter_table_step.filter_string: filter_string
```

This is basically just a rename of one (or several, or all) pipeline-input-fields, to shorter names. If you specify the same value for several keys, then the user input 
for those fields will be re-used for all the keys that have that value (we'll cover that in a later tutorial).

Now let's do our outputs:

```yaml
output_aliases:
    filter_table_step.table_output: filtered_table
```

Output aliases work a bit different to input aliases: for inputs, if we don't specify a field, kiara will just use the auto-generated name (since the need for the input 
to exist does not go away). For output aliases, if we don't specify a field, kiara will ignore that output, and not display it to the user. 
In our case, we are not really interested in the intermediate outputs of the first step, so we only add the `filtered_table` alias that represents our final, filtered result.

Lets see what kiara has to say about the 'API' of our pipelines now:

```
uv run kiara operation explain my_first_pipeline.yaml
```

Much nicer!

### Run the pipeline

Now, all that is left to do is run the pipeline:

```
uv run kiara run --save filtered_table=amsterdam_journals my_first_pipeline.yaml csv_file_path=examples/data/journals/JournalNodes1902.csv filter_string=Amsterdam column_name=City
```

And to confirm this worked, we ask kiara about the value we just stored (alias: `amsterdam_journals`), including it's lineage, 
which should give us the value ids of the intermediate results (in case we ever needed them -- they won't have an alias associated with it, but are still 
persisted in the kiara data store and can be looked up with `kiara data explain <VALUE_ID>` and/or `kiara data load <VALUE_ID>):

```
uv run kiara data explain --lineage alias:amsterdam_journals
```

### Making the pipeline discoverable

One of the main concepts that kiara is trying to facilitate is a modular approach to assembling data workflows. The main ingredient here is the fact that assembled 
pipelines behave like any other operation in kiara, which means that pipelines can also be used as (single) steps in other pipelines.

To make that work in an intuitive and simple way for users, kiara needs to know about all the pipeline descriptions the user intends to use as building blocks. 
If we are only concerned about a single 'top-level' pipeline (as we have done in this tutorial so far), this is not a problem, and all we need to do is point 
kiara to a (yaml- or json-) file containing the pipeline description.

If we want to re-use our pipelines in other pipelines, we need to 'register' them in a kiara context, and give it its own operation id.

kiara tries to make this as simple as possible, so, if you don't do anything, kiara will use the file name that contains the pipeline description (without extension), 
and convert it to a valid operation id. In our case, we'd end up with an operation called 'my_first_pipeline'.

If we want to have more control over this, we can add a property `pipeline_name` to our description, that will override the auto-generated operation id.


#### Using the `extra_pipeline` property

Each kiara context has a property called `extra_pipeline`. On the command-line, you can specify one or several of those via the `--pipelines` (or: `-p`) argument (after the `kiara` 
command-name, before the sub-command name):

```
uv run kiara --pipelines my_first_pipeline.yaml operation list my_first
```

!!! note
    kiara accepts paths to files or folders as the argument to `--pipelines`. If you specify a folder, kiara will look for valid pipeline descriptions under that folder.

Let's test that again, but this time we give our new pipeline a descriptive name, as well as some documentation. Add those two properties to your existing pipeline file:

```yaml
pipeline_name: "import.filtered_table"
doc: |
  Import a table from a csv file, then filter it.

  Filtering is done by matching a 'filter_string' against all cells of a column, both specified by the user.
```

!!! note
    Here, we use a [yaml literal multiline string](https://www.baeldung.com/yaml-multi-line#literal-style) to specify our pipeline documentation (the: '`|`'). This allows us to write as much documentation as we 
    need to, without being restricted to a single line.

```
uv run kiara --pipelines my_first_pipeline.yaml operation list import
```

Going with the information contained in this output, instead of running our pipeline with the path to its file after the `run` sub-command, we could also do it like:

```
kiara --pipelines my_first_pipeline.yaml run import.filtered_table ... ... ...
```

#### Including the pipeline in a kiara plugin

In case we want to 'publish' our pipeline so it can be re-used as part of a kiara plugin (which may or may not contain native 'Python' modules/operations, custom data-types, etc.), 
this is also easy to do. If you have followed the 'writing your own kiara module' tutorial, you'd have created a kiara plugin project from a template. 
To add your pipeline to this plugin, simply copy/move it into the directory `src/kiara_plugin/<YOUR_PLUGIN_NAME>/pipelines`. Naming the pipeline and adding documentation 
works the same as outlined in the previous chapter.

If you add pipelines to a kiara context this way, nothing else should be necessary, kiara will auto-discover all the pipelines added like this, and you can use the assigned 
'pipeline_name' as value of the `module_type` key in your step description, if you want to run it within another pipeline.
