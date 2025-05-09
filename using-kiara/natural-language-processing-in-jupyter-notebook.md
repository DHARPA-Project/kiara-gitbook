# Natural Language Processing in Jupyter Notebook

## Setting up kiara

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

## Choosing a text corpus

Now that the environment is ready, we need a small collection of texts to work with.

For this tutorial, we use a sample of OCRed (machine-readable) front pages of _La rassegna_, an Italian-language newspaper published in the United States from 1898 to 1936. This sample is part of a larger corpus in an open-access digital heritage project, ChroniclItaly 3.0  (Viola and Fiscarelli, 2021; Viola, 2021). The front pages of _La rassegna_ were collected via Chronicling America, a searchable database of newspapers published in the US maintained by the Library of Congress. These texts are a good example for topic modeling, because they contain metadata information, e.g., the publication date.&#x20;

The file name structure is: LCCNnumber\_date\_pageNumber\_ocr.txt. Therefore, the file name ‘sn84037025\_1917-04-14\_ed-1\_seq-1\_ocr.txt ’ refers to the OCR text file of the first page of the first edition of _La Rassegna_ published on 14 April 1917.&#x20;

_kiara_ enables us to retrieve both the text files and the metadata embedded in their filenames. This is very valuable for historical research, as it helps maintain transparency and traceability in how we access and process our sources.

## Finding the right download operation

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

## Understanding the output

kiara provides two outputs:

`file_bundle` shows the group of downloaded text files.&#x20;

`download_metadata` shows metadata about the downloaded files.

## Saving the text bundle

Now, let's save the bundle of text files for later use:

```
file_bundle = outputs['file_bundle']
```

## Preparing the texts for analysis

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

## Natural Language Processing with kiara

Now that we have extracted the text content from our sources, we are ready to begin preparing it for computational analysis.

We will walk through:

* Tokenizing the text (splitting it into words)
* Cleaning and standardizing it
* Running topic modeling (LDA) to extract thematic patterns

## Exploring NLP operations

Kiara includes a set of operations specifically designed for natural **l**anguage processing, available through the `kiara_plugin.language_processing` package.

Let’s see what is included:

```
infos = metadata = kiara.retrieve_operations_info()
operations = {}
for op_id, info in infos.item_infos.items():
    if info.context.labels.get("package", None) == "kiara_plugin.language_processing":
        operations[op_id] = info

print(operations.keys())
```

Expected output:

```
dict_keys([
  'create.stopwords_list',
  'generate.LDA.for.tokens_array',
  'preprocess.tokens_array',
  'remove_stopwords.from.tokens_array',
  'tokenize.string',
  'tokenize.texts_array'
])
```

We will begin with `tokenize.texts_array`.

## Tokenizing the text

Tokenization is the process of splitting text into individual words (or "tokens"). It is a crucial first step before any further analysis.

Let’s explore the operation:

```
kiara.retrieve_operation_info('tokenize.texts_array')
```

We will run the operation using our `text_array` from earlier:

```
inputs = {
    'texts_array': text_array
}

outputs = kiara.run_job('tokenize.texts_array', inputs=inputs)
tokens_array = outputs['tokens_array']
```

Now, each text has been transformed into a list of tokens:

