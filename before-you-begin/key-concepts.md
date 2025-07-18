---
description: 'Understanding these core concepts will help you work effectively with kiara:'
---

# Key concepts

## Module

A module is kiara's basic building block for data operations. Each module is a self-contained unit designed to perform a specific task within your research workflow. A kiara module consists of three essential sections:

* **Input**: Specifies what data the module requires, with descriptions of expected properties to help you make informed decisions.
* **Process**: Contains the Python code required to execute the module.
* **Output**: Defines what the module will produce after processing.

For example, the `<add_nodes>` module requires three inputs: a graph object, a nodes table, and a string (the column name for node indexes), and outputs an augmented graph object. The user can find out what the necessary inputs are at any time by using the command:

```
kiara operation explain [insert module name]
```

<figure><img src="../.gitbook/assets/Screenshot 2025-07-18 at 13.58.11.png" alt=""><figcaption><p>Partial view of Kiara's explanation of the assemble.network_graph module</p></figcaption></figure>

## Plugin

A plugin in kiara is a collection of related modules that extend the system's functionality. Plugins allow researchers to add specialized capabilities without modifying the core system. Anyone with basic Python knowledge can create plugins by adapting existing Python code into kiara's module structure, making the system highly extensible and community-driven.

## Pipeline

A pipeline is a sequence of connected modules where the output of one module becomes the input for another. Kiara's modular approach allows you to create pipelines that compartmentalize workflow steps that would typically appear in a single Python script. This compartmentalization enables greater flexibility, as modules can be rearranged and recombined to create different analytical pathways according to your research needs.

## Workflow

A workflow in kiara represents your entire research process, potentially consisting of multiple pipelines and individual modules. Workflows can be saved, shared, reused, and modified. The workflow concept is central to kiara's approach to transparency and reproducibility, as it captures not just individual operations but the entire sequence of transformations your data undergoes during research.

## Lineage

Lineage (or data ancestry) is kiara's record of how data values are created and transformed throughout your research process. When you request the lineage of a data object (using a command like `<kiara data explain --lineage [object_name]>`), kiara provides a comprehensive history showing all the modules and inputs that contributed to creating that object.

For example, requesting the lineage of a network graph might show that it was first created from an edges table and later augmented with additional node attributes from a separate data source. This lineage documentation is crucial for understanding data provenance, ensuring research transparency, and enabling critical reflection on methodological choices.

<figure><img src="../.gitbook/assets/Screenshot 2022-10-06 at 10.38.56 copy.png" alt=""><figcaption><p>Kiara lineage visualisation</p></figcaption></figure>

## Context

Kiara has a default context which is always available, but users can also create different contexts within their virtual environment for each project they undertake with kiara. Each context stores:

* the data you import or create
* the operations you run
* your notes and decisions
