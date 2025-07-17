---
description: >-
  How to install kiara and its dependencies in a virtual environment for Windows
  users
---

# Windows

### Prerequisites&#x20;

If you haven't already, install the latest versions of:

* [miniconda](https://www.anaconda.com/docs/getting-started/miniconda/main) (a tool for managing Python and dependencies)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) (a version control system)

> Tip: Package managers like Chocolatey or Scoop can be used instead of manual downloads.

### Creating and activating an environment

Open Command Prompt or PowerShell and create an environment to install kiara into. This is like giving kiara its own room, so that it will not interfere with other tools or projects on your computer. For example:

```⏎
conda create -n kiara_explore
```

You can replace `kiara_explore` with any name you choose for your environment.

Activate the conda environment:&#x20;

```⏎
conda activate kiara_explore
```

### Install kiara and its basic plugins

Kiara is installed using a package-management system for Python called `pip`. Simply use:

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

> Tip: to check which version of _kiara_ or any of its plugins are installed at any point, use `pip list | findstr kiara`&#x20;