```
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                          │
│   field          value                                                                                                                   │
│  ─────────────────────────────────────────────────────────────────────────────────────────────────────────                               │
│   tokens_array                                                                                                                           │
│                    ['LA', 'RAGIONE', 'ORGANO', 'DI', 'DIFESA', 'DELLA', "ITALIANITÀ'", 'contro', '1 ...                                  │
│                    ['LA', 'RAG', 'ONE', 'contro', 'i', 'vili', ',', 'i', 'camorristi', ',', 'i', 's ...                                  │
│                    ['LA', 'RAGIONE', 'ORGANO', 'DI', 'DIFESA', 'DELLA', 'ITALIANITÀ', 'contro', 'i' ...                                  │
│                    ['contro', 'i', 'vili', ',', 'i', 'camorristi', ',', 'i', 'sicari', ',', 'i', 'f ...                                  │
│                    ['contro', 'i', 'vili', ',', 'i', 'camorristi', ',', 'i', 'sicari', ',', 'i', 'f ...                                  │
│                    ['LA', 'RAGIONA', 'ORGANO', 'DI', 'DIFESA', 'DELLA', 'ITALIANITÀ', 'contro', 'i' ...                                  │
│                    ['LA', 'RAGIONE', 'ORGANO', 'DI', 'DIFESA', 'DELLA', "ITALIANITÀ'", 'contro', 'i ...                                  │
│                    ['LA', 'RAGIONE', 'contro', 'i', 'vili', ',', '1', 'camorristi', ',', 'i', 'sica ...                                  │
│                    ['contro', 'i', 'vili', ',', 'i', 'camorristi', ',', 'i', 'sicari', ',', 'i', 'f ...                                  │
│                    ['LA', 'RAG', 'ONE', 'ORGANO', 'DI', 'DIFESA', 'DELLA', 'ITALIANITÀ', "''", 'con ...                                  │
│                    ['contro', '1', 'vili', ',', 'i', 'camorristi', ',', 'i', 'sicari', ',', 'i', 'f ...                                  │
│                    ['■■■', 'La', 'Rassegna', '_', 'I', 'Both', 'Phones', 'ANNO', 'L', 'No', '.', '1 ...                                  │
│                    ['La', 'Rassegna', 'Jjoth', 'Phones', 'ANNO', 'L', 'No', '.', '2', 'BASTA', '!', ...                                  │
│                    ['Both', 'Phones', 'ANNO', 'I', '.', 'No', '.', '2', 'BASTA', '!', '...', 'uà',  ...                                  │
│                    ['■', 'jSrìt', '*', '*', 'W', '?', '?', 'iIK', '38®f-', 'i^M', 'F', '<', '5É', ' ...                                  │
│                    ['■Both', 'Phones', 'ANNO', '11', '.', 'No', '.', '5', 'LE', 'COSE', 'A', 'POSTO ...                                  │
│                                                                                                                                    
```

## Preprocessing the tokens

We will now clean the tokens using `preprocess.tokens_array`. This function allows us to apply several  operations:

* Lowercasing
* Removing stopwords
* Filtering out punctuation, numbers, or short tokens

Let’s start by removing a few stopwords (common words with little semantic value).

```
stopword_list = ['la', 'i']

inputs = {
    'tokens_array': outputs['tokens_array'],
    'remove_stopwords' : stopword_list
}

outputs = kiara.run_job('preprocess.tokens_array', inputs=inputs)
outputs
```

Now, let’s combine other preprocessing steps in a single operation:

```
inputs = {
    'tokens_array': outputs['tokens_array'],
    'to_lowercase' : True,
    'remove_non_alpha' : True
}

outputs = kiara.run_job('preprocess.tokens_array', inputs=inputs)
outputs
```

Our texts are now lowercased, cleaned of special characters, and free of some basic stopwords.

## Topic modelling with LDA

With our clean token arrays, we’re ready to perform topic modeling with Latent Dirichlet Allocation (LDA) using the `generate.LDA.for.tokens_array` operation.&#x20;

Let's explore the operation:

```
kiara.retrieve_operation_info('generate.LDA.for.tokens_array')
```

LDA reveals hidden themes across a collection of documents by grouping frequently co-occurring words.

We’ll run the model using the default setting (7 topics):&#x20;

```
inputs = {
    'tokens_array': tokens_array
}

outputs = kiara.run_job('generate.LDA.for.tokens_array', inputs=inputs)
outputs 
```

This is the expected output:&#x20;

