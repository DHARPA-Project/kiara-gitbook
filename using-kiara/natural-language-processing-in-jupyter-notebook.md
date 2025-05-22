# Natural Language Processing in Jupyter Notebook

## Installing kiara and its plugins

Before we begin, we must ensure that kiara and its plugins are correctly installed.

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

This will check (and if needed, install) everything required to use Kiara inside your notebook.

## Choosing a text corpus

Now that the environment is ready, we need a small collection of texts to work with.

For this tutorial, we use a sample of OCRed (machine-readable) front pages of _La rassegna_, an Italian-language newspaper published in the United States from 1898 to 1936. This sample is part of a larger corpus in an open-access digital heritage project, ChroniclItaly 3.0  (Viola and Fiscarelli, 2021; Viola, 2021). The front pages of _La rassegna_ were collected via Chronicling America, a searchable database of newspapers published in the US maintained by the Library of Congress. These texts are a good example for topic modeling, because they contain metadata information, e.g., the publication date.&#x20;

The file name structure is: LCCNnumber\_date\_pageNumber\_ocr.txt. Therefore, the file name ‘sn84037025\_1917-04-14\_ed-1\_seq-1\_ocr.txt ’ refers to the OCR text file of the first page of the first edition of _La Rassegna_ published on 14 April 1917.&#x20;

_kiara_ enables us to retrieve both the text files and the metadata embedded in their filenames. This is very valuable for historical research, as it helps maintain transparency and traceability in how we access and process our sources.

## Downloading the text corpus

kiara has built-in operations for downloading files.

Let's list what is available:

```
kiara.list_operation_ids('download')
```

You will see:

```
['download.file',
 'download.file.from.github',
 'download.file.from.zenodo',
 'download.file_bundle',
 'download.file_bundle.from.github',
 'download.file_bundle.from.zenodo']
```

Since we want to download multiple files, let's have a look at `download.file_bundle`&#x20;

```
kiara.retrieve_operation_info('download.file_bundle')
```

&#x20;From this, we learn:&#x20;

Inputs\
Required:

* `url` – the URL of an archive (e.g., a ZIP file) to download.

Optional:

* `sub_path` – a relative path inside the archive to extract only a specific subfolder or section.

Outputs

* A `file_bundle` object containing the downloaded files.
* `download_metadata` – a dictionary with information about the download process.

So, we need to define the inputs, use `kiara.run_job` with our chosen operation `download.file_bundle` , and store this as our outputs.

To do that, run the following:

```
inputs = {
    "url": "https://github.com/DHARPA-Project/kiara.examples/archive/refs/heads/main.zip",
    "sub_path": "kiara.examples-main/examples/data/text_corpus/data"
}

outputs = kiara.run_job('download.file_bundle', inputs=inputs, comment="")
outputs
```

kiara provides two outputs:

`file_bundle` shows the group of downloaded text files.&#x20;

`download_metadata` shows metadata about the downloaded files.

Now, let's save the bundle of text files for later use:

```
file_bundle = outputs['file_bundle']
```

## Preparing the texts for analysis

Now that we have downloaded the files, we want to give them structure. Working with raw text files is limiting, especially for analysis.

\
Instead, we will convert them into a tabular format, where each row represents a document, and each column contains specific information (like file name or content).

kiara provides the `create.table.from.file_bundle` function for this. Let’s explore all the  available operations:

```
kiara.retrieve_operation_info('create.table.from.file_bundle')
```

This function:

* Inputs a `file_bundle`
* Outputs a table with at least three columns:
  * `id`: A unique identifier for each file
  * `rel_path`: The file’s relative path (including date and edition info)
  * `content`: The full text of the file

Now, let's use it.

```
inputs = {
    'file_bundle': file_bundle
}

outputs = kiara.run_job('create.table.from.file_bundle', inputs=inputs, comment="")
outputs
```

The files have been turned into a table, which makes it easier to investigate what is in the corpus.&#x20;

Most importantly, we now have a column named `content` that holds the OCRed text from each file - this is what we will use for Natural Language Processing (NLP).

## Extracting the text content

As we are interested in the content of one column, we can use kiara operations to pick the column we want and extract its content.

First, let’s list the available table operations:

```
kiara.list_operation_ids('table')
```

Now, we can use the one we want:

```
kiara.retrieve_operation_info('table.pick.column')
```

This operation takes in a `table` and a `column_name` and returns an `array` containing all the values from that column.

In our case, we want the `'content'` column from our table. So, let's run the following:

```
inputs = {
    'table': table,
    'column_name': 'content'
}

outputs = kiara.run_job('table.pick.column', inputs=inputs)
outputs
```

This will return an array that looks like this:

```
│                                                                                                                                          │
│   field   value                                                                                                                          │
│  ──────────────────────────────────────────────────────────────────────────────────────────────────                                      │
│   array                                                                                                                                  │
│             LA RAGIONE                                                                                                                   │
│             LA RAG ONE                                                                                                                   │
│             LA RAGIONE                                                                                                                   │
│             contro i vili, i camorristi, i sicari, i falsari e gli austriacanti, nemici dell ...                                         │
│             contro i vili, i camorristi, i sicari, i falsari e gli austriacanti, nemici dell ...                                         │
│             LA RAGIONA                                                                                                                   │
│             LA RAGIONE                                                                                                                   │
│             LA RAGIONE                                                                                                                   │
│             contro i vili, i camorristi, i sicari, i falsari e gli austriacanti, nemici dell ...                                         │
│             LA RAG ONE                                                                                                                   │
│             contro 1 vili, i camorristi, i sicari, i falsari e gli austriacanti, nemici dell ...                                         │
│             ■■■                                                                                                                          │
│             La Rassegna                                                                                                                  │
│             Both Phones                                                                                                                  │
│             ■ jSrìt** W?? iIK 38®f- i^M                                                                                                  │
│             ■Both Phones                                                                                                                  
```

##

##
