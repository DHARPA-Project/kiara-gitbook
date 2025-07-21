# What is data orchestration?

Data orchestration is the process of managing and conducting data-related tasks. In kiara, you can organize, transform, and track your research data throughout the analytical process.

### From Sources to Data

Historical sources don't arrive as neatly structured data. Letters, census records, newspapers, and other primary sources must be transformed into machine-readable formats before computational analysis becomes possible. This transformation process, in which you _create_ your dataset, involves critical decisions that shape your research outcomes – yet many digital tools obscure that process. When you simply click through software interfaces, you may lose sight of how your sources become data, how that data is manipulated, and ultimately how your interpretations relate to the original materials.

### Data Orchestration, documented

Kiara therefore allows you to both perform and track each of those steps. Rather than processing your data through monolithic "black box" tools, kiara facilitates the creation of data [pipelines](key-concepts.md#pipeline) that consist of distinct, interconnected [modules](key-concepts.md#module). Each module has clearly defined inputs, outputs, and processes, allowing you to understand exactly what is happening to your data at each step. At the same time, kiara automatically tracks the ancestry of your data through every transformation. At any point, you can see the [lineage](key-concepts.md#lineage) of how a particular dataset was created, including all the inputs and processes involved, making your [workflow](key-concepts.md#workflow) and findings both explainable and reproducible.

<figure><img src="../.gitbook/assets/kiara illuminates.png" alt=""><figcaption></figcaption></figure>
