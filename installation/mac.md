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

Activate the conda environment:&#x20;

```⏎
conda activate kiara_explore
```

### Dependencies

Use conda to install the necessary packages. For the network analysis modules, you will need networkx:&#x20;

```⏎
conda install networkx
```

For the topic modelling modules, you will need...

### Install kiara and its plugins

Use pip to install _kiara_:

```⏎
pip install kiara
```

The installation may take a few minutes. Once complete, install _kiara's_ plugins:

```
pip install kiara_plugin.core_types kiara_plugin.onboarding kiara_plugin.tabular
```

> Tip: to check which version of _kiara_ or any of its plugins are installed at any point, use `pip list | grep kiara`&#x20;

### Install relevant git repository

For the network analysis modules, use:

```⏎
pip install git+https://github.com/...
```

For the topic modelling modules, use:

```⏎
pip install git+https://github.com/...
```

To check that the modules are successfully installed and see what operations are possible, use:

```⏎
kiara operation list
```
