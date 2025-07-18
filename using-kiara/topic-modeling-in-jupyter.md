# Topic modeling in Jupyter

## Topic modeling with kiara

Now let's move onto another form of digital analysis possible using kiara: **topic modeling.**

## Why topic modeling?

This analytical tool works best with large collections of unstructured text (i.e., without any machine-readable annotations) and when the main purpose is to obtain a general overview of the topics discussed in a corpus. A topic is understood as a set of terms that occur together in a statistically significant way to form a cluster of words and, as long as it is unstructured, the corpus can be just about anything (e.g., emails, newspaper headlines, a standard .txt document). For this, TM is an excellent distant reading technique that may be used as a data exploration method, for instance to categorise documents within a collection without having to read them all. Its potential, however, is most fully reached when working in tandem with expert knowledge.

There are many variations of the TM algorithm and numerous programs and techniques to implement them. The rationale behind all of them, however, is the same: using statistical modelling to discover topics in a textual collection. Among these very many techniques, Latent Dirichlet Allocation (LDA - [Blei, Ng and Jordan 2003](http://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf)) is perhaps the most widely used. Kiara uses Gensim, which is one way of using LDA with Python.

## Activate your kiara environment

Using a new CLI window, type the following to activate your previously created kiara environment (replacing `kiara_explore` with whatever name you assigned it):

```
conda activate kiara_explore
```

You can also create separate [environments](../installation/mac.md#creating-and-activating-an-environment) for different kiara projects if you want – but keep in mind that your kiara contexts will be available across all environments.

> Tip: to check what environments you have created in the past, you can use `conda env list`

## Dependencies

You already installed some basic plugins when setting up your kiara environment, and you installed Jupyter notebook and Observable when completing [Basic data processing in Jupyter](basic-data-processing-in-jupyter.md). Now you should install the necessary packages for topic modeling, too. Run:

```
pip install git+https://github.com/DHARPA-Project/kiara_plugin.topic_modelling
```

> Tip: to check what packages are already installed in your environment, use `conda list`

## Open Jupyter notebook and set up kiara API

To open the Jupyter interface, run:

```
jupyter notebook
```

To use kiara in Jupyter Notebook, you first need to create a `kiaraAPI` **instance**. This instance allows us to control kiara and see what operations are available. Run the following code in a notebook cell:

```
from kiara.api import KiaraAPI

kiara = KiaraAPI.instance()
```

## Create a project context

As you saw earlier, kiara uses different [**contexts**](../before-you-begin/key-concepts.md#context) for specific projects. To create and use a **new context** for your topic modelling project, e.g. `project3_TM,` run the following code:

```
kiara.set_active_context(context_name='project3_TM', create=True)

print('Available Contexts:', kiara.list_context_names())
print('Current Context:', kiara.get_current_context_name())
```

This operation will also show all your available contexts and confirm which one is currently active. The output will be something like:&#x20;

```
Available Contexts: ['default', 'project1_DP', 'project2_NA', 'project3_TM']
Current Context: project3_TM
```

This confirms that your new context is set up and ready to use.&#x20;

## Data onboarding

Before running topic modeling, you must first onboard your corpus. Kiara offers three options for loading textual data, depending on where your files are stored. The third option uses example data present in the topic modelling plugin.&#x20;

Choose **one** of the following options:

### Option 1: Onboard texts from Zenodo

Use this method if your text files are archived on [Zenodo](https://zenodo.org/). The operation `topic_modelling.create_table_from_zenodo` retrieves a ZIP archive from Zenodo using its DOI and extracts its contents into a table with two columns: `file_name` and `content`.

Run the following:

```
create_table_from_zenodo_inputs = {
    "doi": "4596345",
    "file_name": "ChroniclItaly_3.0_original.zip"
}
create_table_from_zenodo_results = kiara.run_job('topic_modelling.create_table_from_zenodo', inputs=create_table_from_zenodo_inputs, comment= " ")
corpus_table_zenodo = create_table_from_zenodo_results['corpus_table']
create_table_from_zenodo_results
```

The resulting table contains the name of each file and its corresponding text content, ready for further processing.

### Option 2: Onboard texts from GitHub

If your files are hosted in a public GitHub repository, you can use the operation `create.table_from_github_files` to download and structure the data. Provide the repository owner, name, and path to the folder containing your text files.

Run the following:

```
create_table_from_github_files_inputs = {
    "download_github_files__user": "DHARPA-Project",
    "download_github_files__repo": "kiara.examples",
    "download_github_files__sub_path": "kiara.examples-main/examples/workshops/dh_benelux_2023/data",
    "download_github_files__include_files": ["txt"]
}
create_table_from_github_files_results = kiara.run_job('create.table_from_github_files', inputs=create_table_from_github_files_inputs, comment=" ")
create_table_from_github_files_results
```

This method creates a kiara table from the selected `.txt` files, alongside a downloadable file bundle for inspection or archival.

### Option 3: Onboard texts from a local folder

To use text files stored locally on your machine, run the operation `import.table.from.local_folder_path`. This imports all text files from a specified directory and creates a table similar to the above options.

Run the following:

```
import_table_from_local_folder_inputs = {
    "path": "/Users/mariella.decrouychan/Documents/GitHub/kiara_plugin.topic_modelling/tests/resources/data/text_corpus/data"
}
import_table_from_local_folder_results = kiara.run_job('import.table.from.local_folder_path', inputs=import_table_from_local_folder_inputs, comment=" ")
import_table_from_local_folder_results
```

Make sure to replace the `path` with the actual location of your text corpus. The resulting table contains metadata and full content for each text file.

## Subset creation

After onboarding your corpus, the next step is to enrich it with metadata, explore its temporal distribution, and optionally filter it to create a more focused subset for analysis.

### Extract metadata from filenames

To begin, we extract metadata such as publication identifiers and dates directly from the file names with the `topic_modelling.lccn_metadata` operation. This helps structure the dataset for further filtering and analysis.

Run the following operation to extract the metadata and append it to your corpus table:

```
lccn_metadata_inputs = {
    "corpus_table": import_table_from_local_folder_results['table'],
    "column_name": "file_name",
    "map": [["sn84037024","sn84037025"],["La Ragione","La Rassegna"]]   
}
lccn_metadata_results = kiara.run_job('topic_modelling.lccn_metadata', inputs=lccn_metadata_inputs, comment = " ")
lccn_metadata_results
```

This will add three new columns to your table: `date`, `publication_reference`, and `publication_name`, based on patterns identified in the file names.

### Visualize corpus distribution

To understand how your documents are distributed over time and by publication, you can group the corpus by time periods with the `topic_modelling.corpus_distribution` operation.&#x20;

Run the following to compute the distribution:

```
corpus_dist_inputs = {
    "corpus_table": lccn_metadata_results["corpus_table"],
    "periodicity": "month",
    "date_col": "date",
    "publication_ref_col": "publication_name",
}
corpus_dist_results = kiara.run_job('topic_modelling.corpus_distribution', inputs=corpus_dist_inputs, comment = " ")
corpus_dist_results['dist_table'].data
```

This operation returns a table (`dist_table`) and a list (`dist_list`) summarizing how many texts exist per publication and time.&#x20;

You can visualize the results using observable notebooks:

```
from observable_jupyter import embed
embed('@dharpa-project/timestamped-corpus', cells=['viewof chart', 'style'], inputs={"data":corpus_dist_results['dist_list'].data.list_data,"scaleType":'height', "timeSelected":'month'})
```

### Create a subset of the corpus

Once you’ve explored the distribution, you can run the `query.table` operation to filter the corpus based on specific criteria (e.g., a time range) using SQL queries.

To create a subset of documents published in 1917, run the following:

```
date_ref_1 = "1917-1-1"
date_ref_2 = "1917-12-31"
query = f"SELECT * FROM corpus_table WHERE CAST(date AS DATE) <= DATE '{date_ref_2}' AND CAST(date AS DATE) > DATE '{date_ref_1}'"
inputs = {
    'query' : query,
    'table': lccn_metadata_results['corpus_table'],
    'relation_name': "corpus_table"
}

subset = kiara.run_job('query.table', inputs=inputs, comment = " ")
subset
```

This filtered table (`query_result`) can now be used as input for subsequent topic modeling steps.

## Tokenize corpus

With your subset ready, the next step is to convert each document into a list of tokens (words or characters), which will be the basis for topic modeling. This section walks through the process of extracting text content from the corpus, tokenizing it, and applying basic preprocessing steps.

### Extract text content as an array

To prepare the corpus for tokenization, you first need to extract the column that contains the text content and convert it into an array.&#x20;

Run the operation `table.pick.column` to extract the `content` column from the table:

```
pick_column_inputs = {
    "table": import_table_from_local_folder_results['table'],
    "column_name": "content"   
}
pick_column_results = kiara.run_job('table.pick.column', inputs=pick_column_inputs, comment = " ")
```

This returns the text contents as an array, which can now be tokenized.

### Tokenize the text

Next, tokenize the array using the operation `topic_modelling.tokenize_array`. By default, tokenization is done by word (not by character).

Run the following:

```
tokenize_array_inputs = {
    "corpus_array": pick_column_results['array'],
    "column_name": "content"   
}
tokenize_array_results = kiara.run_job('topic_modelling.tokenize_array', inputs=tokenize_array_inputs, comment= " ") 
tokenize_array_results
```

This operation returns an array where each entry corresponds to a list of tokens (words) extracted from the respective document.

### Preprocess the tokens

To clean and standardize the tokens, use the operation `topic_modelling.preprocess_tokens`. This step is optional but recommended, especially for removing punctuation, digits, and very short tokens.

Run the following to lowercase all tokens, keep only alphabetic tokens, and filter out those shorter than three characters:

```
preprocess_tokens_inputs = {
    "tokens_array": tokenize_array_results['tokens_array'],
    "lowercase": True,
    "isalpha": True,
    "min_length": 3,  
}
preprocess_tokens_results = kiara.run_job('topic_modelling.preprocess_tokens', inputs=preprocess_tokens_inputs, comment= " ")
preprocess_tokens_results
```

This will return a cleaned array of token lists, ready to be used for training the topic model.

## Remove stopwords

Stopwords are common words (such as _and_, _the_, etc.) that usually carry little semantic weight in topic modeling. Removing them helps the model focus on the more meaningful vocabulary of your corpus.

### Create a stopwords list

To begin, you can generate a list of stopwords using the operation `topic_modelling.stopwords_list`. This operation allows you to combine standard stopword lists from the Natural Language Toolkit (NLTK) with any custom stopwords relevant to your project.

Run the following to create a stopword list in both English and Italian, with a few additional custom entries:

```
stopwords_list_inputs = {
    "languages": ["english","italian"],
    "stopwords_list": ["test","test"]  
}
stopwords_list_results = kiara.run_job('topic_modelling.stopwords_list', inputs=stopwords_list_inputs, comment= " ")
stopwords_list_results
```

The result is a combined list of stopwords that will be used in the next step to filter your tokenized texts.

### Remove stopwords from the tokens

Now that you have a stopword list, you can remove those words from your preprocessed tokens using the operation `topic_modelling.remove_stopwords`.

Run the following:

```
remove_stopwords_inputs = {
    "tokens_array": preprocess_tokens_results['tokens_array'],
    "stopwords_list": stopwords_list_results["stopwords_list"] 
}
remove_stopwords_results = kiara.run_job('topic_modelling.remove_stopwords', inputs=remove_stopwords_inputs, comment= " ")
remove_stopwords_results
```

This returns a cleaned array of tokens, free from common and custom stopwords. These filtered tokens are now ready to be used in the topic modeling stage.

## Create bigrams

To improve the coherence of your topic modeling results, you can create bigrams from your preprocessed and stopword-filtered tokens. This step helps detect commonly co-occurring word pairs, _"digital\_humanities"_, and treats them as single tokens in the topic modeling process.

To generate bigrams, run the following command:

```
bigrams_inputs = {
    "tokens_array": remove_stopwords_results['tokens_array'],
    "min_count": 3,
}
bigrams_results = kiara.run_job('topic_modelling.get_bigrams', inputs=bigrams_inputs, comment= " ")
bigrams_results
```

This operation uses the `topic_modelling.get_bigrams` module and accepts optional parameters such as `min_count` (minimum frequency of token pairs) and `threshold` (a score threshold for forming phrases, through not provided in the code above). The output is a token array containing the generated bigrams.

## Topic modeling with LDA

After generating bigrams, you can proceed to apply Latent Dirichlet Allocation (LDA) to detect latent thematic structures in the corpus. kiara provides two module options for LDA:

### LDA Multicore

The `topic_modelling.lda` module wraps Gensim’s `LdaMulticore` implementation and is generally faster on multicore machines than the standard LDA implementation. However, it does not expose all LDA parameters.

To run LDA using the multicore implementation, use the following code:

```
lda_inputs = {
    "tokens_array": bigrams_results['tokens_array'],
    "num_topics": 3,
    "passes": 20,
    "chunksize": 30 
}
lda_results = kiara.run_job('topic_modelling.lda', inputs=lda_inputs, comment= " ")
lda_results
```

The results include the top 15 most frequent words and the generated topics.

### LDA with extended parameters

For more flexibility, you can use the `topic_modelling.lda_extended_params` module, which allows configuration of additional parameters such as alpha/eta tuning, topic coherence methods, and minimum topic probability thresholds.

To run this version, use:

```
lda_ext_params_inputs = {
    "tokens_array": bigrams_results['tokens_array'],
    "passes": 20,
    "chunksize": 30,
    "num_topics": 3,
    "alpha": True,
    "eta": True,
}
lda_ext_params_results =  kiara.run_job('topic_modelling.lda_extended_params', inputs=lda_ext_params_inputs, comment= " ")
lda_ext_params_results
```

The output includes:

* `most_common_words`: Top frequent tokens across the corpus.
* `print_topics`: A list of generated topic descriptions.
* `top_topics`: Topic descriptions with coherence scores.

To trace the entire pipeline—from importing your local file to tokenization, stopword removal, bigram creation, and finally LDA—you can inspect the **lineage** of the result:

```
lda_results['print_topics'].lineage
```

This command returns a detailed tree structure showing how the output was generated. It includes every kiara operation used, along with their parameters and input/output relationships. This is particularly useful for:

* Debugging or validating each stage of processing
* Reproducing or modifying specific parts of the pipeline
* Documenting your research for transparency and reuse

### Test model coherence depending on the number of topics <a href="#id-5.3.-test-model-coherence-depending-on-number-of-topics" id="id-5.3.-test-model-coherence-depending-on-number-of-topics"></a>

To compare how well different LDA models fit your data depending on the number of topics, use the `topic_modelling.lda_coherence` operation.

This module allows you to test multiple topic numbers and returns:

* **Coherence scores** for each model, which give a quantitative estimate of how interpretable or semantically consistent the topics are.
* The corresponding **printout of topics** for each number of topics tested.

Run the following to evaluate model coherence across two different topic numbers:

```
lda_coherence_inputs = {
    "tokens_array": bigrams_results['tokens_array'],
    "num_topics_list": [2,5],
    "passes": 20,
    "chunksize": 30,
    "num_topics": 3,
    "alpha": True,
    "eta": True,
}
lda_coherence_results =  kiara.run_job('topic_modelling.lda_coherence', inputs=lda_coherence_inputs, comment= " ")
lda_coherence_results
```

The `coherence_scores` helps you choose the optimal number of topics for interpretation, while the `print_topics` output displays topic-word distributions.
