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
