# in CLI

### Upload your data

Import the .csv file(s) you want to analyse:

```⏎
kiara run import.local.file path=/...
```

Tracking your steps through comments is a fundamental aspect of using _kiara_, so don't forget to include an accurate description after `-c` , as done here with `importing_data` .

### Choose and run an operation

To see the operations available, use:

```
kiara operation list
```

To run any operation, use:&#x20;

```
kiara run <operation_name>
```

> Tip: if you are ever unsure what the _kiara_ names for inputs and outputs are, run `kiara operation explain` followed by the module name.
