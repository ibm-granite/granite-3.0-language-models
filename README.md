<p align="center">
  <img src="figures/granite-3_0-language-models-3x-v1.png" />
</p>

<p align="center">
  :books: <a href="/granite-3-language-models.pdf">Paper</a>&nbsp | :hugs: <a href="https://huggingface.co/collections/ibm-granite/granite-30-language-models-66fdb59bbb54785c3512114f">HuggingFace Collection</a>&nbsp | 
  :speech_balloon: <a href="https://github.com/orgs/ibm-granite/discussions">Discussions Page</a>&nbsp
<br>

---
## Introduction to Granite 3.0 Language Models
Granite 3.0 language models are a new set of lightweight state-of-the-art, open foundation models that natively support multilinguality, coding, reasoning, and tool usage, including the potential to be run on constrained compute resources. All the models are publicly released under an Apache 2.0 license for both research and commercial use. The models' data curation and training procedure were designed for enterprise usage and customization in mind, with a process that evaluates datasets for governance, risk and compliance (GRC) criteria, in addition to IBM's standard data clearance process and document quality checks.

Granite 3.0 includes 4 different models of varying sizes:
- Dense Models: A 2B and 8B parameter model, trained on 12 trillion tokens in total.
- Mixture-of-Expert (MoE) Models: A sparse 1B and 3B MoE model, with 400M and 800M activated parameters respectively, trained on 10 trillion tokens in total.

Accordingly, these options provide a range of models with different compute requirements to choose from, with appropriate trade-offs with their performance on downstream tasks. At each scale, we release a base model — checkpoints of models after pretraining, as well as instruct checkpoints — models finetuned for dialogue, instruction-following, helpfulness, and safety.

## Data Collection
Granite 3.0 language models are trained using data from various sources such as unstructured natural language text and code data from the Web curated by IBM, a collection of synthetic datasets generated by IBM, and publicly available high-quality datasets with permissible licenses. For governance, all our data undergoes a data clearance process subject to technical, business, and governance review. This comprehensive process captures critical information about the data, including but not limited to their content description ownership, intended use, data classification, licensing information, usage restrictions, how the data will be acquired, as well as an assessment of sensitive information (i.e, personal information). For code, we annotate each code file with license information associated with the respective repository, found via Github APIs and only keep files with permissive licenses for model training. In addition, we also filter out all data obtained from sources that match URLs in IBM’s URLs blocking-list. Please refer to [Granite 3.0 Language Models technical report](https://github.com/ibm-granite/granite-3.0-language-models/blob/main/granite-3-language-models.pdf) for more details on the individual categories and datasets.

## Pre-training
Granite 3.0 language models are trained on 10T to 12T tokens of language and code data, sourced from different domains. Data is tokenized via byte pair encoding (BPE, (Sennrich et al., 2015)), employing the same tokenizer as StarCoder (Li et al., 2023d). Below, we highlight the most important components of our pretraining strategy:

**Data Mixture.**
We craft the pretraining data mixture with two goals: 1) maximize the model’s performance across a diverse set of domains and tasks without bias toward a specific type of data or task; 2) leverage both high-quality and medium-quality data for optimal performance. To achieve these two goals, we adopt the 2-stage data mixture strategy used in MiniCPM (Hu et al., 2024) and JetMoE (Shen et al., 2024b).

**Training Hyperparameters.**
In Shen et al. (2024c), we proposed a systematic way of doing a hyperparameter search on a small scale and 0-shot transfer the hyperparameter to a large scale. The core part of this method is maximum update parameterization and a power scheduler.

**Model Parallelism.**
We use a combination of 3D parallelism (Tensor Parallelism (Shoeybi et al., 2020), Pipeline Parallelism (Narayanan et al., 2021b) and Data Parallelism (Li et al., 2020)) for training all our models.

## Post-training
We develop the post-training (instruct) variants of our Granite 3.0 models by further training the pre-trained checkpoints, focusing on instruction-following capabilities and alignment with human values. We employ a diverse set of techniques with a structured chat format, including curriculum-based supervised finetuning, model alignment using proximal policy optimization (PPO), best-of-N sampling, BRAIn (Pandey et al., 2024), and model merging.

## Evaluation Results
We conduct an extensive evaluation of Granite 3.0 language models on a comprehensive list of benchmarks. We evaluate our based models on benchmarks covering a variety of domains, namely *Human Exams*, *Commonsense*, *Reading Comprehension*, *Reasoning*, *Code*, and **Math**. In addition, the evaluation of our instruct models also covers benchmarks in *Instruction Following* and *Multilinguality*.