```
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                                                                          │
│   field             value                                                                                                                │
│  ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────  │
│   coherence_map                                                                                                                          │
│                       dict data     {}                                                                                                   │
│                       dict schema   {                                                                                                    │
│                                       "title": "dict",                                                                                   │
│                                       "type": "object"                                                                                   │
│                                     }                                                                                                    │
│                                                                                                                                          │
│   coherence_table   -- none/not set --                                                                                                   │
│   topic_models                                                                                                                           │
│                       dict data     {                                                                                                    │
│                                       "7": [                                                                                             │
│                                         [                                                                                                │
│                                           0,                                                                                             │
│                                           "0.031*\"di\" + 0.024*\"e\" + 0.017*\"che\" + 0.015*\"il\" + 0.013*\"non\" + 0.012*\"a\" …     │
│                                         ],                                                                                               │
│                                         [                                                                                                │
│                                           1,                                                                                             │
│                                           "0.043*\"di\" + 0.027*\"e\" + 0.025*\"che\" + 0.017*\"il\" + 0.016*\"a\" + 0.016*\"non\" …     │
│                                         ],                                                                                               │
│                                         [                                                                                                │
│                                           2,                                                                                             │
│                                           "0.023*\"di\" + 0.022*\"e\" + 0.021*\"che\" + 0.014*\"a\" + 0.011*\"per\" + 0.011*\"il\" …     │
│                                         ],                                                                                               │
│                                         [                                                                                                │
│                                           3,                                                                                             │
│                                           "0.043*\"di\" + 0.028*\"e\" + 0.026*\"che\" + 0.019*\"il\" + 0.016*\"a\" + 0.013*\"non\" …     │
│                                         ],                                                                                               │
│                                         [                                                                                                │
│                                           4,                                                                                             │
│                                           "0.025*\"di\" + 0.020*\"che\" + 0.018*\"e\" + 0.016*\"a\" + 0.013*\"un\" + 0.012*\"il\" +…     │
│                                         ],                                                                                               │
│                                         [                                                                                                │
│                                           5,                                                                                             │
│                                           "0.030*\"di\" + 0.019*\"e\" + 0.016*\"che\" + 0.016*\"il\" + 0.011*\"un\" + 0.011*\"a\" +…     │
│                                         ],                                                                                               │
│                                         [                                                                                                │
│                                           6,                                                                                             │
│                                           "0.029*\"di\" + 0.018*\"e\" + 0.013*\"che\" + 0.012*\"il\" + 0.010*\"si\" + 0.009*\"per\"…     │
│                                         ]                                                                                                │
│                                       ]                                                                                                  │
│                                     }                                                                                                    │
│                       dict schema   {                                                                                                    │
│                                       "title": "dict",                                                                                   │
│                                       "type": "object"                                                                                   │
│                                     }                                                                                                    │
│                                                                                                                                          │
│                                                                                                                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

The output shows:

* `topic_models`: The list of topics with key terms and weights
* `coherence_map` and `coherence_table`: Optional evaluation scores

## Recording and tracing the data

We've successfully downloaded, organised, and pre-processed our text files, and now generated some topics for it.

This means that we have already made several interpretative and technical decisions — from removing stopwords to setting parameters for topic modeling.

In humanities research, documenting how we transform sources is as important as the research results themselves. kiara enables us to trace our entire data lineage — every transformation, operation, and input value is automatically recorded.

This makes our research process transparent, traceable, and reproducible.

Let’s say we’ve stored the output from our topic modeling as `topics`. We can view the lineage of how these topics were created by running the following:

```
topics = outputs['topic_models']

