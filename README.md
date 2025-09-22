# DGT 2025: Error Span Annotation project
Materials for the ESA MT project @ DGT workshop 2025.

## Project goal

Find the best setup for applying LLMs to **annotating translation errors** on the level of error spans.

## Our data
The data we will work with are documents from various domains. The source documents are (mostly) in English, translations are into various languages.

Currently, we have the following data available (to be updated):

#### (1) Data samples from DGT (CONFIDENTIAL):

**Domain**: legal documents.

Samples from the documents translated by a subcontractor and then evaluated in DGT. 

![](img/sample-dgt.png)

**Error categories:**

- **Terminology** (minor/major)
- **Linguistic norm** (minor/major)
- **Job-specific style** (minor/major)
- **General style** (minor/major)
- **Design** (minor/major)

#### (2) Data from WMT24 (public):

**Domains**: books, news, social networks

Data from the [WMT24 shared task](https://www2.statmt.org/wmt24/). Documents were collected by shared task organizers, translated by various systems (LLM-based), and annotated by human annotators.

> ![](img/sample-wmt.png)

**Error categories**: Minor, Major.


## Our experiments

Inspired by the papers [Kocmi et al. (2024)](https://aclanthology.org/2024.wmt-1.131/) and [Kasner et al. (2025)](https://arxiv.org/abs/2504.08697), we will use LLMs to automate finding errors in translated texts.

### How to (not) go about that?

The naive approach would be to fire up the web interface of any LLM-based service, paste the source document and its translation into the chat window, and ask the model to *"check whether there are any translation errors"*. 

However, such an approach would lead to results that are too diverse and prompt sensitive. Crucially, it would not be possible to **automate and evaluate** such an approach.

### Better approach: Error span annotation

To make the results more consistent, we will formalize the task as error span annotation (ESA):

![alt text](img/span_eval_tasks.png)

For each error, we will ask for the following:
- character index of the **start and end** of the error span,
- span **error category**,
- (optionally) **explanation** of the error.

To make sure that we can automatically parse the LLM output, we will constrain its output to a list of annotations in the JSON format:

```json
{
  "annotations": [
    { 
      "reason": "[REASON]",
      "text": "[TEXT_SPAN]",
      "annotation_type": "[CATEGORY_INDEX]",
      "occurence_index": "[OCCURRENCE_INDEX]"
    },
    ...
  ]
}
```
where:

- `REASON` is the explanation of the annotation,
- `TEXT_SPAN` is the textual content of the annotated span,
- `CATEGORY_INDEX` is a number from the list of annotation categories,
- `OCCURRENCE_INDEX` is a 0-based index for disambiguating multiple occurrences of the same text.

> [!WARNING]  
> We will **not** ask LLMs for the start/end character indices of the span. As the model does not have access to these indices, it would only guess. Instead, we generate the textual content of the span (→LLMs are ok-ish at *copying* text) and find it in the source text during postprocessing.

### What can we experiment with?

| Area                    | Goal                                                                                                                         | What to experiment with                                                          | Expectations (=what have we observed so far)                                                                                                                                                   |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Models**              | Find the model that can perform ESA robustly across domains and languages.                                                   | Try different models (out of the available ones - see below).                    | Smaller open LLMs often struggle even with the format of the output itself, while larger LLMs can provide actionable outputs.                                                                  |
| **Error categories**    | Find the set of error categories that the LLMs can handle and are useful for later processing.                               | Try to change the number of error categories and their descriptions.             | LLMs can handle any number of categories, as long as they are described precisely.                                                                                                             |
| **Text chunk size**     | Find the length of the text segments that give the best results.                                                             | Try to change the size of text segments we evaluate.                             | LLMs tend to provide a constant number of annotations per output → they will overannotate shorter chunks and underannotate longer chunks.                                                      |
| **Prompt instructions** | Investigate whether the LLM outputs are robust to how we formulate the prompt.                                               | Try to reformulate the prompt.                                                   | Some part of the prompt are *critical*, some are *less critical*. What needs to be described precisely are the error categories and the output format.                                         |
| **Error explanations**  | Investigate whether model explanations are accurate and whether asking for them tends to improve results.                    | Manually analyze model explanations, try to enable / disable the `reason` field. | Asking for an explanation is similar to the chain-of-thought prompting, which can improve the results (but not necessarily). Also the explanations themselves are accurate only in some cases. |
| **Evaluation**          | Investigate how to evaluate the annotations produced by the model (either with reference human annotation or referenceless). | Try various metrics for evaluating the results.                                  | There are multiple ways to evaluate ESA. F1-score weighed by the length of the overlap tends to perform the best. Finding a good metric for evaluating error explanations is an open problem.  |

## Our framework
Constraining LLM outputs, preparing the prompts, visualizing the outputs, and evaluating the annotations can be a lot of work.

To make our lives easier, we will use [factgenie](https://github.com/ufal/factgenie): an annotation and visualization tool we created for this purpose.

<div style="border:1px solid #ccc; padding:10px; display:block; margin: auto;">
<a href="https://github.com/ufal/factgenie">
    <img src="img/fg-prev.png" alt="Preview" style="max-width:400px; height:auto; display:block; margin:auto;">
    </a>
</div>


### Factgenie instance

You will use the factgenie instance we have prepared specifically for this project at https://quest.ms.mff.cuni.cz/nlg/dgt-workshop/.

> [!NOTE]
> You will be given the username and the password in person.


The instance contains data and configuration files for you to start with.

Before starting any experiments, please, make yourselves familiar with the **usage guide**: 👉️ https://github.com/ufal/factgenie/wiki.

### How to run the experiments

#### LLM annotations
The basic workflow for collecting LLM annotations is the following:
- Go to **Annotate with LLMs**
- Create a new campaign
- Choose one of the example configuration files
- Modify the parameters (categories, prompt, etc.)
- Run the campaign
- Examine the results 
  - manually in the Browse / Analyze interface
  - programatically (factgenie CLI or custom code)

#### Human annotations
We can also collect data manually. For that, we can use the interface **Annotate with human annotators**. Please wait for further instructions regarding this workflow (will depend on available data).


> [!NOTE]
> Note that this guide is preliminary and can be superseded by instructions given in person. 

### Available models (to be updated)
- [**gemma3:27b**](https://ollama.com/library/gemma3:27b) (default for the DGT'25 workshop)
- permanently serviced at ÚFAL:
  - [**llama3.3:70b**](https://ollama.com/library/llama3.3:70b)
  - [**deepseek-r1:70b**](https://ollama.com/library/deepseek-r1:70b)
  - [**qwen2.5-coder-32B-instruc**t](https://ollama.com/library/qwen2.5-coder:32b)
- other ad hoc models
  - [**gpt-oss:20b**](https://ollama.com/library/gpt-oss:20b) (reasoning model)

We can run any other reasonably-sized open models that we agree on.