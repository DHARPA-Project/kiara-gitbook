# Topic modeling in jupyter notebook

Before you run any notebook, you will need to install kiara on your computer.&#x20;

This process is quite manageable, and I will walk you through it step by step.

You will need:

* a tool called **conda** or **miniconda** for managing Python and dependencies
* a special **environment** where kiara will live
* the **kiara** software itself, plus a few plugins

## Install conda (or miniconda)

**Conda** is a popular tool that helps you manage different versions of Python and software libraries. If you already have Conda (or Miniconda) installed, you can skip this step.

If not:

* Go to the Miniconda [download page](https://www.anaconda.com/docs/getting-started/miniconda/main).
* Download the version that matches your computer (Windows, macOS, or Linux).
* Follow the instructions to install it.

\*Double-check that you choose the correct version for your operating system and architecture (most modern computers use the 64-bit version).

## Create a kiara environment

Once Conda is installed, open your **Terminal** (on Mac or Linux) or **Command Line Interface (CLI)** (on Windows).

You are now going to create a special **environment** just for kiara. This is like giving kiara its own room, so that it will not interfere with other tools or projects on your computer.

Type this command:

```
conda create -n kiara_testing python jupyter
```

This creates an environment called `kiara_testing` and installs a recent version of Python and **Jupyter Notebook** (the tool you will use to run the notebooks).

You can replace `kiara_testing` with any name you like for your environment.

## Activate the envrionment

Next, tell Conda that you want to activate this new environment:

```
conda activate kiara_testing
```

## Install kiara

Now that your environment is set up, let's install **kiara** iteself.&#x20;

kiara is installed using a package-management system for Python called `pip`. Just type:

```
pip install kiara
```

The first installation may take a few minutes. That is normal.

## Install kiara plugins

To make kiara more useful, you will also install some basic **plugins** by running:&#x20;

```
pip install kiara_plugin.core_types kiara_plugin.onboarding kiara_plugin.tabular 
```

These plugins provide support for:

* core data types
* helpful onboarding tools
* tabular data (spreadsheets, CSVs, etc.)

To see which versions of kiara and its plugins are installed, you can run:

```
pip list | grep kiara
```

At the time of writing, the versions installed are:

```
kiara                     0.5.13
kiara_plugin.core_types   0.5.2
kiara_plugin.onboarding   0.5.2
kiara_plugin.tabular      0.5.6
```

## Install the topic modeling package

To install the **topic modeling package**, run:

```
pip install git+https://github.com/DHARPA-Project/kiara_plugin.topic_modelling
```

Alternatively, you can run:&#x20;

```
! pip install git+https://github.com/DHARPA-Project/kiara_plugin.topic_modelling
```

## Start Jupyter notebook

To open the Jupyter interface, run:

```
jupyter notebook
```

## Import kiara and create an API

Now that kiara and its plugins are installed, let's set up `kiaraAPI`.

To start using kiara in Jupyter Notebook, you first need to create a `kiaraAPI` **instance**. This instance allows us to control kiara and see what operations are available.&#x20;

To set this up, run the following code in a notebook cell:

```
from kiara.api import KiaraAPI

kiara = KiaraAPI.instance()
```

## 1. Data onboarding

Before running topic modeling, you must first onboard your corpus. Kiara offers three options for loading textual data, depending on where your files are stored. The 1.3 option uses example data present in the topic modelling plugin.&#x20;

Choose **one** of the following options:

## 1.1 Onboard texts from Zenodo

Use this method if your text files are archived on [Zenodo](https://zenodo.org/). The operation `topic_modelling.create_table_from_zenodo` retrieves a ZIP archive from Zenodo using its DOI and extracts its contents into a table with two columns: `file_name` and `content`.

Run the following:

```
create_table_from_zenodo_inputs = {
    "doi": "4596345",
    "file_name": "ChroniclItaly_3.0_original.zip"
}
create_table_from_zenodo_results = kiara.run_job(
    'topic_modelling.create_table_from_zenodo',
    inputs=create_table_from_zenodo_inputs,
    comment=" "
)
corpus_table_zenodo = create_table_from_zenodo_results['corpus_table']
```

The resulting table contains the name of each file and its corresponding text content, ready for further processing.

## Option 2: Onboard texts from GitHub

If your files are hosted in a public GitHub repository, you can use the operation `create.table_from_github_files` to download and structure the data. Provide the repository owner, name, and path to the folder containing your text files.

Run the following:

```
create_table_from_github_files_inputs = {
    "download_github_files__user": "DHARPA-Project",
    "download_github_files__repo": "kiara.examples",
    "download_github_files__sub_path": "kiara.examples-main/examples/workshops/dh_benelux_2023/data",
    "download_github_files__include_files": ["txt"]
}
create_table_from_github_files_results = kiara.run_job(
    'create.table_from_github_files',
    inputs=create_table_from_github_files_inputs,
    comment=" "
)
dl_file_bundle_gh = create_table_from_github_files_results['download_github_files__file_bundle']

```

This method creates a kiara table from the selected `.txt` files, alongside a downloadable file bundle for inspection or archival.

## Option 3: Onboard texts from a local folder

To use text files stored locally on your machine, run the operation `import.table.from.local_folder_path`. This imports all text files from a specified directory and creates a table similar to the above options.

Run the following:

```
import_table_from_local_folder_inputs = {
    "path": "/Users/mariella.decrouychan/Documents/GitHub/kiara_plugin.topic_modelling/tests/resources/data/text_corpus/data"
}
import_table_from_local_folder_results = kiara.run_job(
    'import.table.from.local_folder_path',
    inputs=import_table_from_local_folder_inputs,
    comment=" "
)

```

Make sure to replace the `path` with the actual location of your text corpus. The resulting table contains metadata and full content for each text file.

## 2. Subset creation

After onboarding your corpus, the next step is to enrich it with metadata, explore its temporal distribution, and optionally filter it to create a more focused subset for analysis.

## 2.1 Extract metadata from filenames

To begin, we extract metadata such as publication identifiers and dates directly from the file names with the `topic_modelling.lccn_metadata` operation. This helps structure the dataset for further filtering and analysis.

Run the following operation to extract the metadata and append it to your corpus table:

`lccn_metadata_inputs = {`\
`"corpus_table": import_table_from_local_folder_results['table'],`\
`"column_name": "file_name",`\
`"map": [["sn84037024", "sn84037025"], ["La Ragione", "La Rassegna"]]`\
`}`

\
`lccn_metadata_results = kiara.run_job(`\
`'topic_modelling.lccn_metadata',`\
`inputs=lccn_metadata_inputs,`\
`comment = " "`\
`)`

This will add three new columns to your table: `date`, `publication_reference`, and `publication_name`, based on patterns identified in the file names.

