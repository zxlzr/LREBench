# LREBench: A low-resource relation extraction benchmark.


## Contents

- [LREBench](#lrebench)
  - [Contents](#contents)
  - [Environment](#environment)
  - [Datasets](#datasets)
  - [Normal Prompt-based Tuning](#normal-prompt-based-tuning)
    - [1 Initialize the answer words](#1-initialize-the-answer-words)
    - [2 Split the Dataset](#2-split-the-dataset)
    - [3 Prompt-based Tuning](#3-prompt-based-tuning)
    - [4 Different prompts](#4-different-prompts)
  - [Balancing](#balancing)
    - [1 Re-sampling](#1-re-sampling)
    - [2 Re-weighting Loss](#2-re-weighting-loss)
  - [Data Augmentation](#data-augmentation)
    - [1 Prepare the environment](#1-prepare-the-environment)
    - [2 Try different DA methods](#2-try-different-da-methods)
  - [Self-training for Semi-supervised learning](#self-training-for-semi-supervised-learning)
  - [Standard Fine-tuning](#standard-fine-tuning)


## Environment

To install requirements:

```shell
>> conda create -n LREBench python=3.9
>> conda activate LREBench
>> pip install -r requirements.txt
```


## Datasets

We provide 8 benchmark datasets and prompts used in our experiments.

- [SemEval](https://github.com/thunlp/PTR/tree/main/datasets/semeval)
- [TACREV](https://github.com/thunlp/PTR/tree/main/datasets/tacrev)
- [Wiki80](https://github.com/thunlp/OpenNRE/blob/master/benchmark/download_wiki80.sh)
- [SciERC](http://nlp.cs.washington.edu/sciIE/)
- [ChemProt](https://github.com/ncbi-nlp/BLUE_Benchmark)
- [DialogRE](https://dataset.org/dialogre/)
- [DuIE2.0](https://github.com/qxiaomo1128/DuIE2.0-data)
- [CMeIE]([dataset/CMeIE](https://tianchi.aliyun.com/dataset/dataDetail?dataId=95414))

All processed full datasets can be [download](https://drive.google.com/drive/folders/1OXxMr4SXUhehJx1XmDdrhJAppfo_ZNUh?usp=sharing) and need to be placed in the [*dataset*](dataset) folder.
The expected files of one dataset contains **rel2id.json**, **train.json** and **test.json**.
And we also provide the sampling code and to convert them into 8-shot ([sample_8shot.py](sample_8shot.py)) and 10% ([sample_10.py](sample_10.py)) datasets. For example,


## Normal Prompt-based Tuning

### 1 Initialize the answer words

Use the command below to get the answer words to use in the training.

```shell
>> python get_label_word.py --modelpath roberta-large --dataset semeval
```

The `{modelpath}_{dataset}.pt` will be saved in the *dataset* folder, and you need to assign the `modelpath` and `dataset` with names of the pre-trained language model and the dataset to be used before.

### 2 Split the Dataset

If few-shot datasets are required, run the following commands. For example, train on 10% SemEval:

```shell
>> python sample_10.py -i dataset/semeval/train.json -o dataset/semeval/
>> cd dataset/semeval
>> mkdir 10-1
>> cp rel2id.json test.jsom ./10-1/
>> cp train10per_1.json ./10-1/train.json
>> cp unlabel10per_1.json ./10-1/label.json
```


### 3 Prompt-based Tuning

All running scripts for each dataset are in the *scripts* folder. 
For example, train *KonwPrompt* on SemEval, CMeIE and ChemProt with the following command:

```shell
>> bash scripts/semeval.sh  # RoBERTa-large
>> bash scripts/CMeIE.sh    # Chinese RoBERTa-large
>> bash scripts/ChemProt.sh # BioBERT-large
```


### 4 Different prompts

Simply add parameters to the scripts.

Template Prompt: `--use_template_words 0`

Schema Prompt: `--use_template_words 0 --use_schema_prompt True`

PTR: refer to [PTR](https://github.com/thunlp/PTR)


## Balancing

### 1 Re-sampling

- Create the re-sampled training file based on the 10% training set by *resample.py*.
  
  ```shell
  usage: resample.py [-h] --input_file INPUT_FILE --output_dir OUTPUT_DIR --rel_file REL_FILE

  optional arguments:
    -h, --help            show this help message and exit
    --input_file INPUT_FILE, -i INPUT_FILE
                          The path of the training file.
    --output_dir OUTPUT_DIR, -o OUTPUT_DIR
                          The directory of the sampled files.
    --rel_file REL_FILE, -r REL_FILE
                          the path of the relation file
  ```
    
  ```shell
  >> mkdir dataset/semeval/10sa-1
  >> python resample.py -i dataset/semeval/10/train.json -r dataset/semeval/rel2id.json -o dataset/semeval/
  >> cd dataset/semeval
  >> cp rel2id.json test.json ./10sa-1/
  >> cp sa_1.json ./10sa-1/train.json
  ```

### 2 Re-weighting Loss

Simply add the *useloss* parameter to script for choosing various re-weighting loss.

For exampe: `--useloss MultiFocalLoss`.
(chocies: MultiDSCLoss, MultiFocalLoss, GHMC_Loss, LDAMLoss)


## Data Augmentation

### 1 Prepare the environment

Following the instructions [nlpaug](https://github.com/makcedward/nlpaug)

(Note: `transformers 4.7.0` is required in prompt-tuning and also can be used in DA.)

### 2 Try different DA methods

- Generate augmented data
    
  ```shell
  >> python DA.py -h
  usage: DA.py [-h] --input_file INPUT_FILE --output_file OUTPUT_FILE --DAmethod
                {word2vec,TF-IDF,word_embedding_bert,word_embedding_distilbert,word_embedding_roberta,random_swap,synonym}
                [--model_name MODEL_NAME] [--locations {sent1,sent2,sent3,ent1,ent2} [{sent1,sent2,sent3,ent1,ent2} ...]]

  optional arguments:
    -h, --help            show this help message and exit
    --input_file INPUT_FILE, -i INPUT_FILE
                          Input file containing dataset
    --output_file OUTPUT_FILE, -o OUTPUT_FILE
                          Output file containing perturbed dataset
    --DAmethod {word2vec,TF-IDF,word_embedding_bert,word_embedding_distilbert,word_embedding_roberta,random_swap,synonym}, -d {word2vec,TF-IDF,word_embedding_bert,word_embedding_distilbert,word_embedding_roberta,random_swap,synonym}
                          Data augmentation method
    --model_name MODEL_NAME, -mn MODEL_NAME
                          model from huggingface
    --locations {sent1,sent2,sent3,ent1,ent2} [{sent1,sent2,sent3,ent1,ent2} ...], -l {sent1,sent2,sent3,ent1,ent2} [{sent1,sent2,sent3,ent1,ent2} ...]
                          List of positions that you want to manipulate
  ```

  Take context-level DA based on contextual word embedding on ChemProt for example:

  ```bash
  python DA.py \
      -i dataset/ChemProt/train10per_1.json \
      -o dataset/ChemProt/aug \
      -d word_embedding_bert \
      -mn dmis-lab/biobert-large-cased-v1.1 \
      -l sent1 sent2 sent3
  ```

- Delete repeated instances and get the final augmented data

  ```shell
  >> python merge_dataset.py -h
  usage: merge_dataset.py [-h] [--input_files INPUT_FILES [INPUT_FILES ...]] [--output_file OUTPUT_FILE]

  optional arguments:
    -h, --help            show this help message and exit
    --input_files INPUT_FILES [INPUT_FILES ...], -i INPUT_FILES [INPUT_FILES ...]
                          List of input files containing datasets to merge
    --output_file OUTPUT_FILE, -o OUTPUT_FILE
                          Output file containing merged dataset
  ```

  For example:

  ```bash
  python merge_dataset.py \
      -i dataset/ChemProt/train10per_1.json dataset/ChemProt/aug/aug_1.json \
      -o dataset/ChemProt/aug/merge_1.json
  ```

- Obtain the 30% and 100% augmented dataset
  ```shell
  usage: augmented.py [-h] --input_file INPUT_FILE --output_dir OUTPUT_DIR
  
  optional arguments:
    -h, --help            show this help message and exit
    --input_file INPUT_FILE, -i INPUT_FILE
                          Input file containing the augmented dataset
    --output_dir OUTPUT_DIR, -o OUTPUT_DIR
                          The directory of the augmented files.
  ```
  
  ```shell
  >> python augmented.py -i dataset/ChemProt/aug/merge_1.json -o dataset/ChemProt/aug
  ```

## Self-training for Semi-supervised learning

- Train a teacher model on a few label data
- Place the unlabeled data **label.json** (that can obtain by randomly sample training sets) in the dataset folder
- Assigning pseudo labeled using the trained teacher model: add `--labeling True` to the script and get the pseudo-labeled dataset *label2.json*.
- Put the gold-labeled data and pseudo-labeled data together like 
  ```shell
  >> python self-train.py -g dataset/semeval/10-1/train.json -p dataset/semeval/10-1/label2.json -la dataset/semeval/10la-1
  >> cd dataset/semeval
  >> cp rel2id.json test.json ./10la-1/
  ```
- Train the final student model: add `--stutrain True` to the script


## Standard Fine-tuning
[Fine-tuning](Fine-tuning/)
