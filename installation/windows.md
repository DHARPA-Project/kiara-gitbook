---
description: >-
  How to install _kiara_ and its dependencies in a virtual environment for
  Windows users
---

# Windows

### Prerequisites&#x20;

If you haven't already, install the latest versions of:

* miniconda (download Windows installer from the miniconda website)
* Git (download from the Git website)&#x20;

> Tip: Package managers like Chocolatey or Scoop can be used instead of manual downloads.

### Creating and activating an environment

Open Command Prompt or PowerShell and create an environment to install _kiara_ into. For example:

```⏎
conda create -n kiara_explore
```

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

### Install \_kiara\_ and its plugins

Use pip to install _kiara_:

```⏎
pip install kiara
```

Once complete, install _kiara's_ plugins:

```
pip install kiara_plugin.core_types kiara_plugin.onboarding kiara_plugin.tabular
```

> Tip: to check which version of _kiara_ or any of its plugins are installed at any point, use `pip list | findstr kiara`&#x20;

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
