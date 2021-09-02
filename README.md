# code_review_automation

This repository is the replication package of the research work **"Automating Code Review Activities 2.0"**.

In our work we trained different T5 models for different tasks on different datasets. Here we provide everything needed to replicate our experiments. We also provide all our results.

In order to replicate our results you have two options:
* use our fine-tuned models to generate new predictions;
* train from scratch your own models.

For the second option (train your own models) you will need a **Google Colab** pro account and a **Google Cloud Storage** account (more details later).

## Resources

* In the `code` folder we provided the Google Colab notebook we used to:
  * `Preprocessing.ipynb`: preprocess the pre-training dataset and train the Sentencepiece model;
  * `PreTraining.ipynb`: pre-trian the T5 model;
  * `FineTuning.ipynb`: fine-tuning the T5 models on different tasks.

* `manual analysis.xlsx`: contains the results of the manual analysis we performed on some non perfect predictions.

[Here](https://zenodo.org/record/5387856#.YTDrPZ4zZyo) we stored the extra materials you need in order to replicare our results:

* `automating_code_review.zip` contains all the necessary to successfully run our Google Colab notebooks (see section **Train your T5 models** for more details).

* `datasets.zip` contains all the processed and splitted datasets we used:
  * fine-tuning
  * pre-training 
    * `pre-training.tsv`
  * fine-tuning
    * new_large
      * code-to-code
        * `test.tsv`, `train.tsv`, `val.tsv`
      * code-to-comment
        * `test.tsv`, `train.tsv`, `val.tsv`
      * code&comment-to-code
        * `test.tsv`, `train.tsv`, `val.tsv`
    * Tufano_etal_ICSE21
      * code-to-code
        * `test.tsv`, `train.tsv`, `val.tsv`
      * code&comment-to-code
        * `test.tsv`, `train.tsv`, `val.tsv`

* `generate_predictions.zip` contains the material to successfully generate predictions using a T5 model chekpoint (see section **Use our fine-tuned T5 models** for more details)

* `models.zip` contains the (best) checkpoints of our T5 models (pre-trained or not), for all the tasks (_code-to-code_, _code-to-comment_, _code&comment-to-code_) and both the datasets (_new_large_dataset_, _Tufano_etal_dataset_) we used. We also stored the checkpoint of the pre-trained model without any fine-tuning. The following is the content of the `models` folder:
  * T5_non_pre-trained_new_large_dataset_code-to-code
  * T5_non_pre-trained_new_large_dataset_code-to-comment
  * T5_non_pre-trained_new_large_dataset_code&\comment-to-code
  * T5_non_pre-trained_Tufano_etal_dataset_code-to-code
  * T5_non_pre-trained_Tufano_etal_dataset_code&\comment-to-code
  * T5_pre-trained
  * T5_pre-trained_new_large_dataset_code-to-code
  * T5_pre-trained_new_large_dataset_code-to-comment
  * T5_pre-trained_new_large_dataset_code&\comment-to-code
  * T5_pre-trained_Tufano_etal_dataset_code-to-code
  * T5_pre-trained_Tufano_etal_dataset_code&\comment-to-code

* `tokenizer.zip` contains the Sentencepiece model and vocabulary we trained on our pre-training dataset:
  * `TokenizerModel.model`, `TokenizerModel.vocab`

* `results.zip` contains for each dataset (_new_large_dataset_, _Tufano_etal_dataset_) the results obtained from each model (pre-trained or not) fine-tuned on each task (_code-to-code_, _code-to-comment_, _code&comment-to-code_). For each of these cases we stored the following files:
  * `source.txt`: input file for the model;
  * `target.txt`: target file (expected output);
  * `predictions_<k>.txt`: generated predictions file with *BEAM_SIZE = k (k=1,3,5,10)*.
  * `code_bleu_<k>.txt` or `bleu_<k>.txt`: **code_BLEU** or **BLEU** scores file (depending on the task) with *BEAM_SIZE = k (k=1,3,5,10)*
  * `confidence_<k>.txt`: confidence scores file with *BEAM_SIZE = k (k=1,3,5,10)* 


## Use our fine-tuned T5 models

In order to generate predictions with our models you need:
* the models checkpoints stored in `models.zip`;
* the content of the archive `generate_predictions.zip`;
* the datasets stored in `datasets.zip`.

The folder `generated_predictions` stores all the necessary code to generate the predictions of the T5 models with different *beam sizes* end evaluate them in terms of perfect predictions and *codeBLEU* (_code-to-code_ and _code&comment-to-code_ tasks) or BLEU (_code-to-comment_ task) score.

Fisrt, you need to convert the chekpoint model in PyThorch. To do that you need to run the following command from the `generated_prediction` folder:

```
python3 ./tf_2_pytorch_T5.py --tf_checkpoint_path <model_path> --config_file ./config.json --pytorch_dump_path ./dumps
```

where `<model_path>` is the path of the checkpoint model you want to use. For example, if you want to generate the predictions for the _code-to-code_ task on the _new_large_dataset_ using the pre-trained T5 model, you need to run the following command:

```
python3 ./tf_2_pytorch_T5.py --tf_checkpoint_path ../models/T5_pre-trained_new_large_dataset_code-to-code/model.ckpt-best --config_file ./config.json --pytorch_dump_path ./dumps
```

In the python script `generate_predictions/generate_predictions.py` set up the beam size (line 45) right task (line 47) and the path to the right dataset (line 48). For example:

```
beam_size = 1
batch_size = 64
task = 'code2code: '  # possible options: 'code2code: ', 'code&comment2code: ', 'code2comment: '
data_dir = "../dataset/fine-tuning/new_large/code-to-code/"
```

The script output is a textual file, `predictions_k.txt` (where k = beam_size), stored in the same dataset folder, containing all the generated predictions.

In order to evaluate the generated predictions in terms of perfect predictions and *codeBLEU* or BLEU score, you need to run one of the python scripts `generate_predictions/for_codeBLEU.py` or `generate_predictions/for_BLEU.py`, after you set the right paths to the target file, the predictions files and where to store the results (lines 69-71 or 17-19).


## Train your T5 models



