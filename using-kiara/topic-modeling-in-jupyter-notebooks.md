# Topic modeling in Jupyter notebooks

## Set up kiara

Before we begin, we must ensure that Kiara and its plugins are correctly installed.

To set up kiara, run the following code:

```
try:
    from kiara_plugin.jupyter import ensure_kiara_plugins
except:
    import sys
    print("Installing 'kiara_plugin.jupyter'...")
    !{sys.executable} -m pip install -q kiara_plugin.jupyter
    from kiara_plugin.jupyter import ensure_kiara_plugins

ensure_kiara_plugins()

from kiara import KiaraAPI
kiara = KiaraAPI.instance()
```

This will check (and if needed, install) everything required to use Kiara inside your notebook.

## Choose a text corpus

Now that the environment is ready, we need a small collection of texts to work with.

For this tutorial, we use a sample of OCRed (machine-readable) front pages of _La rassegna_, an Italian-language newspaper published in the United States from 1898 to 1936. This sample is part of a larger corpus in an open-access digital heritage project, ChroniclItaly 3.0  (Viola and Fiscarelli, 2021; Viola, 2021). The front pages of _La rassegna_ were collected via Chronicling America, a searchable database of newspapers published in the US maintained by the Library of Congress. These texts are a good example for topic modeling, because they contain metadata information, e.g., the publication date.&#x20;

The file name structure is: LCCNnumber\_date\_pageNumber\_ocr.txt. Therefore, the file name ‘sn84037025\_1917-04-14\_ed-1\_seq-1\_ocr.txt ’ refers to the OCR text file of the first page of the first edition of _La Rassegna_ published on 14 April 1917.&#x20;

_kiara_ enables us to retrieve both the text files and the metadata embedded in their filenames. This is very valuable for historical research, as it helps maintain transparency and traceability in how we access and process our sources.

## Find the right download operation

kiara has built-in operations for downloading files.

Let's list what is available:

```
kiara.list_operation_ids('download')
```

You will see:

```
['download.file', 'download.file_bundle']
```

Since we want multiple files, we choose:

```
download.file_bundle
```

## Download the text corpus

Now we’ll download the sample text files from an archive.

To do that, run the following:

```
inputs = {
    "url": "https://github.com/DHARPA-Project/kiara.examples/archive/refs/heads/main.zip",
    "sub_path": "kiara.examples-main/examples/data/text_corpus/data"
}

outputs = kiara.run_job('download.file_bundle', inputs=inputs)
outputs
```

## Understand the output

kiara provides two outputs:

`file_bundle` shows the group of downloaded text files.&#x20;

`download_metadata` shows metadata about the downloaded files.

## Save the text bundle

Now, let's save the bundle of text files for later use:

```
file_bundle = outputs['file_bundle']
```

## Prepare the texts for analysis

Now that we’ve downloaded the files, we want to give them structure. Working with raw text files is limiting, especially for analysis.

\
Instead, we’ll convert them into a tabular format, where each row represents a document, and each column contains specific information (like file name or content).

kiara provides a function for this:

```
create.table.from.file_bundle
```

Let’s explore the following operation:

```
kiara.retrieve_operation_info('create.table.from.file_bundle')
```

This operation:

* Takes in a `file_bundle`
* Outputs a table with at least three columns:
  * `id`: A unique identifier for each file
  * `rel_path`: The file’s relative path (including date and edition info)
  * `content`: The full text of the file

Now, let's use it.

```
inputs = {
    'file_bundle': file_bundle
}

outputs = kiara.run_job('create.table.from.file_bundle', inputs=inputs)
outputs
```

The files have been turned into a table, which makes it easier to investigate what’s in the corpus.&#x20;

Most importantly, we now have a column named `content` that holds the OCRed text from each file - this is what we will use for Natural Language Processing (NLP).

## Extract the text content

As we are interested in the content of one column, we can use kiara operations to pick the column we want and extract its content.

First, let’s list the available table operations:

```
kiara.list_operation_ids('table')
```

Now, we can use the one we want:

```
kiara.retrieve_operation_info('table.pick.column')
```

For this operation, we need two inputs: the table we just made and the name of the column we want to pick.

In our case, we want the `'content'` column from our table.

Run the following:

```
inputs = {
    'table': table,
    'column_name': 'content'
}

outputs = kiara.run_job('table.pick.column', inputs=inputs)
outputs
```

This gives us a new output field:

```
text_array = outputs['array']
```