topics.lineage
```

This is the expected output:

```
generate.LDA.for.tokens_array
├── input: compute_coherence (boolean) = 5137a237-fe0f-45bd-abe3-cc84700a2bb6
├── input: num_topics_max (integer) = 75399d70-bbef-4215-b9b5-5dacfa03b2ba
├── input: num_topics_min (integer) = 75399d70-bbef-4215-b9b5-5dacfa03b2ba
├── input: tokens_array (array) = 02d01eb7-70d6-4ef7-811e-66ed25f920bb
│   └── preprocess.tokens_array
│       ├── input: remove_all_numeric (boolean) = 5137a237-fe0f-45bd-abe3-cc84700a2bb6
│       ├── input: remove_alphanumeric (boolean) = 5137a237-fe0f-45bd-abe3-cc84700a2bb6
│       ├── input: remove_non_alpha (boolean) = 8b1b93ec-a51e-4bbd-84cf-c5a1efd78e9b
│       ├── input: remove_short_tokens (integer) = f5df1b36-9884-413d-92d0-81209227f106
│       ├── input: remove_stopwords (list) = bb8a79b2-369c-46ae-a85a-2b0f85c9da22
│       ├── input: to_lowercase (boolean) = 8b1b93ec-a51e-4bbd-84cf-c5a1efd78e9b
│       └── input: tokens_array (array) = d1db365d-2e59-4455-ae05-78447e5a4268
│           └── preprocess.tokens_array
│               ├── input: remove_all_numeric (boolean) = 5137a237-fe0f-45bd-abe3-cc84700a2bb6
│               ├── input: remove_alphanumeric (boolean) = 5137a237-fe0f-45bd-abe3-cc84700a2bb6
│               ├── input: remove_non_alpha (boolean) = 5137a237-fe0f-45bd-abe3-cc84700a2bb6
│               ├── input: remove_short_tokens (integer) = f5df1b36-9884-413d-92d0-81209227f106
│               ├── input: remove_stopwords (list) = 524b5812-c4df-4ea0-a50a-d0ec5166c22f
│               ├── input: to_lowercase (boolean) = 5137a237-fe0f-45bd-abe3-cc84700a2bb6
│               └── input: tokens_array (array) = a3c66f00-7f67-483d-8018-a64714094fa4
│                   └── tokenize.texts_array
│                       ├── input: texts_array (array) = 3db76a98-88e6-45ee-8618-7c95fdf8232c
│                       │   └── table.pick.column
│                       │       ├── input: column_name (string) = 33ebce29-be63-4644-b66b-9e82a3c56236
│                       │       └── input: table (table) = bd56aae9-6289-4f3e-b3f4-edbc55310689
│                       │           └── create.table
│                       │               └── input: file_bundle (file_bundle) = 214ae90d-224b-447a-b0e8-112024a8e6d4
│                       │                   └── download.file_bundle
│                       │                       ├── input: sub_path (string) = 89c3d000-a486-4089-9592-142253d8f3d3
│                       │                       └── input: url (string) = 10d94fa6-0c3d-4d6e-a457-9fa1e7b63e99
│                       └── input: tokenize_by_word (boolean) = 8b1b93ec-a51e-4bbd-84cf-c5a1efd78e9b
└── input: words_per_topic (integer) = cd1319e3-a6ec-4d8d-99b3-34ef873e1d13
```

This command shows:

* Every operation that was run (e.g., `tokenize.texts_array`, `preprocess.tokens_array`, `generate.LDA.for.tokens_array`)
* All input parameters used (e.g., which stopwords were removed)
* The source data (the original file)

## Building a reusable workflow

Now that we have introduced the key concepts and preprocessing steps in NLP, we will implement these processes practically using a workflow.&#x20;

This section walks through the creation of a NLP pipeline using Kiara’s `Workflow` functionality.

We begin by creating a `Workflow` object. This manages the internal state, execution history, and consistency of our processing steps.

To create a `Workflow` object, run the code:

```
doc = """Example topic-modeling end-to-end workflow."""
workflow = Workflow.create("topic_modeling", doc=doc, replace_existing_alias=True)
```

In the code above, we define a description for the workflow and assign it the alias `"topic_modeling"`, which allows us to reference it easily later. The parameter `replace_existing_alias=True` ensures that any existing workflow with the same alias will be replaced.&#x20;

## Assembling the workflow

To construct our topic modeling pipeline, we assemble a series of processing steps from kiara’s modular system. Each module handles a specific task, such as importing files, preprocessing text, or generating topics using LDA. Below, we describe each step and how they are connected together. Each step in the workflow is added first and then connected to the output of a previous step.

## The steps of the workflow

## Step: `import_text_corpus`

This module imports a folder of text files from the local filesystem as a file bundle.

```
# Creating step: import_text_corpus
workflow.add_step(operation="import.file_bundle", step_id="import_text_corpus")
```

## Step: `create_stopwords_list`

In this module, we create a list of stopwords based on selected languages or custom lists.&#x20;

```
# Creating step: create_stopwords_list
workflow.add_step(operation="create.stopwords_list", step_id="create_stopwords_list")
```

## Step: `create_text_corpus`

Here we transform the imported text files into a table structure, and then we connect the input of this step to the output of the previous step.

```
# Creating step: create_text_corpus
step_create_text_corpus_config = {'constants': {}, 'defaults': {}, 'source_type': 'text_file_bundle', 'target_type': 'table', 'ignore_errors': False}
workflow.add_step(
    operation="create.table",
    module_config=step_create_text_corpus_config,
    step_id="create_text_corpus")