Our evaluation results show that our Granite 3.0 models outperform models of similar parameter sizes on many of these benchmarks, demonstrating strong performance particularly in knowledge, reasoning, function calling, multilingual, code support, as well as enterprise tasks like cybersecurity and retrieval augmented generation (RAG). The following figures show the average performance of base and instruct models along with other models of similar size. We provide further evaluation results in [Granite 3.0 Language Models technical report](https://github.com/ibm-granite/granite-3.0-language-models/blob/main/granite-3-language-models.pdf).

<figure>
  <img src="./figures/coming_soon.jpg""
  alt="Base models performance.">
  <figcaption>Average performance of base models across 19 tasks from 6 domains.</figcaption>
</figure>

<figure>
  <img src="./figures/coming_soon.jpg""
  alt="Instruct models performance.">
  <figcaption>Average performance of instruct models across 23 tasks from 8 domains.</figcaption>
</figure>

## How to Use our Models?
To use any of our models, pick an appropriate `model_path` from:
1. `ibm-granite/granite-3.0-2b-base`
2. `ibm-granite/granite-3.0-2b-instruct`
3. `ibm-granite/granite-3.0-8b-base`
4. `ibm-granite/granite-3.0-8b-instruct`
5. `ibm-granite/granite-3.0-1b-a400m-base`
6. `ibm-granite/granite-3.0-1b-a400m-instruct`
7. `ibm-granite/granite-3.0-3b-a800m-base`
8. `ibm-granite/granite-3.0-3b-a800m-instruct`

### Inference
This is a simple example of how to use Granite-3.0-1B-A400M-Instruct model.

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

device = "auto"
model_path = "ibm-granite/granite-3.0-1b-a400m-instruct"
tokenizer = AutoTokenizer.from_pretrained(model_path)
# drop device_map if running on CPU
model = AutoModelForCausalLM.from_pretrained(model_path, device_map=device)
model.eval()
# change input text as desired
chat = [
    { "role": "user", "content": "Please list one IBM Research laboratory located in the United States. You should only output its name and location." },
]
chat = tokenizer.apply_chat_template(chat, tokenize=False, add_generation_prompt=True)
# tokenize the text
input_tokens = tokenizer(chat, return_tensors="pt").to(device)
# generate output tokens
output = model.generate(**input_tokens, 
                        max_new_tokens=100)
# decode output tokens into text
output = tokenizer.batch_decode(output)
# print output
print(output)
```
## How to Download our Models?
The model of choice (granite-3.0-1b-a400m-instructin this example) can be cloned using:
```shell
git clone https://huggingface.co/ibm-granite/granite-3.0-1b-a400m-instruct
```

<!-- ### Finetuning -->
<!-- We use [Dolomite Engine](https://github.com/ibm-granite/dolomite-engine/) for finetuning (or instruction tuning) all our models. We provide sample scripts for finetuning `ibm-granite/granite-3b-code-base`. To finetune the models, simply follow these steps:
```shell
git clone https://github.com/ibm-granite/dolomite-engine/
cd dolomite-engine

# you might need to modify configs/granite-example/training.yml
sh scripts/finetune.sh configs/granite-example/training.yml

# once the model is trained, convert to HuggingFace-compatible safetensors
sh scripts/export.sh configs/granite-example/export.yml
```

> [!TIP]
> If you would like to use [padding-free transformers](https://huggingface.co/blog/mayank-mishra/padding-free-transformer) to save memory footprint and FLOPs during training, follow the instructions in the [Dolomite Engine README](https://github.com/ibm-granite/dolomite-engine?tab=readme-ov-file#huggingface-compatible-custom-models) for more details. -->

## How to Contribute to this Project?
Plese check our [Guidelines](/CONTRIBUTING.md) and [Code of Conduct](/CODE_OF_CONDUCT.md) to contribute to our project.

## Model Cards
The model cards for each model variant are available in their respective HuggingFace repository. Please visit our collection [here](https://huggingface.co/collections/ibm-granite/granite-30-language-models-66fdb59bbb54785c3512114f).

## License 
All Granite 3.0 Language Models are distributed under [Apache 2.0](./LICENSE) license.

## Would you like to provide feedback?
Please let us know your comments about our family of language models by visiting our [collection](https://huggingface.co/collections/ibm-granite/granite-30-language-models-66fdb59bbb54785c3512114f). Select the repository of the model you would like to provide feedback about. Then, go to *Community* tab, and click on *New discussion*. Alternatively, you can also post any questions/comments on our [github discussions page](https://github.com/orgs/ibm-granite/discussions).
