---
description: >-
  How to install kiara and its dependencies in a virtual environment for Mac
  users
---

# Mac

### Prerequisites&#x20;

If you haven't already, install the latest versions of:

* [miniconda](https://www.anaconda.com/docs/getting-started/miniconda/main) (a tool for managing Python and dependencies)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) (a version control system)

> Tip: We recommend using Homebrew or another package manager for installing these.

### Creating and activating an environment

Open a new Terminal window and create an environment to install kiara into. This is like giving kiara its own room, so that it will not interfere with other tools or projects on your computer. For example:

```⏎
conda create -n kiara_explore
```

You can replace `kiara_explore` with any name you choose for your environment.

Proceed by typing `y` and hitting enter, before activating the conda environment using:

```⏎
conda activate kiara_explore
```

### Install kiara and its basic plugins

Kiara is installed using a package-management system for Python called `pip`. To install pip into your environment, use:

```
conda install pip
```

Then:

```⏎
pip install kiara
```

The installation may take a few minutes. Once complete, install kiara's plugins:

```
pip install kiara_plugin.core_types kiara_plugin.onboarding kiara_plugin.tabular
```

These plugins provide support for:

* core data types
* helpful onboarding tools
* tabular data (spreadsheets, CSVs, etc.)

> Tip: to check which version of kiara or any of its plugins are installed at any point, use `pip list | grep kiara`&#x20;