```

```
# Connecting input(s) of step 'create_text_corpus'
workflow.connect_fields("create_text_corpus.text_file_bundle", "import_text_corpus.file_bundle")
```

## Step: `extract_texts_column`

We extract the column containing the textual content from the table and convert it into an array suitable for tokenization.

```
# Creating step: extract_texts_column
workflow.add_step(operation="table.cut_column", step_id="extract_texts_column")
```

```
# Connecting input(s) of step 'extract_texts_column'
workflow.connect_fields("extract_texts_column.table", "create_text_corpus.table")
```

## Step: `extract_filename_column`

Here we extract the filenames as a separate array.

<pre><code><strong># Creating step: extract_filename_column
</strong>workflow.add_step(operation="table.cut_column", step_id="extract_filename_column")
</code></pre>

```
# Connecting input(s) of step 'extract_filename_column'
workflow.connect_fields("extract_filename_column.table", "create_text_corpus.table")
```

## Step: `create_date_array`

We parse the filenames to extract date information, assuming they include date stamps.

```
# Creating step: create_date_array
workflow.add_step(operation="parse.date_array", step_id="create_date_array")
```

```
# Connecting input(s) of step 'create_date_array'
workflow.connect_fields("create_date_array.array", "extract_filename_column.array")
```

## Step: `tokenize_content`

The text array is tokenized to prepare it for further processing.

```
# Creating step: tokenize_content
workflow.add_step(operation="tokenize.texts_array", step_id="tokenize_content")
```

```
# Connecting input(s) of step 'tokenize_content'
workflow.connect_fields("tokenize_content.texts_array", "extract_texts_column.array")
```

## Step: `preprocess_corpus`

Here we apply several preprocessing techniques, such as removing stopwords, filtering short or irrelevant tokens. This helps clean the data and reduce noise before modeling.

```
# Creating step: preprocess_corpus
workflow.add_step(operation="preprocess.tokens_array", step_id="preprocess_corpus")
```

```
# Connecting input(s) of step 'preprocess_corpus'
workflow.connect_fields("preprocess_corpus.tokens_array", "tokenize_content.tokens_array")
workflow.connect_fields("preprocess_corpus.remove_stopwords", "create_stopwords_list.stopwords_list")
```

## Step: `generate_lda`

We apply  LDA to identify latent topics in the text corpus. The model is computed for different numbers of topics, and coherence scores are generated to assess model quality.

```
# Creating step: generate_lda
workflow.add_step(operation="generate.LDA.for.tokens_array", step_id="generate_lda")
```

```
# Connecting input(s) of step 'generate_lda'
workflow.connect_fields("generate_lda.tokens_array", "preprocess_corpus.tokens_array")
```

## Setting workflow input/output names (optional)

To make our workflow easier to understand and reuse, we can assign custom aliases to the input and output fields. These aliases provide more intuitive names for parameters that users may want to configure or retrieve results from.

For example, rather than referring to input fields like `extract_texts_column.column_name`, we can assign the alias `content_column_name` . Similarly, we can rename output fields for clarity. See the example below:

```
workflow.set_input_alias(input_field="extract_texts_column.column_name", alias="content_column_name")
workflow.set_input_alias(input_field="extract_filename_column.column_name", alias="filename_column_name")
workflow.set_input_alias(input_field="import_text_corpus.path", alias="text_corpus_folder_path")
workflow.set_input_alias(input_field="create_date_array.min_index", alias="date_parse_min")
workflow.set_input_alias(input_field="create_date_array.max_index", alias="date_parse_max")
workflow.set_input_alias(input_field="create_date_array.force_non_null", alias="date_force_non_null")
workflow.set_input_alias(input_field="create_date_array.remove_tokens", alias="date_remove_tokensl")
workflow.set_input_alias(input_field="tokenize_content.tokenize_by_word", alias="tokenize_by_word")
workflow.set_input_alias(input_field="generate_lda.num_topics_min", alias="num_topics_min")
workflow.set_input_alias(input_field="generate_lda.num_topics_max", alias="num_topics_max")
workflow.set_input_alias(input_field="generate_lda.compute_coherence", alias="compute_coherence")
workflow.set_input_alias(input_field="generate_lda.words_per_topic", alias="words_per_topic")
workflow.set_input_alias(input_field="create_stopwords_list.languages", alias="languages")
workflow.set_input_alias(input_field="create_stopwords_list.stopword_lists", alias="stopword_lists")
workflow.set_input_alias(input_field="preprocess_corpus.to_lowercase", alias="to_lowercase")
workflow.set_input_alias(input_field="preprocess_corpus.remove_alphanumeric", alias="remove_alphanumeric")
workflow.set_input_alias(input_field="preprocess_corpus.remove_non_alpha", alias="remove_non_alpha")
workflow.set_input_alias(input_field="preprocess_corpus.remove_all_numeric", alias="remove_all_numeric")
workflow.set_input_alias(input_field="preprocess_corpus.remove_short_tokens", alias="remove_short_tokens")
workflow.set_input_alias(input_field="preprocess_corpus.remove_stopwords", alias="remove_stopwords")


workflow.set_output_alias(output_field="import_text_corpus.file_bundle", alias="text_corpus_file_bundle")
workflow.set_output_alias(output_field="create_text_corpus.table", alias="text_corpus_table")
workflow.set_output_alias(output_field="extract_texts_column.array", alias="content_array")
workflow.set_output_alias(output_field="tokenize_content.tokens_array", alias="tokenized_corpus")
workflow.set_output_alias(output_field="preprocess_corpus.tokens_array", alias="preprocessed_corpus")
workflow.set_output_alias(output_field="generate_lda.topic_models", alias="topic_models")
workflow.set_output_alias(output_field="generate_lda.coherence_map", alias="coherence_map")
workflow.set_output_alias(output_field="generate_lda.coherence_table", alias="coherence_table")
workflow.set_output_alias(output_field="create_date_array.date_array", alias="date_array")
```

## Checking workflow status

Once all steps are added and connected, and aliases assigned, we can inspect the workflow’s current state. This allows us to verify which inputs and outputs are ready or still need to be set:

```
workflow.current_state
```

## Viewing the execution graph

To understand how many steps of the workflow pipeline are connected, we can look at the execution graph.

```
graph_to_image(workflow.pipeline.execution_graph)
```

## Setting workflow inputs

Once a workflow has an assembled pipeline, we can set its inputs. We use the input field names that we got from the result of the `workflow.current_state` call. These include the path to your text corpus, the names of relevant columns, and preprocessing options. Here’s how inputs are set and how we execture the workflow:

```
workflow.set_input("text_corpus_folder_path", "/home/markus/projects/kiara/dev/kiara.examples/examples/pipelines/topic_modeling/../../data/text_corpus/data")
workflow.set_input("content_column_name", "content")
workflow.set_input("filename_column_name", "file_name")
workflow.set_input("date_force_non_null", None)
workflow.set_input("date_parse_min", 11)
workflow.set_input("date_parse_max", 21)
workflow.set_input("date_remove_tokensl", None)
workflow.set_input("tokenize_by_word", None)
workflow.set_input("languages", ['italian'])
workflow.set_input("stopword_lists", [])
workflow.set_input("to_lowercase", None)
workflow.set_input("remove_alphanumeric", None)
workflow.set_input("remove_non_alpha", None)
workflow.set_input("remove_all_numeric", None)
workflow.set_input("remove_short_tokens", None)
workflow.set_input("num_topics_min", 7)
workflow.set_input("num_topics_max", 9)
workflow.set_input("compute_coherence", True)
workflow.set_input("words_per_topic", None)


# process all workflow steps that can be processed
workflow.process_steps()

# print the current state, after we set our inputs
workflow.current_state
```

## Printing workflow outputs

Once we have executed the workflow, we can inspect the current outputs using:

```
workflow.current_output_values
```

This code returns the key results generated by the topic modeling pipeline. These include various structured outputs that can be used for evaluation, visualization, or further analysis, such as coherence map, coherence table, topic models, etc.&#x20;

## Snapshotting the workflow

So far, our workflow only exists in memory. To save the current pipeline state for reuse or inspection in other environments, create a snapshot:

```
workflow.snapshot(save=True)
```

This saves the workflow with its current structure, inputs, and outputs. In addition, you can  retrieve and rerun the workflow under its alias (in our case: `topic_modeling`).

To list all available workflows:

```
!kiara workflow list
```

To view the structure and settings of your saved workflow:

```
!kiara workflow explain topic_modeling
```

This provides a comprehensive view of the pipeline steps, configuration parameters, input sources, and output fields.
